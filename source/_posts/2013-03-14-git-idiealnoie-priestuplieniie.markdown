---
layout: post
title: "Git: Идеальное преступление"
date: 2013-03-14 12:09
comments: true
categories: Git
tags:
  - git merge
  - fast-forward
  - fuckup
---
В компании, в которой я  работаю в настоящее время, мы используем при
разработке довольно распространённую [модель ветвления](http://nvie.com/posts/a-successful-git-branching-model/), а в качестве
инструмента для стандартизации работы по этой схеме [git-flow](https://github.com/nvie/gitflow). Так же мы
дополнили эту схему ещё одной веткой -- `staging`. Это временная ветка, в
которую сливаются `feature branches`, и на основе неё делаются
промежуточные релизы на тестовый сервер. Естественно, всё, что
происходит в этой ветке, нужно для предварительного просмотра и
тестирования выполненных задач и никак не должно попадать ни в основную
ветку разработки, ни в master, то есть в итоговый релиз. В принципе,
схема работы достаточно удачная. Она упрощает взаимодействие и работу
большой команды с разным навыком работы с git, и нештатные ситуации
случаются достаточно редко, но зато, если случаются, то это запоминается
надолго.

<!-- more -->

Началось всё с того, что мы стали замечать, что в `master` или в `develop`
ветку начинают попадать коммиты из ветки `staging`. Самые распространённые
из них -- это мержи веток с задачами. Первый раз решили, что кто-то по
невнимательности примержил стейджинг в ветку с задачей и потом вылил,
разбираться точно как и кто не стали, просто всем ещё раз рассказали как
нужно вести разработку и посоветовали внимательнее следить за тем, что
они делают.

Затем история повторилась, но уже в другой команде с другими людьми.
Стали копать, выяснили, что и раньше такое было, просто никто не замечал
лишних коммитов в истории, потому что в принципе код работал, тесты
проходили и ладно.

Разгадка проста и понятна сразу, кто-то примержил staging в ветку со
своей задачей, а потом через некоторое время, когда она вышла в релиз,
все нежелательные коммиты и попали в `develop`, а  потом -- в `master`. Но
как? Если первый раз нам удалось найти злополучный коммит вида `Merge
branch staging into 'feature/fourth'`, то в последний раз во всей истории
всё было чисто. Каждый участник команды говорил, что он всё делал по
инструкции и по модели ветвления.

Те кто знаком с тем, как происходит мерж веток в git, наверное, уже
догадались почему так. А остальным я продемонстрирую на примере.

Мы имеем новый репозиторий с несколькими коммитами в `master` ветку и
одним в `develop`

{% img /images/content/gitfuckup/1.png [] %}

Далее создадим новую задачу(`first`), закоммитим в неё первый коммит и
опубликуем.

```
$ git flow feature start first
$ touch first
$ git add first
$ git commit -m "First feature commit"
$ git flow feature publish first
```

То же самое сделаем для `second`

```
$ git flow feature start second
$ touch second
$ git add second
$ git commit -m "Second feature commit"
$ git flow feature publish second
```

{% img /images/content/gitfuckup/2.png [] %}

Теперь менеджер попросил нас вылить вторую задачу на тестовый сервер:

```
$ git checkout -b staging
$ git merge feature feature/second
$ git merge feature/second
$ git push --set-upstream origin staging
```

Но, предположим, у нас произошла ошибка в приложении, связанная только с
тестовым сервером и другого программиста попросили её исправить:

```
$ touch staging-fix
$ git add staging-fix
$ git commit -m "Staging fixed"
$ git push
```

{% img /images/content/gitfuckup/3.png [] %}

Далее нас попросили вылить на тестовый сервер первую задачу. Для этого
мы перешли в `staging`, обновили его и влили в него первую задачу:

```
$ git checkout staging
$ git pull --rebase origin staging
From github.com:Andrew8xx8/try_git
 * branch            staging    -> FETCH_HEAD
Current branch staging is up to date.
$ git merge feature/first
$ git push
```

{% img /images/content/gitfuckup/4.png [] %}

Пока всё верно. От ветки `develop` сделаны две задачи. В ветку `staging` влиты обе ветки с задачами и есть ещё один коммит нужный только для неё.

Мы возвращаемся к разработке второй задачи, но нас кто-то отвлёк и случайно получается, что "на автомате из истории" мы вполняем такую последовательность комманд:

```
git checkout feature/first
git pull --rebase origin staging
```

И ничего вроде бы страшного не произошло, в дальнейшей рутине мы можем ничего не заметить. Так называемый мерж коммит не создался, потому что мерж прошёл в режиме `fast-forward`.

История, если бы в ней не было коммита `Staging fixed`,  тоже сильно не
изменилась.

```
$ git log --oneline
87bb177 Merge branch 'feature/first' into staging
dc4b6d6 Staging fixed
56351b4 Second feature commit
7b843ef First feature commit
f664597 Commit in develop
dfcf2d5 Removed all the cats
a0184dc Add all the octocat txt files
1aad731 Added cute octocat story
```

Но кто смотрит историю после перехода в ветку? Поэтому вы продолжаете
работу. Делаем ещё один коммит в первую задачу (хотя в реале их может
быть гораздо больше) и публикуем её.

```
$ touch first-finish
$ git add first-finish
$ git commit -m "First feature finished"
$ git push
```

Никаких подозрительных сообщений в консоли. Всё вроде бы нормально.
Поэтому мы продолжаем спокойно спать по ночам, а остальная комманда
работать над проектом, потому что по сути только наша ветка содержит
сейчас стейджинг.

{% img /images/content/gitfuckup/5.png [] %}

Хотя уже по графику видно, что произошёл некоторый факап, но когда веток
и людей много этого незаметно, да и кто смотрит графики.

Затем нас просят выпустить первую задачу и сделать релиз, что мы и
делаем:

```
$ git flow feature finish first
$ git push
$ git flow release start v1
$ git flow release finish v1
$ git push
```

{% img /images/content/gitfuckup/6.png [] %}

Ну вот и всё. Полный фарш. В релиз попало то, что было в `staging`. В истории всё чисто с точки зрения того, что нет мерж коммита:

```
$ git log --oneline
5db05f9 Merge branch 'release/v1'
ffb5ee5 Merge branch 'feature/first' into develop
d93f812 First feature finished
87bb177 Merge branch 'feature/first' into staging
dc4b6d6 Staging fixed
56351b4 Second feature commit
7b843ef First feature commit
f664597 Commit in develop
dfcf2d5 Removed all the cats
a0184dc Add all the octocat txt files
1aad731 Added cute octocat story
```

Если посмотреть reflog, то вот те строчки, которые были сгенерированы во
время выполнения ошибочной команды:

```
87bb177 HEAD@{6}: rebase finished: returning to refs/heads/feature/first
87bb177 HEAD@{7}: checkout: moving from feature/first to 87bb1774e8d6555a06707a03ef1f0f9636206317^0
7b843ef HEAD@{8}: checkout: moving from feature/second to feature/first
```

Тоже ничего необычного.

Причина такого поведения -- это fast-forward мерж. При таком слиянии в
случае, если это возможно, указатель ветки просто перемещается на то же
место, куда указывает ветка, которая вливается:

```
git checkout develop
git merge feature
```

<img src="http://8xx8.ru/git-kmb/images/mr1.png" alt="" />

Как исправить получившуюся ситуацию? Никак. Можно попробовать найти
ветку, в которую влит `staging` и попробовать откатить её слияние в
`develop`,
а также удалить коммиты с релизом и почистить все ветки, но это долго и
сложно, особенно если веток с задачами много.

Что делать чтобы такое не произошло?

Что делать, чтобы такое не произошло?

1. Внимательно следить за исполняемыми командами.
2. Если используется git-flow, то использовать его команды для
   обновления и публикации веток.
3. Использовать либо полный синтаксис команды `git push/pull`,  либо
   полностью сокращённый синтаксис без указания веток. Потому что в
ситуации из приведённого примера не указывается, в какую ветку слить
изменения, поэтому так и получилось, что информация из `staging` попала в
ветку с задачей.



