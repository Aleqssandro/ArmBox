# ArmBox
Estudos sobre adaptação do u-boot para  diversas arquiteturas de Tv Box para execução do SO Armbian

A análise do u-boot nas tv box é possível através da conexão UART na PCB utilizando um adaptador USB para TTL, no caso das Tv Boxes o Baud Rate é de 115200 baud.

Para a modificação de um arquivo de imagem para ser compatível com uma Tv Box nova é necessário a configuração do u-boot e os arquivos Device Tree Blob (DTB) descrevem a configuração de hardware de um sistema para o kernel.
A ideia é extrair o DTB original desses modelos e alterar o DTB presente em uma imagem do Armbian, de forma que ela compatibilize com a Tv Box.

Para isso, o primeiro passo é a extração do DTB original da tv box. Nesse procedimento deve-se acessar uma conexão UART na tv box e após a completa inicialização acessar o terminal sudo:
```sh
su
```
Após isso é necessário fazer a cópia de uma partição ou de todas as partições na forma de um arquivo de imagem (ex: .img) para um dispositivo de armazenamento conectado a tv box, como um pendrive ou sd card, para isso
```sh
# 📌 Verificar as partições disponíveis no sistema
ls /dev/block  # Exibe dispositivos como "emmc" (SD Card interno) ou "nand" (memória NAND)

# 📌 Criar um diretório para montar a partição
mkdir -p /mnt/sdcard  # Exemplo: Criando um ponto de montagem para um SD card

# 📌 Montar a partição desejada
mount /dev/block/mmcblk0p1 /mnt/sdcard  # Substitua pelo nome correto da partição

# 📌 Verificar partições que podem ser montadas na TV Box
mount

# 📌 Criar um arquivo de imagem da partição da Tv Box
dd if=/dev/block/nand0p1 of=/mnt/sdcard/particao.img  # Substitua "nand0p1" pela partição correta

# 📌 Após a cópia, desmonte a partição para evitar corrupção de dados
umount /mnt/sdcard

# 📌 Montar a imagem da partição da tv box para ver seus dados
fdisk -l caminho_para_arquivo.img # Verificar as partições internas na imagem
mount -o loop,offset=$((8192 * 512)) caminho_para_arquivo.img /mnt # Exemplo de montagem da imagem; 8192 -> é o setor de início da partição
```

Exemplo de uso, o dispositivo usado é um SD card com a partição que será montada chamada de "mmcblk0p1". A pasta que será montada é "/mnt/sdcard". A partição de interesse da tv box é "nand0p1"
```sh
su
mkdir /mnt/sdcard
mount /dev/block/mmcblk0p1 /mnt/sdcard
dd if=/dev/block/nand0p1 of=/mnt/sdcard/particao.img
umount /mnt/sdcard
```
Explicação das conversões de DTB para DTS e vice versa
```sh
utc -I dtb -O dts -o <path for output>/deviceTree.dts <path for input>/<file name>.dtb #Conversão de DTB para DTS
utc -I dts -O dtb -o <path for output>/deviceTree.dtb <path for input>/<file name>.dts #Conversão de DTS para DTB
```
