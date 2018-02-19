---
title: Кастомизация reviewers в Pull-Request`ах на Bitbucket
date: 2017-04-13 00:00:00 Z
tags:
- bitbucket
- cicd
- devops
- scriptrunner
layout: post
comments: true
excerpt_separator: "<!--more-->"
---

_Нам понадобилась возможность делать merge пулл-реквеста для определенной группы пользователей без необходимости одобрения со стороны approvers. Т.е. главные разработчики могут позволить себе делать merge без code-review. Как оказалось, таких возможностей в Bitbucket нет. В случае задания в настройках репозитория опции "Requires N approvers" невозможно выполнить merge, какими бы полномочиями пользователь не обладал, Bitbucket требует одобрения! Конфигурация Branch Permissions также не может данную проблему. Выручить может ScriptRunner._

<!--more-->

Поэтому был добавлен небольшой ScriptRunner-скрипт _"Require a number of approvers"_ в секцию _"Script Merge Checks"_ (в глобальных настройках ScriptRunner). Перед этим была создана группа для главных разработчиков _bitbucket-sw-managers_, пользователи которой могут свободно делать merge без review/approve. Также необходимо выключить опцию _Pull requests → Requires N approvers_ в настройках репозиториев, если она включена (мы будем управлять этой опцией динамически).

Сам скрипт выглядит следующим образом:

```
import com.atlassian.sal.api.component.ComponentLocator
import com.atlassian.bitbucket.user.UserService
import com.atlassian.bitbucket.auth.AuthenticationContext
 
def userService = ComponentLocator.getComponent(UserService)
def authContext = ComponentLocator.getComponent(AuthenticationContext)
 
def permitted_groups = ["bitbucket-sw-managers"]
def need_approve = true
 
for (group in permitted_groups) {
    if (userService.isUserInGroup(authContext.getCurrentUser(), group)) {
        need_approve = false
    }
}
 
return need_approve
```

_`permitted_groups` является списком, потому что вполне вероятно, что одной группой не обойтись в ближайшем будущем._

Не забыть выставить требуемое число ревьюеров в поле _Minimum number of approvers_, например _1_.
