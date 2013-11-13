---
layout: post
title: "Catchable fatal error после обновления до MODx 2.2.2"
date: 2011-10-03 13:57
comments: true
categories: PHP
tags:
  - modx
  - факап
---
После обновления до 2.2.2 в админской зоне стала появляться такая ошибка:

```
Catchable fatal error: Argument 1 passed to xPDOObject::load() must be an instance of xPDO, instance of modX given in core/xpdo/om/xpdoobject.class.php on line 404
```

Очень долго думал как решить. Оказалось суть в том, что при выходе из скриптанеправильно обрабатываются хендлеры повешенные на сессию. Что бы это исправить нужно добавить перед каждым выовом exit() в файлах /manager/min/index.php и /manager/min/lib/Minify.php вот такую строку:

```php
@session_write_close();
```

Возможно есть ещё места где может понадобиться такой "хак".
