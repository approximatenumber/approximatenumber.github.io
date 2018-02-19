---
title: Как сделать бинарный RPM-пакет из готового набора файлов
date: 2016-06-27 00:00:00 Z
tags:
- linux
- rpm
layout: post
comments: true
excerpt_separator: "<!--more-->"
---

Для примера мы возьмем файл (в данном случае пустой sh-скрипт) `testapp.sh`, который находится в каталоге `/opt/testapp`. Представим, что это программа, состоящая всего лишь из одного файла, и ее необходимо распространять в виде RPM-пакета.

<!--more-->

Создаем директорию для `Source`-приложения и кладем туда наше приложение:

    user@localhost mkdir -p testapp-1.0/testapp
    user@localhost cp /opt/testapp.sh testapp-1.0/testapp/

Сделаем архив `Source`-приложения:

    user@localhost tar -xzf testapp-1.0.tar.gz testapp-1.0

Настоятельно рекомендуется собирать RPM-пакеты НЕ под учетной записью root в целях безопасности. Я собираю пакеты под учетной записью `rpmbuild`. Все дальнейшие действия выполняются под ней.

Теперь необходимо создать дерево каталогов (окружение) для сборки rpm следующего содержания:

    rpmbuild@localhost mkdir -p ~/RPMBUILD/{RPMS,SRPMS,SPECS,SOURCES,BUILD,BUILDROOT}

Переопределить макросы в файле `~/.rpmmacros`:

{%highlight bash%}
%_topdir /home/rpmbuild/RPMBUILD
%_builddir %{_topdir}/BUILD
%_rpmdir %{_topdir}/RPMS
%_sourcedir %{_topdir}/SOURCES
%_specdir %{_topdir}/SPECS
%_srcrpmdir %{_topdir}/SRPMS
{%endhighlight%}
    
    
Архив `Source`-приложения `testapp-1.0.tar.gz` должен находиться в директории `~/RPMBUILD/SOURCES`.

Самое главное - правильно заполнить файл спецификации `~/RPMBUILD/SPECS/testapp.spec`:

{%highlight bash%}
Summary: Test App
Name: testapp
Version: 1.0
Release: 1
License: Commercial
Group: Misc/Applications
Source0: %{name}-%{version}.tar.gz
Vendor: Me
Packager: rpmbuild

%description
Just an application for testing.


%prep
%setup -n %{name}-%{version}


%install

[ "$RPM_BUILD_ROOT" != "/" ] && rm -rf $RPM_BUILD_ROOT

mkdir $RPM_BUILD_ROOT
install -d testapp $RPM_BUILD_ROOT/opt/testapp
install -m 755 testapp/testapp.sh $RPM_BUILD_ROOT/opt/testapp

%clean
rm -rf $RPM_BUILD_ROOT

%files

%defattr(755,root,root)

%dir /opt/testapp
/opt/testapp/testapp.sh
{%endhighlight%}

И запустить сборку RPM-пакета:

    rpmbuild@localhost ~/RPMBUILD/SPECS $ rpmbuild -bb testapp.spec     

По умолчанию, `rpmbuild` собирает пакет под текущую архитектору. В данном случае, x86_64. 
В случае успешной сборки готовый rpm-пакет `testapp-1.0-1.x86_64.rpm` будет располагаться в каталоге `~/RPMBUILD/RPMS/x86_64`. 

Для другой архитектуры, например, i686, используйте опцию `--target i686`.

Теперь можно посмотреть информацию о нашем пакете и убедиться, что всё в порядке:

{%highlight bash%}
rpmbuild@localhost ~/RPMBUILD/RPMS/x86_64 $ rpm -qlip testapp-1.0-1.x86_64.rpm 
Name        : testapp
Version     : 1.0
Release     : 1
Architecture: x86_64
Install Date: (not installed)
Group       : Misc/Applications
Size        : 0
License     : Commercial
Signature   : (none)
Source RPM  : testapp-1.0-1.src.rpm
Build Date  : Пн 27 июн 2016 11:43:25
Build Host  : localhost
Relocations : (not relocatable)
Packager    : rpmbuild
Vendor      : Me
Summary     : Test App
Description :
Just an application for testing.

/opt/testapp
/opt/testapp/testapp.sh
{%endhighlight%}
