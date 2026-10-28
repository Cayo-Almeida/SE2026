# 02 — Configuração do Buildroot

## 1. Objetivo

O Buildroot foi utilizado para gerar uma imagem de Linux embarcado destinada ao Raspberry Pi 4, com arquitetura ARM64. O sistema foi configurado para executar os serviços necessários ao projeto, incluindo o Docker Engine e a aplicação Flask empacotada em um contêiner.

A proposta é manter o sistema direcionado à aplicação, evitando a instalação de componentes gráficos desnecessários.

## 2. Versão e ambiente de compilação

- **Sistema de compilação:** Buildroot.
- **Versão identificada no repositório:** `2026.08-345-ga962be2eea`.
- **Plataforma-alvo:** Raspberry Pi 4.
- **Arquitetura-alvo:** ARM64 (AArch64).
- **Ambiente de compilação:** máquina virtual Ubuntu Server no Microsoft Azure.
- **Diretório do Buildroot:** `/home/azureuser/buildroot`.
- **Imagem gerada:** `output/images/sdcard.img`.

A compilação foi realizada na máquina virtual para utilizar os recursos de processamento e memória disponíveis no ambiente de nuvem.

## 3. Configuração da plataforma

A configuração-base utilizada foi `raspberrypi4_64_defconfig`, associada ao projeto externo por meio da variável `BR2_EXTERNAL`.

O projeto externo está localizado em:

`/home/azureuser/TP-Aula08/br2-external`

O comando utilizado para carregar a configuração foi:

```bash
make O=/home/azureuser/buildroot-check \
  BR2_EXTERNAL=/home/azureuser/TP-Aula08/br2-external \
  raspberrypi4_64_defconfig
```

Esse comando prepara uma configuração de compilação no diretório informado por `O`. Antes de utilizá-lo novamente, deve-se verificar se o diretório de saída e a configuração correspondem à versão que será entregue.

## 4. Componentes relevantes

A configuração do sistema deve contemplar os seguintes componentes, conforme as opções efetivamente habilitadas no arquivo `.config`:

- **Linux embarcado:** sistema operacional destinado ao Raspberry Pi 4.
- **Docker Engine:** execução e gerenciamento de contêineres.

## 5. Integração com o projeto externo

O diretório `br2-external` permite manter configurações específicas do trabalho separadas dos arquivos originais do Buildroot.

Entre os elementos relevantes estão:

- `Config.in` e `external.mk`: integração do projeto externo com o sistema de compilação.
- `configs/raspberrypi4_64_defconfig`: configuração-base disponibilizada pelo projeto externo.
- `board/rpi4/overlay/`: arquivos incorporados ao sistema de arquivos raiz.
- `board/rpi4/overlay/etc/init.d/S99docker-flask`: script de inicialização da aplicação em contêiner.
- `board/rpi4/users.table`: tabela de usuários, caso esteja referenciada na configuração final.

O overlay permite adicionar arquivos e scripts específicos do projeto à imagem gerada, sem precisar modificar diretamente os arquivos originais do Buildroot.

## 6. Inicialização da aplicação

O script `S99docker-flask` foi destinado a automatizar a inicialização da aplicação Flask durante a inicialização do sistema.

A lógica prevista é aguardar a disponibilidade do Docker, carregar a imagem da aplicação e iniciar o contêiner. Dessa forma, a execução não deve depender de comandos manuais de carregamento e inicialização após cada reinicialização.

## 7. Imagem gerada

A imagem de disco produzida pelo processo de compilação está localizada em:

`/home/azureuser/buildroot/output/images/sdcard.img`

O arquivo representa o resultado da compilação do sistema embarcado.

## 8. Segurança

A imagem foi gerada no ambiente de compilação.
