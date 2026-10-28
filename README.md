# TP-Aula08 — Linux Embarcado com Buildroot e Docker

## 1. Identificação do projeto

- **Disciplina:** [preencher]
- **Curso:** Engenharia da Computação
- **Instituição:** Universidade Santa Cecília (Unisanta)
- **Integrantes:** [nomes completos e RAs]
- **Professor(a):** [preencher]
- **Plataforma-alvo:** Raspberry Pi 4
- **Sistema de construção:** Buildroot
- **Versão do Buildroot:** 2026.08-345-ga962be2eea

## 2. Objetivo

Desenvolver uma imagem Linux personalizada para a Raspberry Pi 4 utilizando Buildroot, com suporte ao Docker Engine e execução automática de uma aplicação web desenvolvida em Python com Flask.

A proposta é inicializar o sistema com hostname personalizado, usuário configurado durante a construção da imagem, mensagem de boas-vindas no login e serviços necessários para disponibilizar a aplicação na rede local.

A aplicação deve responder à seguinte URL:

`http://IP_DA_RASPBERRY:5000`

A página inicial deve exibir exatamente:

`Helooowww world eng. Computacional!!!!!!!`

## 3. Arquitetura

O projeto é composto pelos seguintes elementos:

1. **Buildroot:** gera o sistema Linux personalizado.
2. **Kernel Linux:** configurado para a plataforma Raspberry Pi 4.
3. **Sistema de arquivos raiz:** contém os arquivos e serviços necessários ao funcionamento do sistema.
4. **Docker Engine:** executa e gerencia os contêineres.
5. **Imagem Docker ARM64:** contém a aplicação Flask e suas dependências.
6. **Script de inicialização:** carrega a imagem Docker, quando necessário, e inicia o contêiner automaticamente.
7. **Aplicação Flask:** disponibiliza a página web na porta 5000.

O Docker publica a porta 5000 do contêiner na porta 5000 do sistema hospedeiro.

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

O diretório `br2-external/` reúne as personalizações específicas deste projeto. O diretório `flask-app/` contém o código-fonte e os arquivos utilizados na construção da imagem Docker.

O arquivo TAR da imagem Docker é incluído no overlay do sistema para permitir seu carregamento local durante a inicialização, sem depender do download da imagem pela Internet.

## 5. Configurações do sistema

| Configuração | Valor |
|---|---|
| Hostname | `agatha-linux` |
| Usuário configurado | `SE2026` |
| Diretório pessoal | `/home/SE2026` |
| Shell do usuário | `/bin/sh` |
| Porta da aplicação | `5000` |
| Nome do contêiner | `flask-tp8` |
| Imagem Docker | `flask-tp8:1.0` |
| Arquitetura da imagem Docker | ARM64 |
| Formato de saída do Buildroot | `sdcard.img` |

A senha do usuário é configurada por meio da tabela de usuários utilizada pelo Buildroot. O valor da senha não deve ser incluído neste documento nem publicado no repositório.

## 6. Pré-requisitos

Para reproduzir a construção, são necessários:

- Ambiente Linux compatível com o Buildroot.
- Código-fonte do Buildroot na versão utilizada pelo projeto.
- Git.
- Dependências de compilação exigidas pelo Buildroot.
- Espaço em disco suficiente para os fontes, ferramentas e arquivos gerados.
- Docker, caso seja necessário reconstruir a imagem da aplicação.

A compilação original deste projeto foi realizada em uma VM Ubuntu Server na Azure.

## 7. Construção da imagem Linux

Os comandos a seguir pressupõem que o Buildroot esteja em `~/buildroot` e o repositório do projeto em `~/TP-Aula08`.

### 7.1. Configurar o Buildroot

```bash
cd ~/buildroot

make BR2_EXTERNAL=/home/azureuser/TP-Aula08/br2-external \
    raspberrypi4_64_defconfig
```

Esse comando carrega a configuração específica da Raspberry Pi 4 mantida no repositório externo do projeto.

### 7.2. Compilar o sistema

```bash
cd ~/buildroot

make BR2_EXTERNAL=/home/azureuser/TP-Aula08/br2-external -j2
```

A compilação pode demorar, dependendo dos recursos disponíveis e da necessidade de baixar e compilar dependências.

### 7.3. Localizar a imagem gerada

Após uma compilação bem-sucedida, o arquivo esperado é:

```text
~/buildroot/output/images/sdcard.img
```

Para verificar o arquivo:

```bash
ls -lh ~/buildroot/output/images/sdcard.img
file ~/buildroot/output/images/sdcard.img
sha256sum ~/buildroot/output/images/sdcard.img
```

O hash SHA-256 permite verificar a integridade do arquivo. O registro da compilação realizada está em `evidence/sdcard-sha256.txt`.

Os arquivos gerados em `buildroot/output/` e a imagem `.img` não são versionados diretamente pelo Git.

## 8. Gravação do cartão SD

A gravação da imagem deve ser realizada em um cartão SD adequado à Raspberry Pi 4.

1. Copie `sdcard.img` para um computador com leitor de cartão SD.
2. Abra uma ferramenta de gravação de imagens, como o Raspberry Pi Imager ou o balenaEtcher.
3. Selecione a imagem `sdcard.img`.
4. Selecione cuidadosamente o cartão SD de destino.
5. Grave a imagem e aguarde a conclusão.
6. Insira o cartão na Raspberry Pi 4.

**Atenção:** a gravação sobrescreve os dados do dispositivo selecionado. Confirme o destino antes de iniciar.

## 9. Inicialização do sistema

Após inserir o cartão SD e ligar a Raspberry Pi 4, o sistema deverá inicializar com o hostname configurado e disponibilizar o login no console.

O usuário definido para o projeto é:

```text
Usuário: SE2026
Senha: a senha definida durante a construção da imagem
```

A senha deve ser obtida pela equipe responsável pelo projeto. Não a armazene neste README nem a publique no Git.

A mensagem personalizada de login está configurada no Buildroot.

## 10. Inicialização do Docker e da aplicação

O script `S99docker-flask` é instalado em:

```text
/etc/init.d/S99docker-flask
```

Durante a inicialização, ele:

1. Aguarda o Docker Engine ficar disponível.
2. Verifica se a imagem `flask-tp8:1.0` já está carregada.
3. Carrega o arquivo TAR local caso a imagem ainda não exista.
4. Cria e inicia o contêiner `flask-tp8` caso ele não esteja em execução.
5. Publica a porta 5000 do contêiner na porta 5000 do sistema.

O contêiner é iniciado com a política de reinicialização `unless-stopped`.

O arquivo da imagem Docker está localizado no sistema de arquivos da imagem Linux em:

```text
/opt/docker/flasf-tp8-arm64.tar
```

## 11. Aplicação Flask

O código da aplicação está em `flask-app/app.py`.

A aplicação utiliza a rota `/` e deve escutar em todas as interfaces de rede do contêiner, na porta 5000.

Resposta esperada:

```text
Helooowww world eng. Computacional!!!!!!!
```

A imagem Docker é construída para a arquitetura ARM64, correspondente à plataforma-alvo do projeto.

Para reconstruir a imagem em um ambiente que possua Docker e suporte à compilação multiplataforma:

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

Depois de gerar o arquivo, confira as permissões e atualize a cópia utilizada no overlay:

```bash
sudo chown "$USER":"$USER" docker/flasf-tp8-arm64.tar
chmod 644 docker/flasf-tp8-arm64.tar

cp docker/flasf-tp8-arm64.tar \
    br2-external/board/rpi4/overlay/opt/docker/
```

Se a imagem Docker for alterada, será necessário reconstruir a imagem Linux para incorporar a nova versão do TAR ao sistema de arquivos.

## 12. Verificação na Raspberry Pi

Após a inicialização, os comandos abaixo podem ser utilizados no console da Raspberry Pi.

### 12.1. Verificar o hostname

```bash
hostname
```

Resultado esperado:

```text
agatha-linux
```

### 12.2. Verificar o Docker Engine

```bash
docker info
docker ps
docker images
```

Deve ser possível identificar a imagem `flask-tp8:1.0` e o contêiner `flask-tp8`.

### 12.3. Verificar os logs da aplicação

```bash
docker logs flask-tp8
```

### 12.4. Verificar a resposta HTTP localmente

Se o sistema possuir `curl`:

```bash
curl http://127.0.0.1:5000
```

Resposta esperada:

```text
Helooowww world eng. Computacional!!!!!!!
```

Se `curl` não estiver disponível na imagem, utilize outro cliente HTTP disponível ou teste a partir de um computador da mesma rede.

## 13. Acesso pela rede local

Para descobrir o endereço IP atribuído à Raspberry Pi, execute:

```bash
ip addr
```

Identifique o endereço IPv4 da interface conectada à rede local.

Em outro computador ou celular conectado à mesma rede, abra:

```text
http://IP_DA_RASPBERRY:5000
```

Substitua `IP_DA_RASPBERRY` pelo endereço IPv4 real da placa.

A página deverá exibir exatamente:

```text
Helooowww world eng. Computacional!!!!!!!
```

O acesso externo depende da conexão de rede, do endereço IP atribuído e da publicação correta da porta 5000.

## 14. Evidências e testes

As evidências do projeto devem ser organizadas no diretório `evidence/`.

Até a elaboração desta documentação, foram realizadas a construção da imagem Linux na VM Azure, a construção da imagem Docker ARM64 e a verificação da resposta Flask dentro do contêiner.

A imagem `sdcard.img` foi gerada pelo Buildroot, e seu hash SHA-256 foi registrado em `evidence/sdcard-sha256.txt`.

**Pendente de validação na Raspberry Pi física:**

- Inicialização pelo cartão SD.
- Login com o usuário configurado.
- Exibição da mensagem personalizada.
- Inicialização automática do Docker.
- Carregamento da imagem Docker sem depender de download externo.
- Inicialização automática do contêiner.
- Acesso à aplicação a partir de outro dispositivo na mesma rede.
- Confirmação da resposta HTTP exata exigida pela atividade.

Os resultados desses testes deverão ser registrados em `docs/testes.md` e complementados por logs, capturas de tela ou fotografias pertinentes.

## 15. Limitações conhecidas

A compilação em uma VM Azure não substitui a validação da imagem na Raspberry Pi 4 física.

O comportamento de inicialização, a rede e a compatibilidade de todos os componentes devem ser confirmados no hardware-alvo antes de considerar a entrega integralmente validada.

## 16. Repositório e versão final

- **Repositório Git:** [inserir URL do repositório]
- **Tag da entrega:** `tp8-final` — criar após a validação final.
- **Versão final:** preencher após a conclusão dos testes no hardware.

O histórico de commits deve registrar as etapas relevantes de desenvolvimento e integração, e não somente o estado final do projeto.
