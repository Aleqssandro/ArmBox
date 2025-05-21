# 01 – Introdução ao DTB/DTS em TV Boxes com Armbian

Muitos dispositivos como TV Boxes Android utilizam processadores ARM que **não descobrem o hardware automaticamente**, como ocorre em PCs tradicionais. Em vez disso, eles dependem de uma estrutura chamada **Device Tree**.

---

## 🌳 O que é o Device Tree?

O **Device Tree** é uma forma de descrever o hardware do dispositivo para o kernel Linux. Ele informa ao sistema operacional:

- Quantidade e localização da memória RAM
- Endereços dos dispositivos (UART, USB, Ethernet, etc.)
- GPIOs disponíveis
- Clock, voltagem, pinos

Essa descrição vem de arquivos `.dts` (Device Tree Source) que são **compilados** em `.dtb` (Device Tree Blob), o que o kernel realmente usa no boot.

---

## 📁 Principais Arquivos

| Extensão | Significado                | É legível? | Pode ser editado? |
|----------|----------------------------|------------|-------------------|
| `.dts`   | Device Tree Source         | ✅ Sim     | ✅ Sim            |
| `.dtb`   | Device Tree Blob (binário) | ❌ Não     | 🔁 Sim (convertendo) |

---

## 🔄 Conversão entre DTS e DTB

Usamos o utilitário `dtc` (Device Tree Compiler) para converter entre os dois formatos:

```bash
# De DTB (binário) para DTS (texto):
dtc -I dtb -O dts -o arquivo.dts arquivo.dtb

# De DTS (texto) para DTB (binário):
dtc -I dts -O dtb -o arquivo.dtb arquivo.dts
```

---

## ⚙️ Lógica de Boot no Armbian em TV Boxes

Ao ligar a TV Box, o processo de boot passa por uma sequência bem definida, controlada principalmente pelo **bootloader U-Boot** e os arquivos da pasta `/boot`.

### 🔹 1. Bootloader (U-Boot)

O **bootloader** geralmente está armazenado em uma partição especial da memória (eMMC, NAND ou cartão SD), e é responsável por iniciar o carregamento do sistema.

> Nos dispositivos Android, o bootloader pode estar bloqueado ou modificado. Ao usar Armbian, normalmente usamos um `u-boot` customizado na mídia removível (SD ou USB) para contornar isso.

---

### 📁 2. Estrutura da pasta `/boot`

Após o U-Boot encontrar o sistema de arquivos, ele carrega o kernel e as configurações localizadas na pasta `/boot`. Os arquivos mais relevantes são:

| Arquivo                 | Função                                                           |
|--------------------------|------------------------------------------------------------------|
| `Image` ou `zImage`     | Arquivo do kernel Linux (núcleo do sistema)                      |
| `*.dtb`                 | Device Tree Blob – descreve o hardware da TV Box                 |
| `extlinux/extlinux.conf`| Arquivo de configuração do bootloader (semelhante ao GRUB)       |
| `uInitrd` (opcional)    | Ramdisk usado em alguns casos para inicializar módulos extras    |

---

### 🧾 Exemplo de `extlinux.conf`

```txt
LABEL Armbian
  LINUX /zImage
  INITRD /uInitrd
  FDT /dtb/amlogic/meson-sm1-tx3-mini.dtb
  APPEND root=LABEL=ROOTFS rootflags=data=writeback rw console=ttyAML0,115200n8
```

---

### 🔁 Ordem de Boot

```text
🔌 TV Box Power ON
     ↓
🧭 Bootloader (U-Boot)
     ↓
📄 Carrega extlinux.conf
     ↓
📦 Define kernel (Image/zImage)
🌳 Define Device Tree (.dtb)
⚙️ Aplica parâmetros de boot
     ↓
🚀 Inicializa o sistema Linux (Armbian)
```

---

## 🧪 Comunicação UART com adaptador USB-TTL

Durante o processo de adaptação do Armbian, é comum precisar visualizar mensagens de boot e interagir diretamente com o sistema — especialmente se a TV Box não apresentar vídeo ou se travar no boot.

Para isso, usamos a interface **UART (serial)** presente na maioria das placas, conectando-a a um computador via adaptador **USB-TTL**.

### 🔌 Materiais necessários

- Adaptador USB-TTL (3.3V, ex: CH340, CP2102, FTDI)
- Jumpers macho-fêmea
- Acesso à porta UART da TV Box (geralmente identificada como TX, RX e GND)

### ⚙️ Configuração típica

- **Baud rate:** `115200`
- **Paridade:** `Nenhuma`
- **Bits de dados:** `8`
- **Stop bit:** `1`
- **Flow control:** `None`

Você pode usar ferramentas como:

```bash
sudo apt install minicom
minicom -D /dev/ttyUSB0 -b 115200
```

ou

```bash
sudo screen /dev/ttyUSB0 115200
```

---

> A UART é uma das ferramentas mais valiosas para **debug** e para recuperar dispositivos com boot corrompido. Além disso, ela permite extrair arquivos, editar configurações de boot ou até iniciar Armbian pela shell serial.
