---
layout: post
title: "Как поттратить свои изменения в git"
date: 2016-10-06 11:26:29 +0400
comments: true
categories: Git
tags:
  - git commit
  - amend
---

**TL;DR** При изменение коммита не откатывайте изменения в файле полностью, если этот коммит уже мержился в дургую ветку.

Иначе, если вы полностью отмените сделанные в файле изменения, то он пропадет из дифа этого коммита, и при повторном мерже с ним ничего не произойдет.  Все будет так, как и было в целевой  ветке до мержа. `git merge` будет говорить Already up-to-date, но контент файлов будет разный в этих ветках.

Простой пример из практики. Вы написали `console.log` в своей ветке, замержили в стейджинг, задеплоили, потестили, вспомнили, что был лишний код, пошли откатили этот `console.log`, замержили в стейджинг снова, все замержилось без конфликтов, но `console.log` в стейджинге остался, а у вас в ветке нет.

Притом это работает в любую сторону на уровне файла. Удалили строку в файле –> замерижили куда-то –> вернули туже строку в файл поменяв коммит –> замержили снова. Там где меняли строка есть, там куда мержили нет. Получается, классическая ситуация: "А у меня на ноутбуке все работает, сами ищите что не так!".

## Погнали разбираться ну как так-то

Создадим репозиторий и в нем два файла:

<!-- more -->

```bash
/tmp/git_fuckup $ git init
Initialized empty Git repository in /tmp/git_fuckup/.git/
/tmp/git_fuckup $ touch correct.txt broken.txt
/tmp/git_fuckup $ git add .
/tmp/git_fuckup $ git commit -m "Init commit"
[master (root-commit) d2b04c9] Init commit
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 broken.txt
 create mode 100644 correct.txt
```

Создадим новую ветку, напишем что-нибудь в первый и второй файлы и замержим в мастер:

```bash
/tmp/git_fuckup $ git commit -m "Init commit"
[master (root-commit) d2b04c9] Init commit
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 broken.txt
 create mode 100644 correct.txt
/tmp/git_fuckup[master] $ git checkout -b feature
Switched to a new branch 'feature'
/tmp/git_fuckup[feature] $ echo 'I need to be here!' >> correct.txt
/tmp/git_fuckup[feature] $ echo 'I need to be here!' >> broken.txt
/tmp/git_fuckup[feature] $ git add .
/tmp/git_fuckup[feature] $ git commit -m "Texts added"
[feature 551d298] Texts added
 2 files changed, 2 insertions(+)
/tmp/git_fuckup[feature] $ git checkout master
Switched to branch 'master'
/tmp/git_fuckup[master] $ git merge feature
Updating d2b04c9..551d298
Fast-forward
 broken.txt  | 1 +
 correct.txt | 1 +
 2 files changed, 2 insertions(+)
```

Прверяем что все ок:

```bash
/tmp/git_fuckup[master] $ cat correct.txt
I need to be here!
/tmp/git_fuckup[master] $ cat broken.txt
I need to be here!
/tmp/git_fuckup[master] $ git log --oneline
551d298 Texts added
d2b04c9 Init commit
```

Переходим снова в фичу и возвращяем файл таким каким он был до коммита, при этом не создаем новый коммит, а меняем его. Для упрощения я откатил его через git, но разницы нет никакой, можно удалить строку в файле вручную.

```bash
/tmp/git_fuckup[master] $ git checkout feature
Switched to branch 'feature'
/tmp/git_fuckup[feature] $ git checkout HEAD~1 broken.txt
/tmp/git_fuckup[feature] $ git commit --amend
[feature a12625d] Texts added
 Date: Wed Dec 13 23:09:53 2017 +0700
 1 file changed, 1 insertion(+)
/tmp/git_fuckup[feature] $ cat broken.txt
/tmp/git_fuckup[feature] $
```

Файл пуст, все как планировали. Внимательный читатель уже должен заметить, что вместо `2 files changed, 2 insertions(+)` имеем `1 file changed, 1 insertion(+)`. Из дифа полностью пропал один файл, что логично, мы же полностью удалили то, что добавляли и теперь этих изменений не существует в новом коммите. Важное замечание, что при изменении коммита, всегда создается новый комит с новым хешем, это знание пригодтися позже.

Мержим теперь в мастер:

```bash
/tmp/git_fuckup[feature] $ cat broken.txt
/tmp/git_fuckup[feature] $ git checkout master
Switched to branch 'master'
/tmp/git_fuckup[master] $ git merge feature
Merge made by the 'recursive' strategy.
```

Все смержилось, конфликтов нет. Казалось бы что все здорово, но:

```bash
/tmp/git_fuckup[master] $ cat correct.txt
I need to be here!
/tmp/git_fuckup[master] $ cat broken.txt
I need to be here!
/tmp/git_fuckup[master] $ git co feature
/tmp/git_fuckup[feature] $ cat correct.txt
I need to be here!
/tmp/git_fuckup[feature] $ cat broken.txt
/tmp/git_fuckup[feature] $
```

В мастере строка есть, в фиче, нет. Чтобы понять, что просиходит, надо вспомнить, как происходит мерж веток.
При мерже ветки A в B, находится ближайший родительский коммит Y между ними и затем дифы всех коммитов от Y до А поочередно применяются в ветку B.

Посмотрим лог мастер ветки:

```bash
/tmp/git_fuckup[master] $ g log --oneline --graph
*   81825c7 Merge branch 'feature'
|\
| * a12625d Texts added
* | 551d298 Texts added
|/
* d2b04c9 Init commit
```

Посмотрим дифы коммитов, оригинального и измененного.

```bash
/tmp/git_fuckup[feature] $ g show a12625d
commit a12625df8415a45f84a018532e295f6c8421238f
Author: Andreww8xx8 <avk@8xx8.ru>
Date:   Wed Dec 13 23:09:53 2017 +0700

    Texts added

diff --git a/correct.txt b/correct.txt
index e69de29..c20dfcd 100644
--- a/correct.txt
+++ b/correct.txt
@@ -0,0 +1 @@
+I need to be here!
```
```bash
/tmp/git_fuckup[feature] $ g show 551d298
commit 551d2989dc60290122d7853c3373b0e0e09757ce
Author: Andreww8xx8 <avk@8xx8.ru>
Date:   Wed Dec 13 23:09:53 2017 +0700

    Texts added

diff --git a/broken.txt b/broken.txt
index e69de29..c20dfcd 100644
--- a/broken.txt
+++ b/broken.txt
@@ -0,0 +1 @@
+I need to be here!
diff --git a/correct.txt b/correct.txt
index e69de29..c20dfcd 100644
--- a/correct.txt
+++ b/correct.txt
@@ -0,0 +1 @@
+I need to be here!
/tmp/git_fuckup[feature] ±
```

При втором мерже `git` пытается накатить диф коммита `a12625d`. В нем только одно изменение в файле `correct.txt` и оно не конфликтует с тем, что есть в фаловой системе после коммита `551d298`, а про файл `broken.txt` в дифе `a12625d` ничего нет, по этому он тоже останется таким как был в `551d298`.

Все комнады чтобы упростить копипаст, если вы вдруг заходтите воспроизвести ситуацию сами есть [здесь](https://gist.github.com/Andrew8xx8/58319104b9b17e136f0ce7bbdabad1c6).
