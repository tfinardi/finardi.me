---
title: "Criando partições LVM manualmente no Debian e derivados"
subtitle: "Utilizando o Docker via WSL2 no Windows como ambiente de desenvolvimento."
description: "Post contendo os procedimentos para criação de partições LVM manualmente no Debian e derivados."
keywords: ["Linux", "LVM", "Virtualização"]
date: 2016-07-23T21:30:01-03:00
lastmod: 2016-07-27T21:30:01-03:00

authorName: Thiago Finardi
authorEmail: tfinardi@gmail.com
authorLink: https://finardi.me/about

draft: false

tags: ["Linux", "LVM", "Virtualização"]
categories: ["SysAdmin"]

featuredImage: "featured-image.png"
featuredImagePreview: "featured-image.png"

toc:
  enable: true
lightgallery: true
license: "Copyleft"
---

### Introdução:
Durante uma aula do componente de Virtualização no módulo de Redes do Curso Técnico em Informática do Senac, surgiu uma dúvida de um aluno. Ele queria saber como criar manualmente partições virtuais em um dispositivo de armazenamento especializado que fosse posteriormente adicionado ao Hypervisor Xen.

Vejamos então como realizar o procedimento de criação de partições virtuais em um novo disco adicionado ao servidor. Mas antes, vale relembrar alguns detalhes importantes a respeito deste processo.

### O que é uma LVM?

O **LVM (Logical Volume Manager)** é um recurso que permite criar volumes lógicos (um particionamento virtual). Com o LVM é possível, por exemplo, criar mais de 15 partições em discos SCSI e SATA que ainda utilizam o padrão BIOS e não GPT. Isto é muito útil quando utilizamos dispositivos de armazenamento especializados como NAS, DAS, SAN, Etc. Também podemos fazer com que uma partição ocupe dois ou mais discos diferentes.

### O LVM funciona com três elementos básicos:

* **PV (Physical Volume ou Volume Físico)** – É a definição das partições ou áreas do disco (ou discos) que ficarão disponíveis para serem utilizadas pelo LVM.

* **VG (Volume Group ou Agrupamento de volumes)** – É uma subdivisão do PV, algo como uma partição estendida. Pode ser utilizada para criar sistemas de arquivos maiores do que a limitação física de um disco rígido, como por exemplo, uma partição de 2TB em um HD utilizando o MBR (em discos utilizando GPT esta limitação não existe).

* **LV (Logical Volume ou Volume Lógico)** – É como se fosse uma partição lógica dentro de uma partição estendida (VG). Esse elemento é uma área de alocação na qual criamos o filesystem. Ao criarmos um volume lógico, recebemos um dispositivo para referenciarmos, ao criar ou manipular, o sistema de arquivos.

### Implementação do LVM
Neste exemplo, utilizaremos dois discos de 1GB cada, adicionados a uma máquina virtual rodando Debian GNU/Linux. A ideia será criar partições LVM e adicioná-las ao FSTAB para serem utilizadas pelo sistema.

#### Instalando o LVM

A primeira medida necessária será a instalação do pacote responsável por prover o LVM no Debian. O pacote responsável por tal funcionalidade é o lvm2. Para isso, executemos o comando:

```bash
root@BotecoDigital:/# aptitude install lvm2
``` 
#### Criação dos volumes lógicos

Conforme comentei acima, adicionei dois discos de 1GB cada. Vejamos como estão as mesmas no /dev:

```bash
root@BotecoDigital:/# ls /dev/sd?
/dev/sda /dev/sdb /dev/sdc
```

Podemos verificar que os discos adicionados ao host foram o sdb e sdc. Como os discos estão vazios, crie as partições conforme a sua necessidade. Neste exemplo, criarei apenas uma partição em cada disco. Vejamos como ficou:

```bash
root@BotecoDigital:/# ls /dev/sd*
/dev/sda /dev/sda1 /dev/sda2 /dev/sdb /dev/sdb1 /dev/sdc /dev/sdc1
```
Com as partições que receberão os volumes lógicos desmontadas, criaremos o volume físico (PV) com o seguinte comando:

```bash
root@BotecoDigital:/# pvcreate /dev/sdb1 /dev/sdc1
Writing physical volume data to disk "/dev/sdb1"
Physical volume "/dev/sdb1" successfully created
Writing physical volume data to disk "/dev/sdc1"
Physical volume "/dev/sdc1" successfully created
```

Conforme podemos verificar, a escrita foi concluída com sucesso. Para constatar tal informação, podemos consultar o PV criado com o comando pvs:

```bash
root@BotecoDigital:/# pvs
PV          VG    Fmt    Attr   Psize     PFree
/dev/sdb1         lvm2   a--    1023,00m  1023,00m
/dev/sdc1         lvm2   a--    1023,00m  1023,00m
```

A seguir, criaremos um agrupamento de volumes (VG), denominado “boteco” utilizando as duas partições com o seguinte comando:

```bash
root@BotecoDigital:/# vgcreate boteco /dev/sdb1 /dev/sdc1
Volume group "boteco" successfully created
```

Neste comando adicionamos os dois volumes físicos (PV) definidos anteriormente em um único agrupamento, ou seja, agora passamos a ter um VG com aproximadamente 2GB. Podemos verificar esta informação com o comando vgs:

```bash
root@BotecoDigital:/# vgs
VG     #PV   #LV   #SN   Attr   Vsize   VFree
boteco  2     0     0    wz--n- 1,99g   1,99g
```

A partir de agora, você pode criar os volumes lógicos (LV) que desejar. Para criar um volume com 500 MB, com o nome LVM1 dentro do agrupamento (VG) boteco, utilizaremos o seguinte comando:

```bash
root@BotecoDigital:/# lvcreate -L500M -n LVM1 boteco
Logical volume "LVM1" created
```

Para verificar o volume criado podemos rodar o comando lvs:

```bash
root@BotecoDigital:/# lvs
LV     VG     Attr     Lsize Pool Origin Data% Move Log Copy% Convert
LVM1   boteco -wi-a--- 500,00m
```

#### Utilização dos volumes lógicos
Para ser utilizado, cada um dos volumes lógicos deverá ser formatado. Lembre-se de que, geralmente, formatamos os dispositivos existentes em /dev, como /dev/sda1, por exemplo. Quando geramos o VG chamado boteco, foi criado automaticamente o diretório /dev/boteco. Quando criamos o nosso LV LVM1, foi criado o arquivo /dev/boteco/LVM1. Veja:

```bash
root@BotecoDigital:/# ls /dev/boteco
LVM1
```

Ou seja, a partir de agora podemos formatar o dispositivo /dev/boteco/LVM1 como se fosse uma partição comum.

```bash
root@BotecoDigital:/# mkfs.ext4 /dev/boteco/LVM1
```

Agora podemos montar a partição em um diretório e utilizarmos sem maiores problemas. Uma forma de deixar esta operação permanente é adicioná-la ao FSTAB, para isto, basta adicionarmos a seguinte linha em /etc/fstab tendo previamente o diretório de destino criado, em nosso exemplo, o diretório /mnt/part1.

```bash
/dev/boteco/LVM1 /mnt/part1 auto defaults 0 2
```

### Outras possibilidades
Existem diversas possibilidades para manipulação de volumes lógicos. O comando lvm help mostrará um resumo de todas as opções.

### Concluindo
O LVM é um recurso fácil de ser utilizado e que proporciona grande flexibilidade ao administrador do sistema. É possível por exemplo, adicionar mais Hds na máquina e aumentar o tamanho de um volume, sem que haja perda de dados.

Espero ter ajudado! 
Um abraço e até a próxima 🖖