# Testes, Evidências e Limitações

## 1. Objetivo

Este documento reúne as verificações realizadas durante o desenvolvimento do TP-Aula08, que integra Buildroot, Docker e uma aplicação Flask para a plataforma Raspberry Pi 4.

O objetivo é registrar os resultados observados e identificar as validações que ainda dependem de testes adicionais.

## 2. Ambiente de testes

Os testes de desenvolvimento foram realizados em uma máquina virtual com Ubuntu Server 24.04 LTS, hospedada no Microsoft Azure.

A imagem Docker da aplicação foi construída para a arquitetura ARM64, destinada ao Raspberry Pi 4.

## 3. Verificações realizadas

### 3.1. Construção da imagem Docker

A imagem `flask-tp8:1.0` foi construída utilizando Docker Buildx com a plataforma `linux/arm64`.

**Resultado registrado:** a construção foi concluída e a arquitetura da imagem foi verificada.

### 3.2. Verificação da aplicação Flask

A aplicação foi executada no contexto de testes do contêiner, e a resposta HTTP esperada foi obtida:

`Helooowww world eng. Computacional!!!!!!!`

**Resultado registrado:** a mensagem esperada foi retornada durante a verificação da aplicação.

### 3.3. Exportação da imagem

A imagem Docker foi exportada para um arquivo TAR, destinado à incorporação ao sistema de arquivos do Buildroot.

O arquivo utilizado no projeto é:

`docker/flasf-tp8-arm64.tar`

Também foi mantida uma cópia no overlay do Buildroot.

**Resultado registrado:** o arquivo foi preparado para ser incorporado à imagem Linux embarcada.

### 3.4. Configuração do Buildroot

A configuração específica do Raspberry Pi 4 foi carregada utilizando a árvore externa do projeto.

O arquivo de configuração utilizado é:

`br2-external/configs/raspberrypi4_64_defconfig`

**Resultado registrado:** a configuração foi reconhecida pelo Buildroot e os parâmetros relevantes foram verificados.

### 3.5. Geração da imagem Linux

O processo de compilação gerou o arquivo:

`output/images/sdcard.img`

O projeto também contém o arquivo `evidence/sdcard-sha256.txt`, destinado a registrar o hash SHA-256 da imagem.

**Resultado registrado:** a imagem foi gerada no ambiente de compilação. O hash pode ser recalculado para conferir a integridade do arquivo.

## 4. Evidências

As evidências associadas ao projeto incluem:

- O arquivo `evidence/sdcard-sha256.txt`, com o hash SHA-256 registrado.
- O código-fonte da aplicação Flask em `flask-app/app.py`.
- O arquivo `flask-app/Dockerfile`.
- O arquivo TAR da imagem Docker ARM64.
- O script de inicialização `S99docker-flask`.
- A configuração específica do Buildroot em `br2-external/`.

Esses arquivos permitem inspecionar a configuração e os componentes utilizados na construção do projeto.

## 5. Testes pendentes

A geração da imagem e os testes realizados no ambiente virtual não comprovam o funcionamento integral do sistema em hardware físico.

Os seguintes testes permanecem pendentes:

1. Gravar `sdcard.img` em um cartão SD compatível.
2. Inicializar o Raspberry Pi 4 utilizando o cartão preparado.
3. Confirmar a inicialização do Linux e a disponibilidade do serviço Docker.
4. Verificar se o script `S99docker-flask` é executado corretamente.
5. Confirmar o carregamento da imagem Docker a partir do arquivo TAR.
6. Verificar se o contêiner Flask inicia automaticamente.
7. Acessar a aplicação pela rede utilizando a porta HTTP configurada.
8. Registrar os resultados e eventuais mensagens de erro.

## 6. Limitações

A validação realizada no ambiente de desenvolvimento não substitui o teste da imagem no Raspberry Pi 4 físico.

O comportamento do serviço de inicialização, da rede, do Docker e do contêiner depende da configuração final do sistema e deve ser confirmado na plataforma-alvo.

A existência do arquivo `sdcard.img` comprova sua geração, mas não garante, por si só, que a imagem inicializará corretamente no hardware.

## 7. Conclusão

O desenvolvimento produziu os componentes necessários para a integração entre Buildroot, Docker e Flask, incluindo a configuração externa, a imagem Docker ARM64, o script de inicialização e a imagem Linux.

As verificações realizadas no ambiente virtual demonstraram resultados específicos de construção e execução da aplicação. A validação completa da solução embarcada depende da confirmação do funcionamento no Raspberry Pi 4 físico.
