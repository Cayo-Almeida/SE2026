# Aplicação Flask e Integração com Docker

## 1. Objetivo

Este documento descreve a preparação da aplicação Flask, a construção da imagem Docker para a arquitetura ARM64 e a integração do contêiner ao sistema Linux embarcado desenvolvido com Buildroot.

## 2. Aplicação Flask

A aplicação está localizada no diretório `flask-app/`, que contém os arquivos necessários para sua construção:

- `app.py`: código-fonte da aplicação.
- `Dockerfile`: instruções para construir a imagem Docker.
- `requirements.txt`: dependências Python da aplicação.

A aplicação disponibiliza uma resposta HTTP para verificar seu funcionamento.

Mensagem esperada:

`Helooowww world eng. Computacional!!!!!!!`

## 3. Construção da imagem Docker

A imagem foi construída no ambiente Ubuntu Server da máquina virtual do Microsoft Azure, especificando a arquitetura ARM64 para o Raspberry Pi 4.

O comando utilizado foi:

```bash
sudo docker buildx build \
  --platform linux/arm64 \
  -t flask-tp8:1.0 \
  --load ./flask-app
```

Os parâmetros utilizados têm as seguintes funções:

- `docker buildx build`: inicia a construção da imagem.
- `--platform linux/arm64`: define a arquitetura-alvo ARM64.
- `-t flask-tp8:1.0`: atribui nome e versão à imagem.
- `--load`: carrega a imagem construída no armazenamento local do Docker.
- `./flask-app`: indica o diretório que contém os arquivos da aplicação.

Durante a preparação do ambiente, foi necessário configurar o suporte à execução de instruções de outras arquiteturas utilizando QEMU e binfmt.

## 4. Verificação da imagem

Após a construção, a imagem foi identificada como `flask-tp8:1.0`, com arquitetura `linux/arm64`.

A imagem foi exportada para um arquivo TAR para permitir sua incorporação ao sistema embarcado sem depender do download da imagem durante a inicialização.

O arquivo exportado é:

`docker/flasf-tp8-arm64.tar`

O nome do arquivo deve ser mantido consistente com o utilizado nos scripts e no overlay do Buildroot.

## 5. Integração com o Buildroot

O arquivo TAR foi incorporado ao overlay do sistema de arquivos:

`br2-external/board/rpi4/overlay/opt/docker/flasf-tp8-arm64.tar`

O overlay também contém o script de inicialização:

`br2-external/board/rpi4/overlay/etc/init.d/S99docker-flask`

Esse script foi preparado para aguardar a disponibilidade do Docker, verificar se a imagem está carregada e iniciar o contêiner da aplicação.

O serviço utiliza o nome `flask-tp8` para o contêiner e configura a política de reinicialização `unless-stopped`.

A publicação da porta `5000` permite disponibilizar a aplicação por meio da porta correspondente do sistema embarcado.

## 6. Inicialização automática

O script `S99docker-flask` integra a inicialização do serviço ao sistema de inicialização do Linux.

Em termos gerais, o procedimento consiste em:

1. Aguardar que o serviço Docker esteja disponível.
2. Verificar se o Docker está operacional.
3. Carregar o arquivo TAR caso a imagem `flask-tp8:1.0` ainda não esteja disponível.
4. Iniciar o contêiner com o nome configurado e a política de reinicialização definida.

A execução efetiva desse procedimento em um Raspberry Pi 4 físico depende da inicialização correta do sistema, da disponibilidade do Docker e da validação do serviço no hardware.

## 7. Testes realizados

Durante a validação no ambiente de desenvolvimento, foram realizados testes da imagem Docker e da resposta da aplicação.

A mensagem esperada foi obtida durante a verificação da aplicação no contêiner. Também foi verificada a arquitetura ARM64 da imagem.

Esses resultados demonstram a construção da imagem e o funcionamento da aplicação no contexto dos testes realizados. Não comprovam, isoladamente, a inicialização completa do serviço em um Raspberry Pi 4 físico.

## 8. Considerações finais

A integração entre Flask, Docker e Buildroot permite organizar a aplicação em um contêiner e incorporar sua imagem ao sistema Linux embarcado.

A utilização de um arquivo TAR local possibilita que a imagem seja carregada no sistema-alvo sem precisar baixá-la de um registro externo durante a inicialização.

A validação final deve incluir, quando possível, a inicialização do sistema no Raspberry Pi 4 e a verificação do acesso HTTP à aplicação pela rede.
