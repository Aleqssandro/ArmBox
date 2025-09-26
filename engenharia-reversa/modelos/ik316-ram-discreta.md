Para a descaracterização desse modelo deve ser feito o uso de uma imagem do MiniArch funcional para personalizar uma imagem do Armbian de forma que ela se torne compatível

## Etapa 1 - Organizar um diretório para centralizar as ISO's baixadas e os arquivos que serão utilizados
```shell
mkdir -p ~/tv-box/ik316/
cd ~/tv-box/ik316/ # Nesse diretório devem ser copiadas as ISOS a serem utilizadas

mkdir -p mnt/miniarch-boot/ mnt/miniarch-rootfs/
mkdir mnt/armbian-boot/ mnt/armbian-rootfs/
mkdir mnt/ik316-custom/
mkdir miniarch-boot/ miniarch-rootfs/
mkdir armbian-boot/ armbian-rootfs/
```

## Etapa 2 - Extração do boot e do rootfs do miniarch
Para essa etapa é necessário extrair a pasta boot da iso do miniarch, ela se encontra na primeira partição, contendo arquivos importantes como o bootloader, extlinux.conf e o Image

Montando a iso do MiniArch no diretório criado e copiando os arquivos de interesse
```shell
fdisk -l img-miniarch.img # Identifica qual partição é boot e qual é rootfs

# Criar loop
LOOP=$(sudo losetup -f --show img-miniarch.img)
echo $LOOP

# Adicionar partições
sudo partprobe $LOOP # Atualiza tabelas
sudo kpartx -av $LOOP # Cria as partições em dispositivos virtuais

# Montar partição e copiar os arquivos
sudo mount /dev/mapper/loopXpX mnt/miniarch-boot/ # Partição boot
sudo rsync -aAXv mnt/miniarch-boot/* miniarch-boot/

# Desmontar a partição
sudo umount mnt/miniarch-boot/
```

Montando a segunda partição para acessar o diretório /lib e copiar os arquivos
```shell
# Montar a partição com o rootfs
sudo mount /dev/mapper/loopXpX mnt/miniarch-rootfs/

# Copiar arquivos necessários
sudo rsync -aAXv mnt/miniarch-rootfs/* miniarch-rootfs/

# Desmontar e limpar
sudo umount mnt/miniarch-rootfs
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

## Etapa 3 - Extração do boot e do rootfs do Armbian
Nessa etapa, deve-se montar a partição rootfs da img do Armbian e utilizar alguns comandos para clonar todos os arquivos dessa partição para uma pasta, incluindo arquivos, diretórios, links simbólicos e etc.

```shell
fdisk -l img-armbian.img # Identifica qual partição é rootfs

# Criar loop
LOOP=$(sudo losetup -f --show img-armbian.img)
echo $LOOP

# Adicionar partições
sudo partprobe $LOOP # Atualiza tabelas
sudo kpartx -av $LOOP # Cria as partições em dispositivos virtuais

# Montar partição e copiar os arquivos com links simbólicos
sudo mount /dev/mapper/loopXpX mnt/armbian-rootfs/
sudo rsync -aAXv mnt/armbian-rootfs/ armbian-rootfs/

# Desmontar a partição
sudo umount mnt/armbian-rootfs/
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

```shell
fdisk -l img-armbian.img # Identifica qual partição é boot e qual é rootfs

# Criar loop
LOOP=$(sudo losetup -f --show img-miniarch.img)
echo $LOOP

# Adicionar partições
sudo partprobe $LOOP # Atualiza tabelas
sudo kpartx -av $LOOP # Cria as partições em dispositivos virtuais

# Montar partição e copiar os arquivos
sudo mount /dev/mapper/loopXpX mnt/armbian-boot/ # Partição boot
sudo rsync -aAXv mnt/armbian-boot/* armbian-boot/

# Desmontar a partição
sudo umount mnt/armbian-boot/
```

Montando a segunda partição para acessar o rootfs e copiar os arquivos
```shell
# Montar a partição com o rootfs
sudo mount /dev/mapper/loopXpX mnt/armbian-rootfs/

# Copiar arquivos necessários
sudo rsync -aAXv mnt/armbian-rootfs/* armbian-rootfs/

# Desmontar e limpar
sudo umount mnt/armbian-rootfs
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

## Etapa 4 - Criar uma iso com espaço reservado para o bootloader e para o rootfs
Nessa metodologia foi escolhido realizar a criação de uma ISO em branco, para evitar sobreposição de partições. Nesse projeto, foi reservado o primeiro 1MiB para o bootloader do sistema customizado e o restante do espaço será uma partição ext4 para o rootfs.

```shell
tamanho=$((2)) # X deve ser o tamanho em GB da ISO que deverá ser criada, nesse caso é 2
count=$((tamanho*1024)) # Totalizando 2048 MB
dd if=/dev/zero of=ik316-custom.img bs=1M count=$count status=progress
parted ik316-custom.img --script mklabel msdos
parted ik316-custom.img --script mkpart primary ext4 1MiB 100% # Espaço de 1MiB para o bootloader
```

## Etapa 5 - Copiar o bootloader para o 1MiB da ISO customizada e o rootfs do armbian para a partição criada
Nesta etapa é gravado o bootloader extraido do miniarch no primeiro 1MiB da iso customizada.

```shell
dd if=miniarch-boot/bootloader/bootloader.bin of=ik316-custom.img bs=1024 seek=8 conv=notrunc # Revise o caminho do bootloader em 'if'
```

```shell
# Identificar partição rootfs
fdisk -l ik316-custom.img

# Criar loop device
LOOP=$(sudo losetup -f --show ik316-custom.img)
echo $LOOP

# Mapear partições
sudo partprobe $LOOP
sudo kpartx -av $LOOP

# Formatar a partição rootfs em ext4
sudo mkfs.ext4 /dev/mapper/loopXpX

# Montar a partição rootfs
sudo mount /dev/mapper/loopXpX mnt/ik316-custom/

# Copiar todo o conteúdo do rootfs do Armbian
sudo rsync -aAXv --exclude='/boot/*' armbian-rootfs/ mnt/ik316-custom/ # Copia toda a pasta rootfs do Armbian, menos o /boot

# Desmontar
sudo umount mnt/ik316-custom/
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

## Etapa 6 - Substituir todos os arquivos do diretório /boot do rootfs da iso-custom pelos arquivos contido na pasta "boot-miniarch"
Nessa etapa são utilizados os arquivos de boot do Miniarch que são compatíveis com esse modelo de Tv Box. Além disso, é copiado o init do armbian para o boot da iso.

```shell
fdisk -l ik316-custom.img # Identifica qual partição é rootfs

# Criar loop
LOOP=$(sudo losetup -f --show ik316-custom.img)
echo $LOOP

# Adicionar partições
sudo partprobe $LOOP # Atualiza tabelas
sudo kpartx -av $LOOP # Cria as partições em dispositivos virtuais

# Montar partição e copiar os arquivos com links simbólicos
mount /dev/mapper/loopXpX mnt/ik316-custom/ # Montar o loop referente a partição rootfs do ik316-custom.img

sudo rsync -aAXv miniarch-boot/* mnt/ik316-custom/boot/
sudo rsync -aAXv armbian-boot/initrd.img-6.12.47-current-sunxi64 mnt/ik316-custom/boot/initramfs-linux.img

# Desmontar
sudo umount mnt/ik316-custom/
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

## Etapa 7 - Editar os caminhos no extlinux.conf e do fstab da iso-custom

```shell
fdisk -l ik316-custom.img # Identifica qual partição é rootfs

# Criar loop
LOOP=$(sudo losetup -f --show ik316-custom.img)
echo $LOOP

# Adicionar partições
sudo partprobe $LOOP # Atualiza tabelas
sudo kpartx -av $LOOP # Cria as partições em dispositivos virtuais

# Montar partição e copiar os arquivos com links simbólicos
mount /dev/mapper/loopXpX mnt/ik316-custom/ # Montar o loop referente a partição rootfs do ik316-custom.img

### Editando o extlinux.conf ###
sudo tee mnt/ik316-custom/boot/extlinux/extlinux.conf > /dev/null <<'EOF'
LABEL ik316-custom
  LINUX /boot/Image
  INITRD /boot/initramfs-linux.img
  FDT /boot/dtbs/allwinner/sun50i-h616-tanix-tx6s-axp313.dtb

  APPEND root=/dev/mmcblk0p1 rw rootwait earlycon console=ttyS0,115200n8 loglevel=6 consoleblank=0 fsck.repair=yes
EOF

### Editando o fstab ###
sudo tee mnt/ik316-custom/etc/fstab > /dev/null <<'EOF'
# <file system>   <mount point>  <type>  <options>                        <dump> <pass>
/dev/mmcblk0p1    /              ext4    defaults,commit=120,errors=remount-ro  0      1
tmpfs             /tmp           tmpfs   defaults,nosuid                        0      0
EOF

# Desmontar
sudo umount mnt/ik316-custom/
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

## Etapa 8 - Substituir o modules e o firmware do armbian pelo do miniarch

```shell
fdisk -l ik316-custom.img # Identifica qual partição é rootfs

# Criar loop
LOOP=$(sudo losetup -f --show ik316-custom.img)
echo $LOOP

# Adicionar partições
sudo partprobe $LOOP # Atualiza tabelas
sudo kpartx -av $LOOP # Cria as partições em dispositivos virtuais

# Montar partição e copiar os arquivos com links simbólicos
mount /dev/mapper/loopXpX mnt/ik316-custom/ # Montar o loop referente a partição rootfs do ik316-custom.img

# Apagar módulos e firmwares do Armbian
sudo rm -rf mnt/ik316-custom/lib/modules/*
sudo rm -rf mnt/ik316-custom/lib/firmware/*

# Copiar do MiniArch
sudo rsync -aAXv miniarch-rootfs/lib/modules/ mnt/ik316-custom/lib/modules/
sudo rsync -aAXv miniarch-rootfs/lib/firmware/ mnt/ik316-custom/lib/firmware/

# Desmontar
sudo umount mnt/ik316-custom/
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```
