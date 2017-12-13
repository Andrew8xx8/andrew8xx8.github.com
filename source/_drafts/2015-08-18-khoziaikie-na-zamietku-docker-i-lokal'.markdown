---
layout: post
title: "Хозяйке на заметку: Docker, ruby и локаль"
date: 2015-08-18 19:02:54 +0000
comments: true
categories: ХНЗ
tags:
  - docker
  - ubuntu
  - ruby
  - nokogiri
  - wrong
  - json
  - invalid byte sequence
  - utf
  - us-ascii
---

С недавних пор стал использовать для разработки docker вместо vagrant.

Первые странности не заставили себя ждать и сегодня пару часов возился с локалями в ubuntu.

В официальном образе [ruby](https://hub.docker.com/_/ruby/) для docker локаль по умолчанию не работеет с UTF.

Это может вам создаь проблем в самых неожиданных местах.

## Симптомы

Целый веер проблем и падений ruby проекта в самых неожиданных местах:

— в nokogiri вместо ошибок просто молча не работают css селекторы. Возвращается пустота
— падают ассерты wrong с ошибкой `couldn't retrieve source code due to #<ArgumentError: invalid byte sequence in US-ASCII>`
— JSON фикстуры из файлов, до этого вполне работающие падают внутри JSON.parse с ошибкой `Encoding::InvalidByteSequenceError: "\xD0" on US-ASCII`


## Фикс

Пример для русского языка.

```
RUN apt-get update
RUN apt-get install locales
RUN echo 'ru_RU.UTF-8 UTF-8' >> /etc/locale.gen
RUN locale-gen ru_RU.UTF-8
RUN dpkg-reconfigure -fnoninteractive locales
ENV LC_ALL=ru_RU.utf8
ENV LANGUAGE=ru_RU.utf8
RUN update-locale LC_ALL="ru_RU.utf8" LANG="ru_RU.utf8" LANGUAGE="ru_RU"
```
