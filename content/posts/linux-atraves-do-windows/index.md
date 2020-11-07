---
title: "Linux atraves do Windows com WSL2, Docker e ZSH)"
subtitle: "Utilizando o Docker via WSL2 no Windows como ambiente de desenvolvimento."
description: "Ative o Windows Subsystem for Linux (WSL) 2 com suporte ao Docker e tenha o seu ambiente de desenvolvimento de uma maneira super rápida."
keywords: ["WSL", "Docker", "ZSH", "Windows"]
date: 2020-11-04T11:04:23-03:00
lastmod: 2020-11-04T11:04:23-03:00

authorName: Thiago Finardi
authorEmail: tfinardi@gmail.com
authorLink: https://finardi.me/about

draft: false

tags: ["WSL", "Docker", "ZSH", "Windows"]
categories: ["SysAdmin","DevOps"]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "featured-image.png"
featuredImagePreview: "featured-image.png"

toc:
  enable: true
lightgallery: true
license: "Copyleft"
---

Geralmente eu utilizo um Container configurado com as ferramentas e ambientes que trabalho já definidos, porém como nem sempre estou utilizando um dispositivo com Linux, além de ver o lançamento do Windows Subsystem for Linux (WSL) 2, fiquei tentado em construir o mesmo ambiente que utilizo no GNU/Linux, no Windows 10.

{{< admonition note "Nota:" >}}
WSL2 necessita que a virtualização esteja habilitada no BIOS/UEFI do seu dispositivo.
{{< /admonition >}}

### Atualize o Windows

Para utilizar o WSL2, o Windows 10 deve estar atualizado com a versão 2004 (Build 19041) ou superior.
Verifique se a versão que está utilizando atende o requisito, caso contrário, atualize.

### Habilitar o Subsistema do Windows para Linux

Antes de instalar qualquer distribuição do Linux no Windows, você precisará primeiro habilitar o recurso opcional "Subsistema do Windows para Linux".
Abra o PowerShell como administrador e execute:

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

Esse procedimento irá instalar o WSL (v1), agora após reiniciar o computador, atualize para a versão 2.

### Habilitar o recurso de Máquina Virtual (Hyper-V)

Antes de instalar o WSL 2, você precisa habilitar o recurso opcional `Plataforma de Máquina Virtual`.
Abra o PowerShell como administrador e execute:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
### Atualizar o pacote do Kernel do Linux para o WSL 2

Baixar a versão mais recente do [Pacote de atualização do kernel do Linux do WSL2 para computadores x64](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

### Definir o WSL 2 como a sua versão padrão

Abra o PowerShell e execute este comando para definir o WSL 2 como a versão padrão ao instalar uma nova distribuição do Linux:
```powershell
wsl --set-default-version 2
```

### Instalar a distribuição do Linux Ubuntu 20.04 LTS

Abra a [Microsoft Store](https://aka.ms/wslstore) e escolha sua distribuição do Linux favorita. 

![Microsoft Store](store.png#fullsize)

Neste exemplo, utilizaremos o Ubuntu 20.04 LTS.

![WSL Ubuntu 20.04 LTS](ubuntu.png#fullsize)

### Ajustando DNS do WSL
Comigo o DNS padrão que veio configurado no Ubuntu não estava resolvendo nomes, sendo assim, recomendo fortemente que você modifique para o servidor DNS público de sua preferência. Para isso, basta seguir os seguintes passos:

1. Crie o arquivo `/etc/wsl.conf` e coloque o seguinte conteúdo

```code
[network]
generateResolvConf = false
```
2. No Powershell do Windows, rode o comando abaixo para finalizar o WSL em execução

```powershell
wsl --shutdown
```
3. Abra novamente o WSL

4. Modifique o arquivo `/etc/resolv.conf` para utilizar o DNS de sua preferência
```bash
nameserver 8.8.8.8
```
5. Repita os passos 2 e 3 que o DNS estará atualizado. 

### Instalando ferramentas básicas no WSL

Este passo é opcional, mas caso você necessite de algumas ferramentas específicas em seu WSL, basta instalar. Neste exemplo, deixo instalado algumas ferramentas para diagnóstico de rede e compilação, além de modificar meu bash padrão para o ZSH.

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install build-essential net-tools dnsutils nmap -y
```

Instale o ZSH seguindo os passos descritos no post [Maior produtividade no terminal com o Oh My ZSH](https://finardi.me/produtividade-com-oh-my-zsh/).

### Instalando o Docker

A própria Docker aconselha a utilização do aplicativo [Docker para Windows](https://finardi.me/produtividade-com-oh-my-zsh/). Porém resolvi instalar o Docker diretamente no WSL, como faço no Linux. Caso você ache melhor utilizar a aplicação para Windows, basta baixar e instalar. Mas caso queira instalar o Docker dentro do WSL, basta seguir os passos abaixo.

Agora iremos fazer a instalação básica do Docker no WSL para podermos criar nossos ambientes de desenvolvimento. 

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

**Adicionando as chaves oficiais do Docker**

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

**Adicionando o Repositório Oficial do Docker**

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

**Agora basta atualizar a lista de repositórios e instalar o `docker-ce` e o `docker-compose`**

```bash
sudo apt update
sudo apt install docker-ce docker-compose -y
```

{{< admonition warning "Atenção:" >}}
Geralmente o serviço do Docker inicia automáticamente após a instalação. Comigo não foi assim, caso aconteça o mesmo com você, basta rodar o comando **``sudo service docker start``**
{{< /admonition >}}

Aproveite e adicione seu usuário ao grupo docker para possibilitar subir os containeres sem a necessidade do sudo.

```bash
 sudo usermod -a -G docker $USER
```

**Confira se o Docker está rodando lançando um container**

```bash
docker container run -dp 80:80 nginx
```
A saída deverá ser parecida com a imagem abaixo:

![Deploy do container do nginx](run-nginx.png#fullsize)

**Agora basta testar se a comunicação está funcionando lançando o browser apontado para o localhost**

![Página de boas vindas do nginx](nginx.png#fullsize)

### Concluindo

Este post surgiu principalmente da curiosidade de ver o WSL2 em funcionamento, e claro, tentar portar o meu ambiente de ferramentas Linux para dentro do Windows 10. Provavelmente não utilizarei muito, mas pelo menos já serviu para escrever este Post =).

Acabei configurando as fontes powerline dentro no VS Code, ficou tudo funcional, caso alguém tenha interesse, só chamar no Telegram que trocamos uma ideia sobre o que deve ser feito para configurar tudo certinho.

![Editor VS Code rodando no WSL](vscode.png#fullsize)

Um forte abraço e até a próxima.

##### Referências
* [Install WSL on Windows 10](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
* [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)