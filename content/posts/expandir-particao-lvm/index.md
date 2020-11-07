---
title: "Expandir partição LVM"
subtitle: "Seu disco LVM encheu? Hora de estender!"
description: "Post contendo os procedimentos para expandir uma partição LVM no Linux."
keywords: ["Linux", "LVM", "Virtualização"]
date: 2016-09-01T21:30:01.000-03:00
lastmod: 2016-09-02T21:30:01.000-03:00

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

Nos últimos dias, comumente venho tendo que aumentar o tamanho de partições LVM nos servidores virtuais provisionados há muito tempo. A tarefa é simples, basta compreender o funcionamento das partições LVM. Em outro post, expliquei os detalhes importantes a respeito de LVM’s.

### Cenário
No servidor em questão, possuo um disco com a capacidade de 90 GB em produção, que pode ser constatado através da listagem dos volumes físicos

```bash
pvs
  PV         VG     Fmt  Attr  PSize   PFree
/dev/sda5  ensino   lvm2 a--   89,76g       0
```

Este volume físico está atrelado ao agrupamento de volume de nome **“ensino”**, no qual possui duas partições LVM, uma para o raiz e outra para a swap.

```bash
lvs 
LV VG Attr LSize Pool Origin Data% Move Log Copy% Convert 
root ensino -wi-ao-- 89,76g swap_1 ensino -wi-ao-- 724,00m
```

### Adicionando novo disco

Adicionei um novo disco de 100 GB na VM. Agora se faz necessário criar a partição no disco utilizando o fdisk (opção n), alterar o tipo da partição para LVM (opção t e tipo 8e Linux LVM) e gravar.

Agora precisamos criar o volume físico

```bash
pvcreate /dev/sdb1 Physical volume "/dev/sdb1" successfully created
```

Ao rodar novamente o comando pvs, teremos a seguinte saída:

```bash
pvs
 PV         VG     Fmt  Attr PSize   PFree
 /dev/sda5  ensino lvm2 a--   89,76g       0
 /dev/sdb1  ensino lvm2 a--  100,00g    100g
```
### Aumentar partição LVM

Como já possuo o agrupamento de volumes “ensino”, vamos adicionar este volume físico recém criado, ao agrupamento em questão

```bash
vgextend ensino /dev/sdb1
```

Agora, se rodarmos o comando para listar os agrupamentos de volumes, teremos a seguinte saída:

```bash
vgs
  VG     #PV #LV #SN Attr   VSize   VFree
  ensino   2   2   0 wz--n- 89,76g  100g
```

Precisamos somente estender o agrupamento de volumes para adicionar a nova partição

```bash
lvextend -l 100%FREE /dev/mapper/ensino-root /dev/sdb1
```

Com o novo espaço disponível, resta somente redimensionar a partição desejada

```bash
resize2fs /dev/mapper/ensino-root
```

Pronto, está provisionado o novo espaço para a partição.

{{< admonition note "Observação:">}}
Lembrando que todos os procedimentos podem ser executados com a máquina em produção, porém, recomenda-se realizar um snapshot da VM para evitar maiores problemas.
{{< /admonition >}}

Espero ter ajudado! 
Um abraço e até a próxima 🖖