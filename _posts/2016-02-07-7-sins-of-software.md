---
layout: post
title: Семь смертных грехов проекта
tags: bugaenko translation dev
comments: True
excerpt_separator: <!--more-->
---

<small>*Перевод статьи Е. Бугаенко [«Seven Deadly Sins of a Software Project»](http://www.yegor256.com/2015/06/08/deadly-sins-software-project.html).*</small>

*Поддерживаемость* (maintainability) – [самое ценное свойство](http://www.yegor256.com/2014/10/26/hacker-vs-programmer-mentality.html) современной разработки ПО. [Поддерживаемость](https://en.wikipedia.org/wiki/Maintainability) определяется рабочим временем, которое требуется для нового разработчика, чтобы вникнуть в ПО перед тем, как он сможет делать в нем серьезные изменения. Чем больше это время, тем ниже степень. В некоторых проектах этот временно́й параметр приближается к бесконечности, и это означает, что эти проекты буквально невозможно поддерживать. Я считаю, что существует 7 главных **«грехов»**, которые лишают наше ПО возможности поддержки. И вот они.

<!--more-->

## Антипаттерны 
<img src="/images/software-project-sins-1.gif" class="sticker">

К сожалению, языки программирования, которые мы используем, слишком *гибкие*. Они очень много всего позволяют, но мало что запрещают. Например, Java непротив того, чтобы вы поместили целое приложение в один единственный «класс» с несколькими тысячами методов. С технической точки зрения, приложение скомпилируется и запустится. Но это представляет собой известный антипаттерн [«божественный объект»](https://en.wikipedia.org/wiki/God_object).

Таким образом, [антипаттерн](https://en.wikipedia.org/wiki/Anti-pattern) это технически правильный способ проектирования чего-либо, который в общем считается неправильным. В каждом языке существует много антипаттернов. Их наличие в вашем продукте похоже на опухоль в живом организме. Как только она начинает расти, ее очень трудно остановить. Со временем весь организм погибает. Со временем всё ваше ПО становится неподдерживаемым, и оно должно быть переписано.

Как только вы пустите парочку анти-паттернов к себе в проект, вы будете со временем получать их всё больше и больше, и "опухоль" лишь будет расти.

Это особенно касается языков ООП (Java, C++, Ruby и Python), потому что они так много наследуют от процедурных языков (C, Fortran и COBOL). И потому, что ООП-разработчики предпочитают думать процедурным и императивным способом, к сожалению.

Кстати, в дополнение к существующему [списку известных антипаттернов](https://en.wikipedia.org/wiki/Anti-pattern), я также добавлю [немного от себя](http://www.yegor256.com/2014/09/10/anti-patterns-in-oop.html), которые я рассматриваю скорее как плохой подход к программированию.

Моё практическое предложение в таком случае состоит в том, что вам необходимо читать и изучать литературу. Возможно, [эти книги](http://www.yegor256.com/2015/04/22/favorite-software-books.html) вам помогут. Всегда старайтесь скептически относиться к качеству вашего ПО, и не расслабляйтесь, когда оно "просто работает". Как и с раком: чем раньше вы диагностируете его, тем выше шанс остаться живым.


## Непрослеживаемые изменения
<img src="/images/software-project-sins-2.gif" class="sticker">

Когда я смотрю на историю коммитов, у меня должна быть возможность выяснить в каждом конкретном случае изменения, *что* было изменено, *кто* это сделал и *почему*. Более того, время, необходимое для получения трех этих ответов, должно измеряться в секундах. В большинстве проектов это не так. Вот несколько практических рекомендаций:
    
**Всегда используйте тикеты.** Без разницы, насколько мал проект или его команда, даже если это вы сами, создавайте тикеты ("issues" в GitHub) для каждой проблемы, которую вы решаете. Кратко опишите проблему в тикете и напишите там свое мнение. Пишите всё, что может иметь смысл в будущем, когда кто-то еще попытается понять, о чем эти "странные коммиты".

**Ссылайтесь на тикеты в коммитах.** Не стоит и говорить, каждый коммит должен иметь своё сообщение. Коммиты без сообщений — очень плохая практика, я даже не буду это обсуждать. Но просто лишь сообщения недостаточно. Каждое сообщение должно начинаться с номера тикета, над которым вы работаете. GitHub (я уверен, вы им пользуетесь) будет автоматически связывать коммиты и тикеты, улучшая прослеживаемость изменений.

**Ничего не удаляйте.** Git позволяет делать "принудительный" push, который перезаписывает весь branch, который был до этого на сервере. Это пример того, как вы можете удалить историю разработки. Я много раз видел таких людей, которые удаляли свои комментарии в обсуждениях GitHub, чтобы их тикеты выглядели более "чистыми". Это неправильно. Никогда ничего не удаляйте; позволяйте вашей истории оставаться с вами, без разницы, насколько плохо (или беспорядочно) она сейчас для вас выглядит.

## Неадекватные релизы
<img src="/images/software-project-sins-3.gif" class="sticker">

Каждая часть ПО должна быть запакована, прежде чем она будет предоставлена конечному пользователю. Если это Java-библиотека, она должна быть запакована как `.jar`-файл и опубликована в каком-либо репозитории; если это веб-приложение, его необходимо "развернуть" на какой-либо платформе, и т.д. Без разницы, насколько мал или огромен продукт, необходимо, чтобы существовала стандартизированная процедура, которая занимается тестированием, "упаковкой" и развертыванием. 

Идеальное решение — автоматизировать эту процедуру, чтобы можно было выполнить ее из командной строки с помощью одной команды, например:

{%highlight bash%}
$ ./release.sh
...
Выполнено (за 98,7 с)
{%endhighlight%}

Большинство проектов далеки от этого. Их процесс релиза всегда включает в себя немного магии, где ответственный за это (также известный как DevOp) должен нажимать на какие-то кнопки тут и там, входить куда-то, проверять какие-то значения, и т.д. И такой процесс *неадекватного* релиза — это всё ещё типичный недостаток всей инженерной индустрии ПО.

Я могу лишь дать один практический совет: автоматизируйте это. Я использую [rultor.com](http://www.yegor256.com/2014/09/11/deployment-script-vs-rultor.html) для этого, но вы можете использовать любые инструменты, которые вам нравятся. Важно, что вся эта процедура была полностью автоматизирована и могла быть запущена из командной строки. 

## Добровольный статистический анализ
<img src="/images/software-project-sins-4.gif" class="sticker">

Использование [статистического анализа](https://en.wikipedia.org/wiki/Static_program_analysis) позволяет нашему коду выглядеть лучше. Делая это, мы неизбежно позволяем ему и *работать*         лучше. Но это происходит только тогда, когда вся команда вынуждена (!) следовать правилам, созданным статистическим аналитиком (аналитиками). Я написал об этом в статье ["Строгий контроль качества Java-кода"](http://www.yegor256.com/2014/08/13/strict-code-quality-control.html). Я использую [qulice.com](http://www.qulice.com/) в Java-проектах и [rubocop](https://github.com/bbatsov/rubocop) в Ruby, но почти для каждого языка существует множество похожих инструментов.

Вы можете использовать любой из них, но обязательно это делайте. В большинстве проектов, где используется статистический анализ, разработчики просто делают симпатично выглядящие отчеты и продолжают писать код так, как делали это раньше. Такой "добровольный" подход не сделает никаких одолжений проекту. Более того, он создает иллюзию качества.

Я говорю о том, что статистический анализ должен быть обязательным шагом в системе развертывания. Сборка проекта не должна пройти, если какое-либо из правил статистического анализа нарушено.


## Неизвестная степень покрытия тестами
<img src="/images/software-project-sins-5.gif" class="sticker">

Скажем просто, что [покрытие тестами](https://en.wikipedia.org/wiki/Code_coverage) — это значение, характеризующее состояние ПО, прошедшего модульное и интеграционное тестирование. Чем больше степень покрытия, тем больший "объем" кода был исполнен во время прохождения тестов. Очевидно, что большая степень покрытия это хорошо.

Однако, многие разработчики проектов просто не знают этой степени покрытия. Они просто не учитывают эту метрику. Они используют какие-то тесты, но никто не знает, насколько глубоко они проникают в проект, а какие его части не протестированы вообще. Такая ситуация намного хуже той, когда у проекта малая степень покрытия тестами, которая известна и доступна каждому.

Высокая степень покрытия — это не гарантия качества, и это очевидно. Но неизвестная степень покрытия это явный знак того, что существуют проблемы с поддержкой кода. Когда новый разработчик попадает в проект, он должен иметь возможность изменять что-либо и понимать, на что эти изменения влияют. Самое лучшее, когда степень покрытия тестами необходимо проверять таким же, как и статический анализ, способом, и сборка должна прерваться в случае наступления заранее определенного порога (обычно около 80%).

## Непрерывная разработка
<img src="/images/software-project-sins-6.gif" class="sticker">

Под словом "непрерывная" я имею в виду разработку без определенных стадий и релизов. Без разницы, какой именно софт вы пишите, вы должны выпускать релизы и [контролировать их версии](http://semver.org/). Проект без ясной истории релизов — это какой-то неподдерживаемый бардак.

Это происходит потому, что возможность поддержки — моя возможность понимать вас, читая ваш код.

Когда я смотрю на исходный код, на его коммиты и историю релизов, я должен иметь возможность понять, какие намерения были у его автора, что проект делал год назад, куда он движется теперь, каков его путь и проч. Вся эта информация должна присутствовать в исходном коде и, что наиболее важно, в истории Git.

[Теги Git](https://git-scm.com/book/en/v2/Git-Basics-Tagging) и [заметки о релизах](https://git-scm.com/book/en/v2/Git-Basics-Tagging) в GutHub - это два мощных инструмента, которые обеспечивают меня этой информацией. Используйте их по максимуму. Также не забывайте, что каждая бинарная версия продукта должна быть доступна для мгновенной загрузки. У меня должна быть возможность скачать версию 0.1.3 и проверить ее прямо сейчас, даже если текущая версия проекта 3.4.


## Недокументированные интерфейсы взаимодействия
<img src="/images/software-project-sins-7.gif" class="sticker">

Каждый компонент проекта имеет свои интерфейсы, через которые его принято использовать. Если это гем Ruby, то существуют классы и методы, которые я собираюсь использовать в качестве конечного пользователя этого гема. Если это веб-приложение, то существуют веб-страницы, которыми  конечный пользователь будет управлять, чтобы использовать приложение. У каждого проекта есть интерфейсы, и они должны быть ясно описаны. 

Как и всё, что сказано выше, это касается возможности поддержки. Я, новый программист в проекте, буду врубаться в проект через его интерфейсы взаимодействия. Я должен понимать, что он делает, и использовать его сам.

Сейчас я говорю о документации для пользователей, а не для разработчиков. В общем, я против документации внутри ПО. В данном случае я полностью согласен с ["Манифестом Agile"](http://agilemanifesto.org/), — рабочее ПО важней, чем исчерпывающая документация. Но это не относится к "внешней" документации, которую следует читать пользователям, а не разработчикам.

Поэтому процесс взаимодействия конечного пользователя с ПО должен быть понятно задокументирован.

Если ваше ПО это библиотека, тогда ее конечные пользователи - это разработчики ПО, которые собираются ею пользоваться; не вносить в нее свой вклад, а просто использовать ее как "черный ящик".

---------

Это критерии, используемые для оценки open-source проектов, принимающих участие в нашем соревновании за [награду](http://www.yegor256.com/2015/04/16/award.html).