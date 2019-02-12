---
title: Updating Jenkins Plugins With Pipeline
date: 2017-08-02 00:00:00 Z
tags:
- jenkins
- automation
layout: post
comments: true
excerpt_separator: "<!--more-->"
---

![Question](/images/update_ask.png)

Do you ever notice that Jenkins plugins versions are being updatedso fast? It\`s either good or bad: good, because we`ve got new features and bugfixes so often; bad, because we often need to update them manually. :)

But we can automate plugins update procedure with Jenkins cool pipeline! I write an example `Jenkinsfile` using magic of `shell` and `groovy`:

<!--more-->

<script src="https://gist.github.com/approximatenumber/e3054000b6553552682a314fa8376c60.js"></script>

As a result, this job will be started `@monthly`, check available updates, send an email to you to confirm update. If you don\`t confirm it in 24 hours, job will update plugins automatically.

