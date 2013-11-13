---
layout: post
title: "Быстрый способ передать Template Variable (TV) в снипет в MODx Revolution"
date: 2012-01-12 13:59
comments: true
categories: PHP
tags:
  - modx
  - tv
---
Что бы быстро и просто передать Template Variable (TV) параметр в снипет в MODx Revolution достаточно передать его как параметр при вызове снипета. Например создадим параметр TV и назовём его MyColor затем передадим в снипет его таким образом при вызове:

```php
[[MySnippet? &color=`[*MyColor*]`]]
```

Когда система будет обрабатывать шаблон она подставит значение параметра в вызов и оно передасться в сам снипет. Код MySnippet:

```php
echo 'You have entered: '.$color;
```
