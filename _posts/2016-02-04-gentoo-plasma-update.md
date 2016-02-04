---
layout: post
title: Обновление KDE до Plasma в Gentoo
tags: linux; gentoo; plasma; kde
---

При попытке обновления KDE в Gentoo до Plasma могут возникнуть некоторые... проблемы. Опишу некоторые моменты в ходе установки на своей системе.

Следуя [gentoo-wiki](https://wiki.gentoo.org/wiki/KDE/Plasma_5_upgrade) и полезному [сообщению с forums.gentoo.org](https://forums.gentoo.org/viewtopic-p-7842836.html#7842836), я наткнулся на проблему с `qtcore`:
{%highlight sh%}
!!! Multiple package instances within a single package slot have been pulled
!!! into the dependency graph, resulting in a slot conflict:
    dev-qt/qtcore:5  
    (dev-qt/qtcore-5.4.2:5/5::gentoo, installed) pulled in by  
    ~dev-qt/qtcore-5.4.2 required by (dev-qt/qtdbus-5.4.2:5/5::gentoo, installed)  
    ^       ^^^^^                                                          
    (and 10 more with the same problem)  
    (dev-qt/qtcore-5.5.1:5/5::gentoo, ebuild scheduled for merge) pulled in by  
    >=dev-qt/qtcore-5.5.1:5 required by (dev-qt/qtpaths-5.5.1:5/5::gentoo, ebuild scheduled for merge)  
    ^^       ^^^^^^^                                                                             
    (and 3 more with the same problem)  
{%endhighlight%}

Пакет `qtcore` двух различных версий пытается встать в один слот, что некошерно. Попытки собрать `qtcore` определенной версии, удаление установленных пакетов, но которые тянут за собой разные версии `qtcore`, не увенчались успехом. В итоге, как это часто бывает, помог совет emerge, ведь необходимо всегда читать вывод `emerge`, а лучше еще и понимать, о чем там речь. А он советовал попробовать собрать пакеты с опцией `--backtrack=30`, что помогло при обновлении `world`, и системой собрался `qtcore-5.5.1`.

Далее сборка мира прервалась на `kdelibs4support`:
{%highlight sh%}
Makefile:127: recipe for target 'all' failed 
{%endhighlight%}

С этим помог совет с [bugs.gentoo.org](https://bugs.gentoo.org/show_bug.cgi?id=563758), проблема решилась обновлением `qtdesigner` до версии 5.5.1.

Далее, путем удаления еще парочки блокирующих пакетов от KDE4, удалось прийти к успеху с `emerge -uDN @world`, а затем и `emerge plasma-desktop`.  Остается только разобраться с [SDDM](https://wiki.gentoo.org/wiki/SDDM), но, впрочем, в моем случае с этим никаких проблем не возникло.