Para a descaracterização desse modelo deve ser feito o uso de uma imagem do MiniArch funcional para personalizar uma imagem do Armbian de forma que ela se torne compatível

## Etapa 1 - Organizar um diretório para centralizar as ISO's baixadas e os arquivos que serão utilizados
```shell
mkdir -p ~/tv-box/ik316/
cd ~/tv-box/ik316/ # Nesse diretório devem ser copiadas as ISOS a serem utilizadas

mkdir mnt/miniarch-boot/
mkdir mnt/miniarch-rootfs/
mkdir mnt/armbian-rootfs/
mkdir mnt/ik316-custom/
mkdir miniarch-boot/
mkdir miniarch-lib/ # modules e firmware 
mkdir armbian-rootfs/
```

## Etapa 2 - Extração do u-boot do miniarch e dos arquivos de lib
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
cp -r mnt/miniarch-boot/* miniarch-boot/

# Desmontar a partição
umount mnt/miniarch-boot/
```

Montando a segunda partição para acessar o diretório /lib e copiar os arquivos
```shell
# Montar a partição com o rootfs
sudo mount /dev/mapper/loopXpX mnt/miniarch-rootfs/

# Copiar arquivos necessários
cp -r mnt/miniarch-rootfs/lib/modules miniarch-lib/
cp -r mnt/miniarch-rootfs/lib/firmware miniarch-lib/

# Desmontar e limpar
sudo umount mnt/miniarch-rootfs
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

## Etapa 3 - Extração do rootfs do Armbian
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
mount /dev/mapper/loopXpX mnt/armbian-rootfs/
sudo rsync -aAXv mnt/armbian-rootfs/ armbian-rootfs/

# Desmontar a partição
sudo umount mnt/armbian-rootfs/
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

## Etapa 5 - Copiar o bootloader e o rootfs para a ISO customizada
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
sudo rsync -aAXv armbian-rootfs/ mnt/ik316-custom/

# Desmontar
sudo umount mnt/ik316-custom/
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

## Etapa 6 - Substituir todos os arquivos do diretório /boot do rootfs da iso-custom pelos arquivos contido na pasta "boot-miniarch"
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

sudo rsync -aAXv --exclude='/boot/*' armbian-rootfs/ mnt/ik316-custom/
```

## Etapa 7 - Editar os caminhos no /boot da iso-custom


## Etapa 8 - Substituir o modules e o firmware do armbian pelo do miniarch

