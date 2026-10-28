# Compilação da Imagem Linux com Buildroot

## 1. Objetivo

Este documento descreve o processo de configuração e compilação de uma imagem Linux embarcada utilizando o Buildroot, destinada à plataforma Raspberry Pi 4. A configuração integra os componentes necessários para executar uma aplicação Flask em um contêiner Docker.

## 2. Ambiente de desenvolvimento

- **Sistema operacional do ambiente de compilação:** Ubuntu Server 24.04 LTS, em máquina virtual no Microsoft Azure.
- **Sistema de construção:** Buildroot.
- **Versão do Buildroot:** `2026.08-345-ga962be2eea`.
- **Plataforma-alvo:** Raspberry Pi 4, arquitetura ARM64.
- **Árvore externa de configuração:** `br2-external`, localizada no repositório do projeto.

## 3. Estrutura utilizada

O projeto utiliza uma árvore externa do Buildroot para manter as configurações específicas do TP-Aula08 separadas dos arquivos originais do Buildroot.

Os principais arquivos são:

- `br2-external/external.desc`: identifica a árvore externa.
- `br2-external/Config.in`: declara opções de configuração adicionais.
- `br2-external/external.mk`: permite integrar componentes adicionais ao sistema de construção.
- `br2-external/configs/raspberrypi4_64_defconfig`: configuração-base da plataforma.
- `br2-external/board/rpi4/overlay/`: contém arquivos incorporados ao sistema de arquivos final.
- `br2-external/board/rpi4/users.table`: define o usuário adicional do sistema.

## 4. Configuração do Buildroot

O diretório de trabalho do Buildroot utilizado no ambiente é:

`/home/azureuser/buildroot`

O diretório do projeto é:

`/home/azureuser/TP-Aula08`

Para carregar a configuração específica do projeto, foi utilizado o seguinte comando:

```bash
cd ~/buildroot

make O=/home/azureuser/buildroot-check \
  BR2_EXTERNAL=/home/azureuser/TP-Aula08/br2-external \
  raspberrypi4_64_defconfig
```

Esse comando prepara uma configuração de referência em um diretório separado, permitindo verificar se a árvore externa e o arquivo de configuração são reconhecidos pelo Buildroot.

A configuração contempla a plataforma Raspberry Pi 4 de 64 bits, o sistema de arquivos raiz, os arquivos adicionais do projeto e os componentes necessários à execução do Docker.

## 5. Compilação da imagem

Após configurar o ambiente de compilação, o Buildroot é executado utilizando a árvore externa do projeto.

```bash
cd ~/buildroot

make O=/home/azureuser/buildroot \
  BR2_EXTERNAL=/home/azureuser/TP-Aula08/br2-external
```

O processo compila os componentes selecionados e gera os arquivos necessários para a imagem Linux embarcada. O tempo de compilação depende dos recursos disponíveis na máquina virtual e das opções habilitadas.

## 6. Arquivo gerado

A imagem principal gerada pelo processo de compilação é:

`/home/azureuser/buildroot/output/images/sdcard.img`

Essa imagem destina-se à preparação de um cartão SD para a plataforma-alvo. Antes de utilizá-la em hardware físico, é necessário confirmar a integridade do arquivo e seguir um procedimento adequado de gravação do cartão.

## 7. Verificação da integridade

O projeto contém um arquivo de evidência para registrar o hash SHA-256 da imagem:

`evidence/sdcard-sha256.txt`

O hash pode ser recalculado com:

```bash
sha256sum ~/buildroot/output/images/sdcard.img
```

O resultado deve ser comparado com o valor registrado na evidência, desde que o arquivo de imagem não tenha sido alterado posteriormente.

## 8. Limitações

A geração da imagem no ambiente virtual não comprova, por si só, o funcionamento integral do sistema em um Raspberry Pi 4 físico. A validação final no hardware depende da gravação da imagem em um cartão SD compatível, da inicialização da placa e da verificação dos serviços configurados.

## 9. Resultado

O ambiente foi configurado para utilizar o Buildroot com a árvore externa do TP-Aula08, e o processo de compilação gerou a imagem `sdcard.img`. Os testes de inicialização em hardware físico devem ser documentados separadamente, caso sejam realizados.
