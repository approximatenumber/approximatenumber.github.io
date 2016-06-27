---
layout: post
title: Загрузочная флешка с QNX4 и syslinux
tags: linux qnx usb
comments: True
excerpt_separator: <!--more-->
---

Инструкция, как сделать загрузочную флешку с ОС QNX4, загружаемой с помощью `syslinux` -> `memdisk`.

<!--more-->

Сделать образ файловой системы QNX4, `image.tar.bz2`, куда надо положить то, что должно быть в целевой системе. 

Путем долгих усилий удалось заставить загружаться QNX4 с файловой системы FAT, а не с qnx-fs. Это сильно экономит время, не нужно мучаться с этой ФС, писать на которую можно только из самого же QNX, и нет необходимости разбивать флешку на несколько разделов с разными ФС.

 * отредактировать сценарий prestart:
 
{%highlight bash%}
/cram/io-usb-ehci &
/cram/Fsys -r 131071 &
/cram/dinit /dev/ram
/cram/mount /dev/ram /
/cram/Fsys.umass -q fsys -n0=usb
/cram/mount -p /dev/usb0
/cram/Dosfsys -T 5 c=/dev/usb0t131
cd /
/dos/c/target/bin/bzip2 -dck /dos/c/target/image.bz2 | /dos/c/target/bin/pax -r
/dos/c/target/bin/cp -cR -M unix /dos/c/target/etc /
/kpda/bin/mkdir /tmp
/kpda/bin/slay -f Efsys.ram
/kpda/bin/sinit -i /etc/config/sinit1 TERM=qnx
{%endhighlight%}

 В данном случае мы сильно ограничены размером ядра, и если оно превышает 610 кБ (в моем случае критичный размер ядра колебался около этого значения), то ядро не загрузится. Поэтому необходимо уложиться в размер, используя минимальное количество утилит/драйверов для старта системы. `Fatfsys` оказался слишком жирным для cram-fs, поэтому использую `Dosfsys`, вместе с его ограничением на имя файла, из-за этого файлы/директории у меня называются коротко, до 9 символов (например, `image.tar.bz2` стал `image.bz2`).


* положить в `ram/` все необходимые для старта ядра файлы:
 
{%highlight bash%}
$ ls -1 ram
dinit
Dosfsys
Fsys
Fsys.umass
io-usb-ehci
mount
sh
{%endhighlight%}

 Итак, как мы видим, система будет загружаться в RAM, т.е. архив с ФС (`image.bz2`) будет распаковываться в память, что довольно безопасно.
     
 * собрать ядро, сделать образ дискеты с этим ядром (в `Makefile` есть сценарий, который сделает это сам):
 
{%highlight bash%}
$ make clean && make
{%endhighlight%}
 
  Перед этим примонтируйте пустой файл как образ дискеты (ведь вы делаете всё это в виртальной машине, так? :) ). Тогда в итоге у вас получится готовый образ дискеты, который сразу можно использовать для загрузки. Например, `qnx_fat.img`.
 
 * Теперь сделаем загрузочную флешку:
 
 {%highlight bash%}
drive=/dev/sdX
dd if=/dev/zero of=$drive bs=512 count=1 conv=notrunc  # очистим таблицу разделов
dd if=/usr/share/syslinux/mbr.bin of=$drive   # запишем MBR на накопитель
parted --script $drive mklabel msdos mkpart primary fat32 0 100  # создадим раздел размером в 100 Мб
mkfs -t vfat ${drive}1 # создадим файловую систему
syslinux ${drive}1 # установим syslinux в раздел
parted $drive set 1 boot on # сделаем раздел загрузочным
 {%endhighlight%}

*   Осталось положить нужные файлы на флешку:

{%highlight bash%}
mount /dev/sdX /mnt/flash
mkdir /mnt/flash/boot
cp -r syslinux /mnt/flash/boot
cp qnx_fat.img /mnt/flash/boot
{%endhighlight%}
    
Содержимое файла `syslinux.cfg`:

{%highlight bash%}
LABEL QNX
MENU LABEL QNX
KERNEL /boot/memdisk
APPEND initrd=/boot/qnx4_fat.img floppy
{%endhighlight%}


Архив с cram-fs доступен [здесь](/files/cram_fat.tar.gz).
