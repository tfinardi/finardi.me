---
title: "Dupla aborgagem iSCSI No Linux"
subtitle: "Deixe sua conexão com o HBA resiliente"
description: "Post contendo os procedimentos para implementar dupla abordagem ao montar dispositivos iSCSI no Linux."
keywords: ["Storage", "iSCSI", "Linux", "LVM"]
date: 2017-09-04T22:33:27-03:00
lastmod: 2017-09-10T22:33:27-03:00

authorName: Thiago Finardi
authorEmail: tfinardi@gmail.com
authorLink: https://finardi.me/about

draft: false

tags: ["Storage", "iSCSI", "Linux", "LVM"]
categories: ["SysAdmin"]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "featured-image.png"
featuredImagePreview: "featured-image.png"

toc:
  enable: true
lightgallery: true
license: "Copyleft"
---

### Introdução

Para que possamos utilizar dupla abordagem na comunicação com o storage, é necessário configurarmos o __**MultiPath**__ no Linux.
Neste cenário, utilizaremos o __**open-iscsi**__ e o __**multipath-tools**__ para implementar a dupla abordagem.

![Diagrama demonstrativo](ISCSI_MultiPath.png#fullsize)

### Pré requisitos

* Interface de rede configurada na mesma rede que o Storage
* LUN provisionada e Host com permissão de acesso a LUN no Storage
* Node logado no portal iSCSI das controladoras (iscsiadm)

Vamos assumir que o ambiente do Storage e nodes (controladoras) já estão configuradas no host.
Podemos validar com o seguinte comando:

```bash
iscsiadm -m node
10.30.44.100:3260,1 iqn.1992-04.com.emc:cx.ckm00164900140.a2
10.30.44.101:3260,2 iqn.1992-04.com.emc:cx.ckm00164900140.b2
```

O resultado **DEVE** ser algo parecido com a saída acima. Observe que as duas controladoras estão comunicando

**Controladora A** -> 10.30.44.100

**Controladora B** -> 10.30.44.101

### Instalação do MultiPath Tools

Para instalarmos o multipath-tools no Debian/Ubuntu:
```bash
apt-get install multipath-tools
```

Caso o S.O seja CentOS:

```bash
yum install device-mapper-multipath
```

### Configuração do MultiPath Tools

Edite o arquivo /etc/multipath.conf e adicione o seguinte conteúdo:

```bash
defaults {
        find_multipaths yes
        user_friendly_names yes
}

blacklist {
}
```

Reinicie o serviço do multipathd

```bash
systemctl restart multipathd
```

Verifique se o multipathd reconheceu os nodes
```bash
multipath -ll
mpatha (360060160b2e04100241f545ce5b94bcd) dm-2 DGC,VRAID
size=100G features='2 queue_if_no_path retain_attached_hw_handler' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 4:0:0:0 sdc 8:32 active ready running
`-+- policy='service-time 0' prio=10 status=enabled
| `- 3:0:0:0 sdb 8:16 active ready running
```

{{< admonition note "Observação:">}}
Caso ele não reconheça, reinicie o host
{{< /admonition >}}

Verifique se o disco virtual foi reconhecido

```bash
fdisk -l
Disco /dev/mapper/mpatha: 100 GiB, 107374182400 bytes, 209715200 setores
Unidades: setor de 1 * 512 = 512 bytes
Tamanho de setor (lógico/físico): 512 bytes / 512 bytes
Tamanho E/S (mínimo/ótimo): 8192 bytes / 4194304 bytes
Tipo de rótulo do disco: dos
Identificador do disco: 0x14620697
 
Dispositivo              Inicializar Início       Fim   Setores Tamanho Id Tipo
 
/dev/mapper/mpatha-part1               8192 209715199 209707008    100G 8e Linux LVM
```

Este disco virtual aponta para as duas controladoras do Storage, ou seja, caso uma esteja offline, a outra assume automaticamente.

### Montando o novo disco no /etc/fstab

Para finalizarmos a configuração da dupla abordagem, basta adicionar/alterar esse novo disco para o ponto de montagem

```bash
/dev/mapper/mpatha-part1  /home    ext4    _netdev,errors=remount-ro  0  0
```

Rode o comando **__mount -a__** para remontar as partições, ou reinicie a VM.

O resultado esperado ao rodar o comando df -h é a seguinte:

```bash
/dev/mapper/mpatha-part1      98G   2M   98G  0%  /home
```

{{< admonition tip "Fique atento:" >}}
Adapte os dispositivos de acordo com a sua realidade
{{< /admonition >}}

Todos os dispositivos do MultiPath podem ser consultados em /dev/mapper

```bash
find /dev/mapper/mpatha*
/dev/mapper/mpatha
/dev/mapper/mpatha-part1 (1ª Partição do disco virtual. Caso hajam mais, o número é incrementado)
```

Simples né?

Espero ter ajudado!

Um abraço e até a próxima 🖖