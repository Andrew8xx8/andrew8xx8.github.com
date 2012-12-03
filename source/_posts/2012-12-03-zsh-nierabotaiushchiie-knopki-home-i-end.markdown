---
layout: post
title: "ZSH: неработающие кнопки Home и End"
date: 2011-12-03 13:54
comments: true
categories: Linux ZSH
---
При доступе к серверу из Putty или терминала Gnome иногда не работают кнопки Home и End.

Решение простое. Нужно в файле ~/.zshrc добавить эти строки:

{% gist 2768395 %}

Первые две для Putty другие для Gnome (Mate) Trerminal.
