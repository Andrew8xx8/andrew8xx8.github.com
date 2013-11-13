---
layout: post
title: "Будь ленив и быстр. Zsh"
date: 2013-10-23 12:32
comments: true
categories: Git
tags:
  - git
  - cli
  - aliases
---

Влоо

<!-- more -->

Базовые сокращения

```
cp = cherry-pick
sta = status
cl = clone
ci = commit
co = checkout
br = branch
sl = stash list
sa = stash apply
ss = stash save
```

Прагматичный статус

```
st = status -s
```

{% img /images/content/fast_and_lazy/1.png [git st] %}

Diff изменений приготовленныз к комиту (staged)

```
dc = diff --cached
```

Diff изменений в последнем коммите

```
dlc = diff --cached HEAD^
```


Diff изменений в определенном коммите коммите (`git dr dcee010`)
```
dr  = "!f() { git diff "$1"^.."$1"; }; f"
```

Применение изменений из векти так, чтобы все наши коммиты шли после

```
up = pull --rebase
```

Отправление только текущей ветки

```
put = push origin HEAD
```

Сокращения для stash

```

```

Отмена последнего коммита с сохранением состояния stage

```
undo = reset --soft HEAD^
```

Добавить все файлы в stage к последнему коммиту не котрывая редактор
с сообщением

```
aps = commit --amend -C HEAD
```

Определить нахождение коммита

```
where = branch -av --contains
```

Красивый лог

```
lg = log --pretty=format:\"%h %Cblue%ar %Cgreen%an%Creset: %s %Cred%d\"
ll = log -10 --pretty=format:\"%h %Cblue%ar %Cgreen%an%Creset: %s %Cred%d\"
gr = log --graph --pretty=format:\"%Cred%h %Cblue%ar%Creset %Cgreen%an%Creset %s%Cred%d\"o
```

## Шаг второй. Zsh алиасы
