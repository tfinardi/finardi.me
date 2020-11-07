---
title: 'Tilix: Um emulador de Terminal Multi-janelas'
subtitle: 'Melhore sua produtividade no terminal Linux'
description: O Tilix é um emulador de terminal multi terminais, contando com a tão amada possibilidade de entrada simultânea em multiplos terminais (como iTerm2, Terminator  MobaXterm), o que acaba melhorando a produtividade, pois evita o copiar e colar nas abas.
keywords: ["Linux", "Terminal", "Tilix"]
date: 2018-12-07T13:19:29.000-03:00
lastmod: 2020-12-07T13:19:29.000-03:00

authorName: Thiago Finardi
authorEmail: tfinardi@gmail.com
authorLink: https://finardi.me/about

draft: false

tags: ["Terminal", "Linux", "Tilix"]
categories: ["SysAdmin"]

hiddenFromHomePage: false
hiddenFromSearch: false
resources:

featuredImage: "featured-image.png"
featuredImagePreview: "featured-image.png"

toc:
  enable: true
lightgallery: true
license: "Copyleft"
---

### Introdução

O Tilix é um emulador de terminal multi terminais, contando com a tão amada possibilidade de entrada simultânea em multiplos terminais (como iTerm2, Terminator  MobaXterm), o que acaba melhorando a produtividade, pois evita o copiar e colar nas abas. Somando a isso, a facilidade de personalização da interface e atalhos auxilia muito aqueles que assim como eu, não gostam de ficar pegando o mouse :stuck_out_tongue_winking_eye:

O projeto é open source e seu código pode ser encontrado [aqui](https://github.com/gnunn1/tilix/).

### Instalação

A instalação é super simples, basta adaptar a sua distribuição.

| Distribuição | Como instalar |
| --- | --- |
| Archlinux | pacman -S |
| CentOS 7.* | Disponível via EPEL |
| Ubuntu | Disponível para artful e bionic |
| Debian | Disponível para stretch-backports, buster e sid |

### Atalhos

Tilix já vem com um monte de atalho pré-definidos, mas estes são os que eu considero mais úteis:

* Adicionar terminal abaixo: `Ctrl+Alt+D`
* Adicionar terminal à direita: `Ctrl+Alt+R`
* Alternar para o próximo terminal: `Ctrl+Tab`
* Alternar para o terminal anterior: `Ctrl+Shift+Tab`
* Abrir as preferências: `Ctrl+Alt+S`
* Abrir uma nova sesão: `Ctrl+Shift+T`
* Abrir uma nova janela: `Ctrl+Shift+N`
* Redimensionar terminal para baixo: `Alt+Abaixo`
* Redimensionar terminal para esquerda: `Alt+Esquerda`
* Redimensionar terminal para direita: `Alt+Direita`
* Redimensionar terminal para cima: `Alt+Acim`
* Sincronizar entrada em todos os terminais `Ctrl+Espaço`

![Atalhos de sessão do Tilix](atalhos-sessão1.png#fullsize "Atalhos do Tilix - Tela 1")
![Atalhos de sessão do Tilix](atalhos-sessão2.png#fullsize "Atalhos do Tilix - Tela 2")

É possível alterar os atalhos e configurá-los do jeito que quiser, para isso basta acessar a aba `Atalhos do teclado`:
![Configurar atalhos de teclado do Tilix](tilix-atalhos.png#fullsize "Configurar atalhos de teclado do Tilix")

### Tiling (múltiplos terminais)

Tiling pode ser de grande ajuda em sua produtivade no dia a dia. Precisa logar no seu servidor via ssh? `Ctrl+Alt+R` e um novo terminal aparece na mesma janela que você já abriu! Precisa executar um comando em todos os terminais abertos simultaneamente? no meu caso, personalizei esta ação para `Ctrl+Espaço`.

![Múltiplos terminais](tiling.gif#fullsize "Múltiplos terminais com o Tilix")

### Conclusão

Pelo menos para mim, o Tilix se tornou uma excelente ferramenta, melhorando bastante minha produtividade, principalmente quando se faz necessário executar o mesmo comando em vários terminais.

Aconselho você a dar uma olhada na [Documentação Oficial](https://gnunn1.github.io/tilix-web/manual/), visitar o [repositório](https://github.com/gnunn1/tilix/) e brincar com as configurações para customizá-lo da sua maneira.

Espero ter ajudado!

Um abraço e até a próxima 🖖

##### Fontes

* [Documentação Oficial](https://gnunn1.github.io/tilix-web/manual/)
* [Repositório Oficial](https://github.com/gnunn1/tilix/)