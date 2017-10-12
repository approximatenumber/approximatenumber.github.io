---
layout: post
title: :rocket: Backup your volumes from docker-compose.yml
tags: docker devops
comments: True
excerpt_separator: <!--more-->
---

`docker-compose` is a great thing to deploy your services. It\`s a new tool for, so maybe I am wrong, but I cannot find any usable solution to backup volumes from my services... Of course, I can do it manually, but it sucks :)

<!--more-->

So, I made a small solution to make such things, [compose-backuper-script
](https://github.com/approximatenumber/compose-backuper-script). It is a python-script to backup **named** docker-compose volumes. It looks at `docker-compose.yml`, finds named volumes from services, runs [small container](https://hub.docker.com/r/approximatenumber/compose-backuper/) with volumes (`ro`👌) and make a backup archive of data.

Read [detailed manual](https://github.com/approximatenumber/compose-backuper-script/blob/master/README.md) how to use this script.

Simple example of usage:

```sh
$ python compose-backup.py -f ../opt/app/docker-compose.yml -p myproj -d /opt/backups/
INFO:compose-backup:jenkins-home: saving...
INFO:compose-backup:jenkins-home: saved
INFO:compose-backup:DONE! Volumes saved: 1

$ ls /opt/backups
myproj_jenkins-home.tar.gz
```

Thanks.

