---
layout: post
title: "Открываем терминал правильно"
date: 2012-12-03 14:01
comments: true
categories: Linux
---
Я устал от того, что начав работу с проектом и открыв терминал, я вынужден каждый раз открывать вкладки и запускать в них однотипные команды. Отсутствие же адекватных названий вкладок, в случае, если открыто более одного проекта,  превращает навигацию по открытым терминалам в ад. Terminal, Terminale, Termina and so on...

В связи с этим я решил привнести немного автоматизации в этот процесс.

Не секрет, что при запуске терминала можно указать опции, которые повлияют на его последующую работу. Наиболее интересные из них:

```
  --tab Open a new tab in the last-opened window with the default profile
  --hide-menubar Turn off the menubar
  --maximize Maximise the window
  -e, --command Execute the argument to this option inside the terminal
  -t, --title=TITLE Set the terminal title
```

Исходя из этого команда для открытия терминала без строки меню с двумя вкладками и открытыми в них vim и zsh выглядит так:

```
  gnome-terminal --maximize --hide-menubar --tab --tab
```

Добавим названия заголовков:

```
  gnome-terminal --maximize --hide-menubar --tab -t "VIM | myproj" --tab -t "CONSOLE | myproj"
```

Добавим исполнение команд:

```
  gnome-terminal --maximize --hide-menubar --tab -e "zsh -c 'cd ~/myproj && vim'" -t "VIM | myproj" --tab -e "zsh -c 'cd ~/myproj; exec zsh" -t "CONSOLE | myproj"
```

Важный аспект состоит в том, что вкладка закрывается сразу же после выполнения команды, поэтому если нужно загрузить shell, следует исползовать exec.

Писать такую длинную команду каждый раз неудобно, поэтому лучше написать скрипт для загрузки.
Вот мой https://gist.github.com/418330833

{% img images/content/open-terminal.png [Так выглядит мой вариант] %}

P.S.
* Более подробно о доступных опциях терминала можно узнать выполнив: `gnome-terminal --help-all`
* Для Linux Mint все опции в силе. Просто используйте mate-terminal вместо gnome-terminal
* Для bash всё тоже самое, только zsh следует изменить на bash, либо на другой ваш любимый Shell.

