---
title: PXE-сервер на CentOS для сетевой загрузки за 5 минут
date: 2016-11-10 00:00:00 Z
tags:
- linux
- centos
- pxe
- syslinux
layout: post
comments: true
excerpt_separator: "<!--more-->"
---

PXE-сервер - это полезная и удобная вещь во многих случаях:

 - когда у оборудования или у вас проблемы с загрузкой иным способом;
 - когда есть N экземляров железа, N портов в коммутаторе, N патч-кордов... :)
 - когда это просто удобно!
 
<!--more-->
 
Для полноценной работы PXE-сервера необходимо три пакета: `tftp-server`, `syslinux`, `dhcp`. Поэтому:

{%highlight bash%}
$ yum install tftp-server syslinux dhcp
{%endhighlight%}

Проверьте, что в `xinetd` `tftp`-сервис включен

{%highlight bash%}
$ grep disable /etc/xinetd.d/tftp 
  disable                 = no
{%endhighlight%}

Если да, то перезапустите `xinetd`

{%highlight bash%}
$ service xinetd restart
{%endhighlight%}

Если всё хорошо, то включите сервис в автозапуск

{%highlight bash%}
$ systemctl enable xinetd
{%endhighlight%}

Создайте каталог для tftpboot и скопируйте файлы для загрузки `syslinux`:

{%highlight bash%}
$ mkdir /tftboot
$ cp /usr/lib/syslinux/gpxelinux.0 /tftpboot
$ cp /usr/lib/syslinux/mboot.c32 /tftpboot
$ cp /usr/lib/syslinux/chain.c32 /tftpboot
$ cp /usr/lib/syslinux/vesamenu.c32 /tftpboot
{%endhighlight%}

Предположим, что вы будете загружать по сети маленький GNU/Linux Slitaz, поэтому положите то, что надо, куда надо:

{%highlight bash%}
$ mkdir -p /tftpboot/images/slitaz
cp slitaz/{bzImage,rootfs.gz} /tftpboot/images/slitaz
{%endhighlight%}

Создайте каталог для конфигурационного файла

{%highlight bash%}
mkdir /tftpboot/pxelinux.cfg
{%endhighlight%}

Ну и сам конфигурационный файл (пример)

{%highlight bash%}
$ cat /tftpboot/pxelinux.cfg/default

    default vesamenu.c32
    prompt 0
    timeout 100
    ONTIMEOUT local
    MENU TITLE WELCOME TO PXE BOOT
    LABEL local
            MENU LABEL Boot local hard drive
            LOCALBOOT 0
    LABEL Slitaz
            MENU LABEL Slitaz
            KERNEL /images/slitaz/bzImage
            APPEND initrd=/images/slitaz/rootfs.gz root=/dev/null autologin rw vga=normal lang=ru_RU kmap=en user=root sound=no

{%endhighlight%}

Осталось настроить DHCP-сервер. В данном варианте приведены настройки для сервера, который ничего больше не делает, кроме как раздает IP так называемым PXEClients

{%highlight bash%}
$ cat /etc/dhcp/dhcpd.conf 

    ddns-update-style none;
    option domain-name "<your.domain>";
    option domain-name-servers 192.168.111.1;

    default-lease-time 600;
    max-lease-time 7200;
    log-facility local7;

    class "pxeclients" {
    match if substring(option vendor-class-identifier, 0, 9) = "PXEClient";
    filename "gpxelinux.0";
    }

    shared-network pxe {
    subnet 192.168.111.0 netmask 255.255.255.0 {
    }
    pool {
    allow members of "pxeclients";
    range dynamic-bootp 192.168.111.100 192.168.111.200;
    }
    }
    {%endhighlight%}

Если в конфигурации всё ОК, то запустим/перезапустим и добавим в автозапуск DHCP-сервис
    
{%highlight bash%}
$ service dhcp restart
$ systemctl enable dhcp
{%endhighlight%}

Теперь можно загрузиться по сети для проверки и проверить, что всё работает

![PXE](/images/pxe.png)


