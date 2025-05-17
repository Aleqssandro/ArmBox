# 📦 Armbian em TV Boxes Android

Este repositório documenta a instalação e adaptação do Armbian em TV Boxes Android. Ele está dividido em duas partes principais:

- ✅ **Parte 1**: Modelos suportados e passo a passo básico de instalação.
- 🛠️ **Parte 2**: Engenharia reversa de novos modelos usando DTB/DTS e criação de ISOs customizadas.

> ⚠️ **Aviso:** O processo pode **descaracterizar** o firmware original da TV Box. Siga por sua conta e risco.

---

## 📚 Índice

- [📦 Modelos Suportados](#-modelos-suportados)
- [🚀 Instalação Básica](#-instalação-básica)
- [🧠 Engenharia Reversa (DTB/DTS)](#-engenharia-reversa-dtbdts)
  - [📄 O que são DTB e DTS](#-o-que-são-dtb-e-dts)
  - [✏️ Como Editar e Recompilar](#-como-editar-e-recompilar)
  - [💿 Gerar ISO Personalizada](#-gerar-iso-personalizada)
- [🖼️ Imagens](#️-imagens)
- [📌 Contribuições](#-contribuições)
- [📃 Licença](#-licença)

---

## 📦 Modelos Suportados

Consulte os modelos já testados com status de compatibilidade, SoC, RAM e instruções específicas:

👉 [`catalogo-modelos/modelos-suportados.md`](catalogo-modelos/modelos-suportados.md)

---

## 🚀 Instalação Básica

Tutorial simples e direto para instalar Armbian em modelos suportados:

👉 [`catalogo-modelos/passo-a-passo-basico.md`](catalogo-modelos/passo-a-passo-basico.md)

---

## 🧠 Engenharia Reversa (DTB/DTS)

Esta parte é voltada para quem deseja adaptar o Armbian para **novos modelos de TV Box**.

### 📄 O que são DTB e DTS

Explicação dos arquivos Device Tree, como extraí-los e por que são importantes:

👉 [`engenharia-reversa/introducao-dtb-dts.md`](engenharia-reversa/introducao-dtb-dts.md)

### ✏️ Como Editar e Recompilar

Passo a passo para converter `.dtb` em `.dts`, editar e recompilar:

👉 [`engenharia-reversa/editar-dts-dtb.md`](engenharia-reversa/editar-dts-dtb.md)

### 💿 Gerar ISO Personalizada

Como criar uma imagem customizada do Armbian com seu próprio DTB:

👉 [`engenharia-reversa/gerar-iso-personalizada.md`](engenharia-reversa/gerar-iso-personalizada.md)

---

## 🖼️ Imagens

Screenshots, exemplos de estrutura e imagens dos dispositivos:

👉 [`/imagens`](imagens/)

---

## 📌 Contribuições

Pull requests são bem-vindos! Para contribuições maiores, abra uma issue primeiro.

---

## 📃 Licença

Distribuído sob a licença [MIT](LICENSE).

---

> Mantenedor: [Aleqssandro](https://github.com/Aleqssandro)
