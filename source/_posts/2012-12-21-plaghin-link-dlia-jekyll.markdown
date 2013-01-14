---
layout: post
title: "Плагин link для Jekyll"
date: 2012-12-21 01:19
comments: true
categories: 
  - Ruby
  - Jekyll
tags:
  - ruby
  - jekyll
  - links
  - plugin
---

Нужно было для одного статического сайта сделать так, чтобы все ссылки
которые ведут на текущую открытую страницу помечались классом `current`.
Для того что бы пользователь по структуре меню мог определить, где он
находится.

Я написал плагин для этого. [https://github.com/Andrew8xx8/jekyll-link](https://github.com/Andrew8xx8/jekyll-link)
 
<!-- more -->

Плагин добавляет новый тег `link` с помощью которого генерируется HTML
код для ссылки, причём если адрес страницы в ссылке является частью или
полностью совпадает с адресом той страницы на которой она расположена,
то в неё добавится класс `current`.

Например, имеем такой шаблон с меню:

```html
 <ul>
   <li>{% link /blog Blog %}
   <li>
     {% link /about About %}
     <ul>
       <li>{% link /about/author Author %}
     </ul>
   </li>
 </ul>
```

На странице с автором (`/about/author`) сгенерируется такой код:

```html
 <ul>
   <li><a href="/blog">Blog</a></li>
   <li>
     <a href="/about" class="current">About</a>
     <ul>
       <li><a href="/about/author" class="current">Author</a>
     </ul>
   </li>
 </ul>
```

Смешно, нодокументация с примерами к плагину занимет больше строк чем сам код.
