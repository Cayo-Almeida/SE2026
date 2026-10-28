# TP-Aula08 — Linux Embarcado com Buildroot e Docker

## 1. Identificação

- **Curso:** Engenharia da Computação
- **Instituição:** Universidade Santa Cecília (Unisanta)
- **Matéria:** Sistemas Embarcados
- **Professor(a):** Sérgio Schina de Andrade
- **Alunos(as):** Agatha Nicole Marques Fávaro -231335; Letícia Moreira Ribero - ; Cayo - ; Felipe - ; Leonardo - 
- **Plataforma-alvo:** Raspberry Pi 4
- **Sistema de construção:** Buildroot
- **Versão do Buildroot:** `2026.08-345-ga962be2eea`
- **Aplicação web:** Python com Flask
- **Contêiner:** Docker para arquitetura ARM64

## 2. Objetivo

O objetivo deste projeto é construir uma imagem Linux personalizada para a Raspberry Pi 4 utilizando Buildroot, integrando o Docker Engine e uma aplicação web desenvolvida em Python com Flask.

O sistema foi configurado para utilizar um hostname personalizado, um usuário definido durante a construção da imagem, uma mensagem personalizada de login e um script de inicialização responsável por carregar e executar a aplicação em um contêiner Docker.

A aplicação deve ser acessível pela rede local, utilizando a porta 5000, e apresentar na página inicial a seguinte mensagem exata:

`Helooowww world eng. Computacional!!!!!!!`

## 3. Arquitetura do projeto

A solução integra os seguintes componentes:

- **Buildroot:** ferramenta utilizada para gerar o sistema Linux personalizado.
- **Kernel Linux:** configurado para a plataforma Raspberry Pi 4.
- **Sistema de arquivos raiz:** contém as configurações, os arquivos de inicialização, o usuário e os componentes necessários à execução do sistema.
- **Docker Engine:** responsável pela execução e pelo gerenciamento dos contêineres.
- **Imagem Docker ARM64:** contém a aplicação Flask e suas dependências.
- **Script de inicialização:** aguarda a disponibilidade do Docker, carrega a imagem local quando necessário e inicia o contêiner.
- **Aplicação Flask:** disponibiliza a página web na porta 5000.

A porta 5000 do contêiner é publicada na porta 5000 do sistema hospedeiro, permitindo o acesso à aplicação por meio do endereço IP da Raspberry Pi.

## 4. Estrutura do repositório

```text
TP-Aula08/
├── br2-external/
│   ├── board/
│   │   └── rpi4/
│   │       ├── users.table
│   │       └── overlay/
│   │           ├── etc/init.d/S99docker-flask
│   │           └── opt/docker/flasf-tp8-arm64.tar
│   ├── configs/
│   │   └── raspberrypi4_64_defconfig
│   ├── Config.in
│   ├── external.desc
│   └── external.mk
├── buildroot-config/
├── docker/
│   └── flasf-tp8-arm64.tar
├── docs/
├── evidence/
│   └── sdcard-sha256.txt
├── flask-app/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── scripts/
├── .gitignore
└── README.md
```

O diretório `br2-external/` contém as personalizações específicas do projeto para o Buildroot, incluindo a configuração da plataforma, a tabela de usuários e o overlay do sistema de arquivos.

O diretório `flask-app/` contém o código-fonte da aplicação e os arquivos necessários à construção da imagem Docker.

O diretório `docker/` armazena a imagem Docker exportada em formato TAR. Uma cópia desse arquivo é incorporada ao overlay para permitir o carregamento local da imagem no sistema embarcado.

O diretório `evidence/` contém o registro SHA-256 da imagem Linux gerada.

## 5. Configurações implementadas

| Parâmetro | Configuração |
|---|---|
| Plataforma-alvo | Raspberry Pi 4 |
| Arquitetura da imagem Docker | ARM64 |
| Hostname | `agatha-linux` |
| Usuário configurado | `SE2026` |
| Diretório pessoal | `/home/SE2026` |
| Shell do usuário | `/bin/sh` |
| Imagem Docker | `flask-tp8:1.0` |
| Contêiner | `flask-tp8` |
| Porta publicada | `5000:5000` |
| Arquivo de saída | `sdcard.img` |

O usuário é configurado durante a construção do sistema por meio de `users.table`. A senha é definida por hash no arquivo de configuração de usuários e não deve ser publicada no repositório.

A mensagem personalizada de login é configurada por meio das opções do Buildroot.

## 6. Construção da imagem Linux

A compilação foi realizada em uma máquina virtual Ubuntu Server na Azure.

Considerando o Buildroot em `~/buildroot` e o projeto em `~/TP-Aula08`, a configuração pode ser carregada com:

```bash
cd ~/buildroot

make BR2_EXTERNAL=/home/azureuser/TP-Aula08/br2-external \
    raspberrypi4_64_defconfig
```

Em seguida, a imagem pode ser construída com:

```bash
make BR2_EXTERNAL=/home/azureuser/TP-Aula08/br2-external -j2
```

Após a compilação bem-sucedida, o arquivo de imagem é gerado em:

```text
~/buildroot/output/images/sdcard.img
```

Para consultar o tamanho, o tipo e o hash do arquivo:

```bash
ls -lh ~/buildroot/output/images/sdcard.img
file ~/buildroot/output/images/sdcard.img
sha256sum ~/buildroot/output/images/sdcard.img
```

A imagem `sdcard.img` foi gerada durante o desenvolvimento. Seu hash SHA-256 foi registrado em `evidence/sdcard-sha256.txt`.

Os arquivos de saída do Buildroot não são versionados diretamente pelo Git. O arquivo `.gitignore` exclui `buildroot/output/`, imagens `.img`, imagens `.img.gz`, logs e arquivos temporários Python.

## 7. Gravação da imagem no cartão SD

Para utilizar a imagem na Raspberry Pi 4:

1. Disponibilize o arquivo `sdcard.img` em um computador com leitor de cartão SD.
2. Abra uma ferramenta de gravação de imagens, como o Raspberry Pi Imager ou o balenaEtcher.
3. Selecione o arquivo `sdcard.img`.
4. Selecione o cartão SD de destino.
5. Confirme a operação e aguarde a gravação.
6. Insira o cartão na Raspberry Pi 4.

**Atenção:** a gravação sobrescreve os dados do dispositivo selecionado. Verifique cuidadosamente se o cartão SD correto foi escolhido.

## 8. Inicialização do sistema e do Docker

O script de inicialização criado para o projeto está localizado em:

```text
/etc/init.d/S99docker-flask
```

Durante a inicialização, o script:

1. Aguarda o Docker Engine ficar disponível.
2. Verifica se a imagem `flask-tp8:1.0` já está carregada.
3. Carrega a imagem a partir de `/opt/docker/flasf-tp8-arm64.tar`, caso necessário.
4. Verifica se o contêiner `flask-tp8` já está em execução.
5. Cria e inicia o contêiner quando necessário.
6. Publica a porta 5000 do contêiner na porta 5000 do sistema.

O contêiner é executado com a política de reinicialização `unless-stopped`, permitindo que o Docker tente reiniciá-lo automaticamente quando necessário.

A inclusão da imagem Docker no sistema de arquivos permite que ela seja carregada localmente, sem precisar baixá-la durante a apresentação. Isso não elimina a necessidade de rede para acessar a aplicação a partir de outro dispositivo.

## 9. Aplicação Flask

O código-fonte da aplicação está em `flask-app/app.py`. A imagem Docker é construída a partir do `Dockerfile` e das dependências descritas em `requirements.txt`.

A aplicação disponibiliza a rota `/` e utiliza o endereço `0.0.0.0` na porta 5000 para aceitar conexões direcionadas ao contêiner.

A resposta esperada da página inicial é:

```text
Helooowww world eng. Computacional!!!!!!!
```

A imagem Docker foi construída para a arquitetura ARM64, correspondente à plataforma-alvo.

Para reconstruir a imagem Docker em um ambiente com Docker Buildx:

```bash
cd ~/TP-Aula08

sudo docker buildx build \
    --platform linux/arm64 \
    -t flask-tp8:1.0 \
    --load \
    ./flask-app
```

Para exportar a imagem:

```bash
sudo docker save \
    -o docker/flasf-tp8-arm64.tar \
    flask-tp8:1.0
```

Após atualizar a imagem, é necessário substituir também a cópia presente no overlay:

```bash
sudo chown "$USER":"$USER" docker/flasf-tp8-arm64.tar
chmod 644 docker/flasf-tp8-arm64.tar

cp docker/flasf-tp8-arm64.tar \
    br2-external/board/rpi4/overlay/opt/docker/
```

Caso a imagem Docker seja modificada, o Buildroot deverá ser executado novamente para incorporar o arquivo atualizado à imagem Linux.

## 10. Acesso à aplicação pela rede local

Após a inicialização da Raspberry Pi, o endereço IPv4 pode ser consultado com:

```bash
ip addr
```

Em outro dispositivo conectado à mesma rede, o acesso deverá ser realizado por meio de:

```text
http://IP_DA_RASPBERRY:5000
```

Substitua `IP_DA_RASPBERRY` pelo endereço IPv4 real da placa.

A página deverá exibir:

```text
Helooowww world eng. Computacional!!!!!!!
```

A disponibilidade do serviço depende da inicialização correta do Docker, da execução do contêiner, da configuração da rede e da publicação da porta 5000.

## 11. Verificação do funcionamento

No sistema embarcado, os comandos abaixo podem ser utilizados para inspecionar os componentes.

### Hostname

```bash
hostname
```

Resultado esperado:

```text
agatha-linux
```

### Docker

```bash
docker info
docker images
docker ps
```

### Logs do contêiner

```bash
docker logs flask-tp8
```

### Resposta HTTP local

Se houver um cliente HTTP disponível no sistema:

```bash
curl http://127.0.0.1:5000
```

A resposta esperada é:

```text
Helooowww world eng. Computacional!!!!!!!
```

Esses comandos descrevem os procedimentos de verificação a serem utilizados no hardware-alvo.

## 12. Estado atual e evidências

Durante o desenvolvimento, foram realizadas as seguintes etapas:

- Configuração do ambiente Buildroot na VM Azure.
- Configuração da plataforma Raspberry Pi 4.
- Personalização do hostname e da mensagem de login.
- Configuração do usuário por meio da tabela de usuários do Buildroot.
- Construção da imagem Docker para ARM64.
- Criação do script de inicialização automática do contêiner.
- Incorporação do arquivo TAR da imagem Docker ao overlay do sistema.
- Compilação da imagem Linux `sdcard.img`.
- Registro do hash SHA-256 da imagem gerada.
- Organização dos arquivos do projeto em um repositório Git.

A resposta exata da aplicação Flask foi verificada dentro do contêiner durante os testes realizados na VM. A imagem Docker foi identificada como `linux/arm64`.

A compilação da imagem Linux foi concluída no ambiente Azure. Entretanto, não foi realizada a validação final na Raspberry Pi 4 física.

Consequentemente, a inicialização pelo cartão SD, a execução automática dos serviços no hardware-alvo e o acesso à aplicação a partir de outro dispositivo da rede ainda não estão confirmados.

## 13. Limitações

A compilação bem-sucedida no ambiente de desenvolvimento não comprova, por si só, o funcionamento integral da imagem no hardware-alvo.

A validação física depende da disponibilidade de uma Raspberry Pi 4, de um cartão SD gravado com a imagem gerada e de uma rede local para testar o acesso à aplicação.

Os resultados documentados devem ser interpretados de acordo com o estado atual de desenvolvimento descrito nesta seção.

## 14. Repositório

O código-fonte, as configurações e os arquivos de integração do projeto são mantidos em um repositório Git.

**Repositório:** http://github.com/Cayo-Almeida/SE2026

O histórico de commits registra as etapas de desenvolvimento e integração. A versão final deve corresponder aos arquivos efetivamente entregues, sem incluir como concluídos testes que não foram executados.
