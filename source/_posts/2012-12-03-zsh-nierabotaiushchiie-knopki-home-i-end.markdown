---
layout: post
title: "ZSH: неработающие кнопки Home и End"
date: 2011-12-03 13:54
comments: true
categories:
  - Жизнь
tags:
  - cli
  - zsh
  - факап
keywords: "putty, не работают клавиши, home, end, zsh"
---
При доступе к серверу из Putty или терминала Gnome иногда не работают кнопки Home и End.

Решение простое. Нужно в файле ~/.zshrc добавить эти строки:

```
bindkey "^[[1~" beginning-of-line
bindkey "^[[4~" end-of-line

bindkey "^[OH" beginning-of-line
bindkey "^[OF" end-of-line
```

Первые две для Putty другие для Gnome (Mate) Trerminal.
