---
layout: post
title: Логгирование в GrayLog при помощи Monolog
date: '2016-07-28 14:45'
---

Работа с лог файлами отнимает время. Мне надоело грепать текстовые файлы,
пора оптимизировать эту область и попробовать GrayLog. [GrayLog][graylog] - open source решение
для хранения поиска и анализа логов на базе Elasticsearch.

### Коротко о GrayLog

GrayLog позволяет искать строки в логах, во всех или отдельных полях. Логи передаются в GrayLog структурированными --
разбитыми на поля. Но если запись в логах представляют собой единую строку, в GrayLog есть возможность настроить разбивку на поля
при помощи регулярных выражений.

Самое полезное в GrayLog это быстрый поиск и фильтрация логов. Например, можно получить только записи для user_id=23 или
только записи от nginx.

При поиске аномалий пригодится возможность GrayLog строить графики по полю в логах. По ним можно понять, как часто повторяется
запись в логах или какую долю составляет отностиельно всех записей.

### Установка GrayLog на сервере Amazon

Я воспользовался [готовым образом в Amazon EC2][graylog-ec2-image]. Установка и настройка заняла 20 минут.
Сделал все по [документации GrayLog][graylog-doc].

### Логгирование при помощи Monolog

В своем PHP приложении в composer.json добавил библиотеку `"graylog2/gelf-php": "~1.2"`
В Monolog добавил обработчик:

```php
<?php
$logger->pushHandler(
    new \Monolog\Handler\GelfHandler(
        new \Gelf\Publisher(
            new \Gelf\Transport\UdpTransport($gray_log['host'], $gray_log['port'])
        )
    )
);
```

Логгировать можно просто строки:

```php
<?php
$log->info('Импорт CSV завершен');
```

Но лучше добавлять конткекст - словарь с дополнительной информацией, например, id пользователя, имя файла и т.п
Это будет сохранено в отдельных полях GrayLog и поможет в анализе логов:

```php
<?php
$log->info('Импорт CSV завершен', ['user_id'=>$userId, 'csv_file'=>$file]);
```

[graylog]: https://www.graylog.org
[graylog-doc]: http://docs.graylog.org/en/2.0/pages/installation/aws.html
[graylog-ec2-image]: https://github.com/Graylog2/graylog2-images/tree/2.0/aws
