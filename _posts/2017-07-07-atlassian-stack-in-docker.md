---
layout: post
title: Быстрый запуск Atlassian Stack (JIRA+Bitbucket+Confluence) в контейнерах
tags: atlassian jira bitbucket confluence docker
comments: True
excerpt_separator: <!--more-->
---

![Atlassian Stack Pic](/images/atlassian_stack.png)

_Atlassian Stack (я решил его так назвать), объединяющий JIRA, Confluence и Bitbucket - одна из полезнейших связок в сфере сервисов для CI/CD и DevOps. В этой заметке я постараюсь описать, как можно быстро и удобно развернуть этот стек с помощью docker на своем сервере. Вы можете воспользоваться моим [репозиторием](https://github.com/approximatenumber/atlassian-stack_docker) для быстрого старта:_

_Здесь не реализованы все полезные возможности, например, Crowd, а также почтовый сервер работает только для SMTP. Добавляйте фичи сами. Заметку написал после того, как на работе пришлось разворачивать этот стек на небольшом сервере для определенных нужд компании._

<!--more-->

Сначала, естественно, установите `docker`. А также `docker-compose` (я использую именно его для быстрого старта и управления сервисами).

Будем разворачивать связку, как на картинке выше.
Обратите внимание, что я использовал конкретные версии образов, с которыми у меня не возникло ошибок. Будьте внимательны: определенные версии сервисов Atlassian могут не "подружиться" с версией postgres, например.

### Nginx+Letsencrypt

Nginx будет проксировать запросы к сервисам.
Я пользовался официальным образом: [https://hub.docker.com/_/nginx/] (там можно найти более подробное описание)
Создаем директорию для конфигурации:
```
mkdir -p nginx/conf.d
```
И конфигурационный файл `atlassian.conf`:
```
server {
  listen 80;
  server_name rdc-expat.xyz;

  location /.well-known/acme-challenge/ {
    root /var/www/letsencrypt;
  }

  location / {
    return 301 https://$host$request_uri;
  }
}
 
server {
  listen 443 ssl;
 
  ssl_certificate certs/live/domain.com/fullchain.pem;
  ssl_certificate_key certs/live/domain.com/privkey.pem;
 
  server_name domain.com;
 
  location /jira {
    proxy_pass http://domain.com:8080/jira;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    client_max_body_size 10M;
  }

  location /confluence {
    proxy_pass http://domain.com:8090/confluence;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    client_max_body_size 10M;
  }

  location /bitbucket {
    proxy_pass http://domain.com:7990;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    client_max_body_size 10M;
  }
}
```
**Обратите внимание, мы будем использовать letsencrypt-сертификаты для обеспечения работы HTTPS.** Я пользовался отличной статьей, где описывается получение сертификата с помощью контейнера с letsencrypt и конфигурация nginx для этого: <https://devsidestory.com/lets-encrypt-with-docker/>
Если кратко, то:
Создаем директории:
```
mkdir letsencrypt
```
Создаем там `Dockerfile`:
```
FROM debian:jessie-backports
 
RUN apt-get update \
  && apt-get install -y letsencrypt -t jessie-backports \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* \
  && mkdir -p /etc/letsencrypt/live/rdc-expat.xyz \
  && openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/letsencrypt/live/domain.com/privkey.pem \
    -out /etc/letsencrypt/live/domain.com/fullchain.pem \
    -subj /CN=domain.com
```

`docker-compose` для старта nginx+letsencrypt может выглядеть так:
```
version: '2'
services:

  letsencrypt:
    build: ./letsencrypt
    command: /bin/true
    volumes:
      - letsencrypt_certs:/etc/letsencrypt
      - letsencrypt_www:/var/www/letsencrypt

  nginx:
     container_name: nginx
     image: nginx:stable
     volumes:
       - letsencrypt_certs:/etc/nginx/certs
       - letsencrypt_www:/var/www/letsencrypt
       - ./nginx/conf.d:/etc/nginx/conf.d/
     ports:
       - 80:80
       - 443:443
     restart: always
     links:
      - jira
      - confluence
volumes:
  letsencrypt_certs: ~
  letsencrypt_www: ~
```
Собираем letsencrypt и запускаем вместе с nginx
```
docker-compose build letsencrypt
docker-compose up -d nginx letsencrypt
```
Получаем SSL-сертификат
```
docker-compose run --rm letsencrypt \
  letsencrypt certonly --webroot \
  --email me@domain.com --agree-tos \
  -w /var/www/letsencrypt -d domain.com
```
Теперь можно перезапустить nginx уже с полученным сертификатом!
```
docker-compose kill -s SIGHUP nginx
```
Не забыть продумать механизм обновления сертификатов. Я еще не продумал :)

### DB (Postgres)

Понадобится база данных, в которой будут храниться данные сервисов Atlassian. 
Я пользовался официальным образом: [https://hub.docker.com/_/postgres/] (там можно найти более подробное описание)
Создаем том для постоянных данных:
```
docker volume create postgres-data
```
Создаем `Dockerfile` в директории `db` с:
```
FROM postgres:9.2
ADD init.sql /docker-entrypoint-initdb.d/
```
Там же создаем скрипт `init.sql` для инициализации нужных нам баз и пользователей:
```
CREATE USER jirauser PASSWORD 'secure';
CREATE DATABASE jiradb;
GRANT ALL PRIVILEGES ON DATABASE jiradb TO jirauser;

CREATE USER confluenceuser PASSWORD 'secure';
CREATE DATABASE confluencedb;
GRANT ALL PRIVILEGES ON DATABASE confluencedb TO confluenceuser;

CREATE USER bitbucketuser PASSWORD 'secure';
CREATE DATABASE bitbucketdb;
GRANT ALL PRIVILEGES ON DATABASE bitbucketdb TO bitbucketuser;
```
часть `docker-compose.yml` для базы данных может выглядеть так:
```
  db:
     build: ./db
     container_name: db
     ports:
      - 5432:5432
 volumes:
  - postgres-data:/var/lib/postgresql
```
Создаем контейнер с базой:
```
docker-compose create db
```

### JIRA

Пользовался образом `cptactionhank/atlassian-jira-software`.
Создаем контейнеры для данных:
```
docker volume create jira-home
docker volume create jira-install
```
часть `docker-compose.yml` для jira может выглядеть так:
```
  jira:
    image: cptactionhank/atlassian-jira-software:7.4.0
    container_name: jira
    ports:
     - 8080:8080
    volumes:
     - jira-home:/var/atlassian/jira
     - jira-install:/opt/atlassian/jira
    depends_on:
     - db

volumes:
  jira-home:
    external: True
  jira-install:
    external: True
```
После старта необходимо будет настроить `server.xml` (находится в томе jira-install:conf/) для работы nginx+jira, подробности у Atlassian: [https://confluence.atlassian.com/jirakb/integrating-jira-with-nginx-426115340.html]
Если всё сделано правильно, то после старта JIRA будет доступна по `domain.com/jira`.

### Confluence

Пользовался образом `mminks/docker-oracle-jdk-confluence`, потому как он работает на основое Oracle Java, с которой Atlassian, как сам признается, работает лучше. Образы с OpenJDK не завелись с первого раза. Возможно, конечно, это моя вина.

 Создаем контейнеры для данных:
```
docker volume create confluence-home
docker volume create confluence-install
```
Секция `docker-compose.yml` для confluence может выглядеть так:
```
  confluence:
    image: mminks/docker-oracle-jdk-confluence:5.10
    container_name: confluence
    ports:
     - 8090:8090
    volumes:
     - confluence-home:/var/atlassian/confluence
     - confluence-install:/opt/atlassian/confluence
    depends_on:
     - db
volumes:
  confluence-home:
    external: True
  confluence-install:
    external: True
```
`server.xml` для работы через nginx также необходимо настроить, подробности у Atlassian: [https://confluence.atlassian.com/confeap/]running-confluence-behind-nginx-with-ssl-849150880.html
Если всё сделано правильно, то после старта Confluence будет доступен по `domain.com/confluence`.

### Bitbucket

Использовал образ `cptactionhank/atlassian-bitbucket`.
Создаем контейнеры для данных:
```
docker volume create bitbucket-home
docker volume create bitbucket-install
```
Секция `docker-compose.yml` для bitbucket может выглядеть так:
```
  bitbucket:
    image: cptactionhank/atlassian-bitbucket:4.0.2
    container_name: bitbucket
    ports:
     - 7990:7990
    volumes:
     - bitbucket-home:/var/atlassian/bitbucket
     - bitbucket-install:/opt/atlassian/bitbucket
    depends_on:
     - db

volumes:
  bitbucket-home:
    external: True
  bitbucket-install:
    external: True
```
`server.xml` для работы через nginx также необходимо настроить, я воспользовался информацией из этой статьи: [http://blog.sukitsuki.com/2016/05/27/Atlassian-Jira-Bitbucket-Nginx%C2%A0reverse-proxy/]
Если всё сделано правильно, то после старта Bitbucket будет доступен по `domain.com/bitbucket`.

### Mail server
Мне было необходимо, чтобы пользователям сервисов приходили уведомления на их почтовые ящики, поэтому я сделал почтовый сервер (только SMTP).
Воспользовался классным образом `tvial/docker-mailserver`, который очень удобен. На странице проекта в гитхабе описаны подробности настройки. Я же включил только SMTP.
Создаем директории:
```
mkdir -p mail/config
```
Создаем почтовые ящики для сервисов:
```
docker run --rm \
    -e MAIL_USER=<jira/confluence/bitbucket@domain.com \
    -e MAIL_PASS=secure \
    -ti tvial/docker-mailserver:latest \
    /bin/sh -c 'echo "$MAIL_USER|$(doveadm pw -s SHA512-CRYPT -u $MAIL_USER -p $MAIL_PASS)"' >> config/postfix-accounts.cf
```
Секция `docker-compose.yml` для smtp-сервера **с полученными ранее сертификатами letsencrypt** может выглядеть так:
```
  mail:
    image: tvial/docker-mailserver:latest
    hostname: mail
    domainname: domain.com
    container_name: mail
    ports:
    - "25:25"
    volumes:
    - maildata:/var/mail
    - mailstate:/var/mail-state
    - ./mail/config/:/tmp/docker-mailserver
    - letsencrypt_certs:/etc/letsenrypt
    environment:
    - SSL_TYPE=letsencrypt
    - SMTP_ONLY=1
    - ONE_DIR=1
    - PERMIT_DOCKER=network
```
Без последней опции `PERMIT_DOCKER` контейнеры с Atlassian не могли связаться с почтовым сервером.

## Итог

Полный `docker-compose.yml` доступен в моем [репозитории](https://github.com/approximatenumber/atlassian-stack_docker), а также всё, что нужно для того, чтобы выполнить вышеуказанные действия.
