---
layout: post
title: "Каталог lib в Rails"
date: 2012-12-10 10:58
comments: true
categories:
  - Ruby
tags:
  - ruby on rails
keywords: "lib, ruby on rails, tconsole, rake"
description: "Как подключить каталог lib в Ruby On Rails"
---

Однозначного и общеизвестного ответа на вопрос "Что положить в либ?"
нет, обычно туда валится всё, что нельзя положить в другие места. Вопрос
что с этим делать дальше.

<!-- more -->

Первое, что нужно сделать, подключить эту папку в автозагрузку в
`config/application.rb`:

```ruby
config.autoload_paths += %W(#{config.root}/lib)
config.autoload_paths += Dir["#{config.root}/lib/**/"]
```

Затем тестирование.

Плохая идея подключать эти тесты прямо в `test_helper.rb`, типа такого:

```ruby
Dir[Rails.root.join("test/lib/*.rb")].each {|f| require f}
```

Подключённые таким образом тесты будут выполнятся при любом раскладе,
даже при запуске тестов определённого типа например `rake test:functionals`.

Правильным решением этой задачи является подключение дополнительных
тестов через Rake Task.

Для этого нужно в создать или добавить в файл `lib/tasks/test.rake`
новую задачу описывающую запуск дополнительных тестов:

```ruby
namespace :test do
  Rails::SubTestTask.new(:lib => "test:prepare") do |t|
    t.libs << "test"
    t.pattern = 'test/lib/**/*_test.rb'
  end
end
```

Теперь тесты доступны для запуска через `rake test:lib`.

Далее если нужно добавить запуск этих тестов по умолчанию при запуске
всех то дописываем ниже:

```ruby
lib_task = Rake::Task["test:lib"]
test_task = Rake::Task[:test]
test_task.enhance { lib_task.invoke }
```

Чтобы научить tconsole понимать наши тесты нужно добавить либо в проект,
либо в домашнюю директорию файл `.tconsole`, в котором добавить lib как
тип тестов. Моя конфигурация для Rails приложения:

{% gist 4249514 %}
