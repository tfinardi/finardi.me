---
title: "Montagem de Partiçõess iSCSI no GNU/Linux"
subtitle: "Tenha sua LUN montada no seu FileSystem"
description: "Post contendo os procedimentos para montagem de dispositivos iSCSI no Linux."
keywords: ["Storage", "iSCSI", "Linux", "LVM"]
date: 2017-02-03T22:24:28-03:00
lastmod: 2017-02-03T22:24:28-03:00

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

O GNU/Linux possui um módulo iSCSI initiator no kernel que toma o local e forma de um driver SCSI HBA, e portanto, permite a utilização de discos iSCSI. No entanto, antes do Linux poder usar um alvo iSCSI, ele deve encontrar este alvo na rede e fazer uma conexão com ele.

A descoberta, conexão e login é realizada através do utilitário **_iscsiadm_**, enquanto os erros são tratados pelo utilitário **_iscsid_**.
Ambos, iscsiadm e iscsid são parte do pacote **open-iscsi no Debian e derivados** e **iscsi-initiator-utils no Red  Hat e CentOS.**

### Montagem da LUN na GNU/Linux

O procedimento de montagem da LUN no GNU/Linux é relativamente simples. Para isso, precisamos do iSCSI initiator tools que é provido pelo pacote **open-iscsi no Debian/Ubuntu** e **iscsi-initiator-utils no CentOS**.

No exemplo, foi utilizada uma VM contendo um Servidor SFTP para armazenamento dos Backups de configuração dos Ativos de Rede (Roteadores, Switches, vCenter, CallManager, Etc).

A VM em questão possui duas interfaces de rede, sendo uma na VLAN 108 (tráfego da rede de acesso) e a outra na VLAN 344 (que é utilizada para comunicação com o Storage).

Veja o exemplo da configuração de rede abaixo:

**Interface VLAN 108 (Tráfego VM)**

```bash
ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
       inet 10.0.0.16  netmask 255.255.255.0  broadcast 10.0.0.255
       inet6 fe80::250:56ff:fea0:d3c5  prefixlen 64  scopeid 0x20<link>
       ether 00:50:56:a0:d3:c5  txqueuelen 1000  (Ethernet)
       RX packets 827929  bytes 77386599 (73.8 MiB)
       RX errors 0  dropped 2357  overruns 0  frame 0
       TX packets 47937  bytes 4941011 (4.7 MiB)
       TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

**Interface VLAN 344 (Tráfego iSCSI)**

```bash
ens34: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
       inet 10.30.44.16  netmask 255.255.252.0  broadcast 10.30.47.255
       inet6 fe80::250:56ff:fea0:bacb  prefixlen 64  scopeid 0x20<link>
       ether 00:50:56:a0:ba:cb  txqueuelen 1000  (Ethernet)
       RX packets 50532  bytes 17937305 (17.1 MiB)
       RX errors 0  dropped 55  overruns 0  frame 0
       TX packets 73641  bytes 5959229 (5.6 MiB)
       TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### Pré requisitos

1. Possuir Hostname único configurado na rede;
2. Ser possível a comunicação com o IP/nome do equipamento de Storage;
3. A LUN já deve estar provisionada e com permissão de acesso ao host que deseja montar ela;
4. pacote open-iscsi no Debian/Ubuntu ou iscsi-initiator-utils no CentOS.

{{< admonition note "Observação:">}}
Caso você utilize apenas uma interface, basta ignorar a informação das interfaces, pois o importante é conseguir se comunicar com o storage.
{{< /admonition >}}

### Instalação do iscsi-initiator-utils

```bash
apt-get install open-iscsi -y  (Debian/Ubuntu)
ou
dnf install -y iscsi-initiator-utils  (Red Hat/CentOS)
```

Confirme o hostname único e o initiator da VM no arquivo __**/etc/iscsi/initiatorname.iscsi**__
```bash
InitiatorName=iqn.1993-08.org.sftp-bkp:01:b04d8b058cb
```

{{< admonition note "Observação:">}}
Caso seja necessário, altere para outro nome. Lembrando que este nome não pode existir em outro host no Storage.
{{< /admonition >}}

Execute a descoberta dos targets (controladoras) do Storage através do comando

```bash
iscsiadm -m discovery -t st -p 10.30.44.100
iscsiadm -m node --loginall all
```

{{< admonition note "Observação:">}}
Deste modo, os nodes serão inicializados automaticamente com o sistema
{{< /admonition >}}

Confirme se a autenticação foi realizada com sucesso. A saída do comando acima deverá ser parecida com:

```bash
Logging in to [iface: default, target: iqn.1992-04.com.emc:cx.ckm00164900140.a2, portal: 10.30.44.100,3260] (multiple)
Logging in to [iface: default, target: iqn.1992-04.com.emc:cx.ckm00164900140.b2, portal: 10.30.44.101,3260] (multiple)
Login to [iface: default, target: iqn.1992-04.com.emc:cx.ckm00164900140.a2, portal: 10.30.44.100,3260] successful.
Login to [iface: default, target: iqn.1992-04.com.emc:cx.ckm00164900140.b2, portal: 10.30.44.101,3260] successful.
```

Altere o padrão do __**/etc/iscsi/iscsid.conf**__ para iniciar automaticamente:
```bash
node.startup = automatic
```

Para testar, reinicie o serviço do open-iscsi
```bash
systemctl restart open-iscsi
```

Verifique se os discos foram reconhecidos na VM com o fdisk
```bash
fdisk -l

Disco /dev/sdc: 100 GiB, 107374182400 bytes, 209715200 setores
Unidades: setor de 1 * 512 = 512 bytes
Tamanho de setor (lógico/físico): 512 bytes / 512 bytes
Tamanho E/S (mínimo/ótimo): 8192 bytes / 4194304 bytes
Tipo de rótulo do disco: dos
Identificador do disco: 0x14620697

Disco /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 setores
Unidades: setor de 1 * 512 = 512 bytes
Tamanho de setor (lógico/físico): 512 bytes / 512 bytes
Tamanho E/S (mínimo/ótimo): 8192 bytes / 4194304 bytes
Tipo de rótulo do disco: dos
Identificador do disco: 0x14620697
```
{{< admonition tip "Fique atento:" >}}

Cada controladora é representada por um disco, neste exemplo, os discos SDB e SDC são a mesma LUN, ou seja, independente de qual disco seja montado no sistema, os dados serão os mesmos.
Vale salientar que este é apenas um apontamento para a LUN.

{{< /admonition >}}

### Criando a partição para armazenar os dados na LUN

Como a LUN do exemplo é nova, ou seja, está vazia, iremos criar uma partição do tipo Linux LVM (id 8e) utilizando o fdisk.

{{< admonition warning "Atenção:" >}}
  Execute somente em uma LUN nova, sem dados
{{< /admonition >}}
Obs: Caso a LUN seja maior do que 2TB, utilize o padrão GPT com o gdisk no lugar do fdisk

Para isso, execute o comando fdisk e na ordem informe:
```bash
fdisk  /dev/sdb
n = nova partição
p = primária
nº 1 = padrão
Setor inicial = padrão
Setor final = padrão
t = Tipo de partição
8e = Linux LVM
p = listar partições
```

A saída deve ser parecida com:
```bash
/dev/sdb1                 8192 209715199 209707008    100G 8e Linux LVM
```

Salve a configuração com a tecla w

### Formatando a partição criada

Neste exemplo, a partição utilizará o sistema de arquivos EXT4. Caso deseje utilizar outro tipo, basta adaptar.
```bash
mkfs.ext4 /dev/sdb1
```

### Montar a partição no fstab
{{< admonition tip "Fique atento:" >}}
Recomendo utilizar o multipath para possuir dupla abordagem na comunicação com o storage.
{{< /admonition >}}

edite o arquivo __**/etc/fstab **__e adicione o seguinte conteúdo, adaptando para os seus discos e ponto de montagem.
```bash
/dev/sdb1 /home      ext4    _netdev,errors=remount-ro  0  0
```

Neste exemplo, será montada a partição do disco de uma controladora (/dev/sdb1) no diretório /home.

{{< admonition note "Observação:" >}}
Saliento que este tipo de montagem não possui dupla abordagem, ou seja, caso a controladora falhe, o disco ficará somente leitura.
{{< /admonition >}}

Para finalizar a montagem, execute o comando:
```bash
mount -a
```

Para validar a configuração, basta executar o comando df -h para verificar se o disco montou.
```bash
/dev/sdb1      98G   4M   98G  0%  /home
```

Simples né?

Espero ter ajudado!

Um abraço e até a próxima 🖖