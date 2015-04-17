---
layout: post
title: "Минус-мерж"
date: 2015-02-17 19:49:58 +0300
comments: true
categories: Git
tags:
  - cli
  - cd
  - git checkout
  - git merge
---

Большинство скорее всего знакомы с "магическим" аргументом `-` для комнады `cd` в консоли. Это переход в предыдущую выбранную дирректорию.

Работает так:

``` bash
8xx8@mac ~/git-minus[master] $ cd /
8xx8@mac / $ cd -
8xx8@mac ~/git-minus[master] $
```

Некоторые знают о том, что git поддерживает похожее поведение при смене ветки:

```bash
8xx8@mac ~/git-minus[master] $ git co feature
Switched to branch 'feature'
8xx8@mac ~/git-minus[feature] $ git co -
Switched to branch 'master'
8xx8@mac ~/git-minus[master] $ git co -
Switched to branch 'feature'
8xx8@mac ~/git-minus[feature] $
```

Здесь я использовал алиас `co` для `checkout`. Вообще про алиасы у меня была отдельная статья.

В общем все понятно и логично. Но мало кто знает что аргумент `-` можно передавать как аргумент к merge.  Давайте посмотрим что из этого выходит.

<!-- more -->

На этот варанант я натолкнулся совсем случайно. Мы используем git-flow и названия веток у нас, как результат, длинные. Сливать свои изменения в staging приходится достаточно часто, вот и я подумал, а как так сделать, что бы поменьше делать.

```bash
8xx8@mac git-minus[feature/coll_stuff] $ git commit -m "second"
[feature 8f65d91] second
 1 file changed, 0 insertions(+), 0 deletions(-)
  create mode 100644 2
8xx8@mac ~/git-minus[feature/coll_stuff] $ git co staging
Switched to branch 'staging'
8xx8@mac ~/git-minus[staging] $ g merge -
Updating a2c0a4d..8f65d91
Fast-forward
 2 | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 2
8xx8@mac ~/git-minus[staging] $
```

PROFIT!

