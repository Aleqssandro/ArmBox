Para a descaracterização desse modelo deve ser feito o uso de uma imagem do MiniArch funcional para personalizar uma imagem do Armbian de forma que ela se torne compatível

## Etapa 1 - Organizar um diretório para centralizar as ISO's baixadas e os arquivos que serão utilizados
```shell
mkdir -p ~/tv-box/ik316/
cd ~/tv-box/ik316/ # Nesse diretório devem ser copiadas as ISOS a serem utilizadas

mkdir mnt/miniarch-boot/
mkdir mnt/miniarch-rootfs/
mkdir mnt/armbian/
mkdir mnt/ik316-custom/
mkdir boot-miniarch/
mkdir lib-miniarch/ # modules e firmware 
mkdir rootfs-armbian/
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
mount /dev/mapper/loopXpX mnt/ miniarch-boot/ # Partição boot
cp -r mnt/miniarch-boot/* boot-miniarch/

# Desmontar a partição
umount mnt/miniarch-boot/
```

Montando a segunda partição para acessar o diretório /lib e copiar os arquivos
```shell
# Montar a partição com o rootfs
sudo mount /dev/mapper/loopXpX mnt/miniarch-rootfs

# Copiar arquivos necessários
cp -r mnt/miniarch-rootfs/lib/modules lib-miniarch/
cp -r mnt/miniarch-rootfs/lib/firmware lib-miniarch/

# Desmontar e limpar
sudo umount /mnt/imgroot
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
mount /dev/mapper/loopXpX mnt/armbian/
sudo rsync -aAXv --exclude='/boot/*' mnt/armbian/ rootfs-armbian/

# Desmontar a partição
sudo umount mnt/armbian/
sudo kpartx -d $LOOP
sudo losetup -d $LOOP
```

## Etapa 4 - Criar uma iso com espaço reservado para o bootloader e para o rootfs

## Etapa 5 - Copiar o bootloader e o rootfs para a ISO customizada

## Etapa 6 - Substituir todos os arquivos do diretório /boot do rootfs da iso-custom pelos arquivos contido na pasta "boot-miniarch"

## Etapa 7 - Editar os caminhos no /boot da iso-custom

## Etapa 8 - Substituir o modules e o firmware do armbian pelo do miniarch