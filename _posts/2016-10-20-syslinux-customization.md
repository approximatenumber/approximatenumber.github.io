---
layout: post
title: Кастомизация меню syslinux
tags: syslinux, linux
comments: True
excerpt_separator: <!--more-->
---

Можно сделать так, чтобы меню `syslinux`\`а выглядело так, как вам хочется.


<!--more-->

1. Чтобы обеспечить возможность отображения кириллицы, необходимо в директорию с `syslinux` скопировать файл шрифта [866.psf](files/866.psf). Да, кодировка CP-866, поэтому необходимо сконвертировать файл с меню `syslinux` в 866:

{%highlight bash%}
$ iconv -t cp866 syslinux.cfg -o syslinux.cfg
{%endhighlight%}

2. Чтобы установить собственное фоновое изображение, необходимо скопировать его в ту же директорию (обратите внимание на формат изображения).

{%highlight bash%}
$ file splash.png
splash.png: PNG image data, 640 x 480, 8-bit/color RGBA, non-interlaced
{%endhighlight%}

3. Остается отредактировать файл меню загрузчика, вписав туда настройки для шрифта и изображения:

{%highlight bash%}
font 866.psf
MENU BACKGROUND splash.png
{%endhighlight%}

