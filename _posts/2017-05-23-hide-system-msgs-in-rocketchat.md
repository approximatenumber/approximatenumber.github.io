---
layout: post
title: How to hide system messages in Rocket.Chat
tags: rocketchat css
comments: True
excerpt_separator: <!--more-->
---

Rocket.Chat is a great tool, but it doesn`t permit to hide annoying system messages in channels about:
* privacy type
* room name
* room description
* room topic
* new roles

To hide them all use some custom CSS styles below (section **Layout**->**Custom CSS** in **Administation** panel)

<!--more-->

```
// hide system message about topic
.room_changed_topic {
  display: none;
}

// hide system message about roles
.subscription-role-added {
  display: none;
}

// hide system message about description
.room_changed_description {
  display: none;
}

// hide system message about room name
.r {
  display: none;
}

// hide system message about privacy

.room_changed_privacy {
  display: none;
}
```

For example, here is two screenshots:

before adding CSS

![before](/images/rocket_before_css.png)

and after

![after](/images/rocket_after_css.png)
