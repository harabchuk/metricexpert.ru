---
layout: post
title: События и электронная коммерция из PHP
date: '2016-06-08 09:46'
---

Написал низкоуровневую библиотеку [php-ga-tools][php-ga-tools] для работы с Measurement Protocol на PHP.
Уже есть [несколько][mp-php-1] [хороших][mp-php-2] библиотек, но я хотел сделать небольшую, в которой можно работать
с параметрами хитов напрямую.

В библиотеке есть метод отправки события и пример отправки транзакции расширенной коммерции.
Остальные виды хитов нужно формировать самому. Часто, в приложении требуется один-два
вида хитов, поэтому такой подход оправдан.

Напомню, хит -- это массив с параметрами, который отправляется в Google Analytics.
Событие, просмотр страницы, транзакция -- это все хиты.

### О Client ID перед отправкой хита

В Measurement Protocol помимо Tracking ID нужно подавать Client ID. Это уникальный
идентификатор посетителя. Он сохраняется в браузере при посещении сайта, и потом GA по нему
определяет повторные посещения. Но мы хотим отправить хит с сервера поэтому Client ID нужно раздобыть.

Получaть ли реальный Client ID или нет зависит от задачи:

1. Если требуется связать хит с реальным посетителем, значит нужно достать его Client ID.
2. Если нужно просто посчитать что-то, то в качестве Client ID можно использовать любую уникальную строку.

Более интересный вариант с реальным посетителем, потому что отправив хит от его имени
мы увидим в Google Analytics источник этого посетителя, сколько раз заходил на сайт и все,
что GA знает об этом посетителе.

### Отправляем событие

```php
<?php
include "gasender.php";

$clientId = '...'; // предварительно сохраненный Client ID
$trackingId = 'UA-XXXXX-1'; // From your config

$ga = new \PhpGaTools\GaSender($trackingId, $clientId);
$payload = $ga->event('Form', 'Submit', 'Footer');
$ga->send($payload);
```

Метод `event` возвращает массив с параметрами хита,
а метод `send` отправляет этот массив в GA.

### Русский язык в хитах

Можно использовать русский язык при отправке хита. Если кодировка
ваших файлов отличается от UTF-8 нужно добавить в конструктор параметр с кодировкой.
Например, если у вас файлы в Window-1251 то добавьте параметр `CP1251`

```php
<?php
...
$ga = new \PhpGaTools\GaSender($trackingId, $clientId, 'CP1251');
$payload = $ga->event('Форма', 'Отправка', 'Футер');
...
```
Cтроку с вашей кодировкой подсмотрите в [списке кодировок][iconv-encodings].

### Как подготовить свой хит на примере PageView

Если нужно создать новый тип хита я подсматриваю [параметры в документации][ga-pageview]
и делаю метод, возвращающий массив параметров.

```php
<?php
class MyHits
{
  public static function pageview($host, $page, $title, $referrer=null, $ip=null){
    return array(
      'dh'=>$host,
      'dp'=>$page,
      'dt'=>$title,
      'dr'=>$referrer,
      'uip'=>$ip
    );
  }
}

$ga = new \PhpGaTools\GaSender($trackingId, $clientId);
$payload = MyHits::pageview('yoursite.ru', '/blog/how-to-create-rocket.html', 'Как сделать ракету');
$ga->send($payload);
```

Для примеров всегда сложно придумать названия классов, поэтому у меня получился MyHits.
Уверен, вы придумаете, что-то поумнее.

### Добавляем utm метки в pageview

Добавим поддержку utm меток. Но чтобы в методе `pageview` не было слишком много параметров
сделаем в виде отдельного метода `utm`.

```php
<?php
class MyHits
{
  public static function utm($campaign, $source, $medium, $keyword=null, $content=null){
    return array(
        'cn'=>$campaign,
        'cs'=>$source,
        'cm'=>$medium,
        'ck'=>$keyword,
        'cc'=>$content
    );
  }

  public static function pageview($host, $page, $title, $referrer=null, $ip=null){
     ...
  }
}

$ga = new \PhpGaTools\GaSender($trackingId, $clientId);
$payload = array_merge(
  MyHits::pageview('yoursite.ru', '/blog/how-to-create-rocket.html', 'Как сделать ракету'),
  MyHits::utm('Хобби', 'direct.yandex.ru', 'cpc', 'как сделать ракету')
);
$ga->send($payload);
```

### Отправка транзакции расширенной электронной коммерции

Напомню, что расширенную электронную коммерцию нужно специально включать в настройках
Google Analyitcs. Если вы используете обычный режим электронной коммерции, нужно использовать
[тип хита transaction][ga-transaction].

В расширенной электронной коммерции у транзакции нет своего типа хита, она отправляется
как дополнительные параметры к событию.

В библиотеке есть пример класса отправки транзакции, покажу как отправлять заказ.

```php
<?php
include "gasender.php";
include "ecommerce.php";

$clientId = '...'; // Google Analytics Client ID
$trackingId = 'UA-XXXXX-1'; // From your config

$ga = new \PhpGaTools\GaSender($trackingId, $clientId);
$ec = new \PhpGaTools\EnhancedEcommerce();

$products = array(
	array('id'=>'00411', 'name'=>'The Jungle Books', 'brand'=>'Rudyard Kipling', 'price'=>330, 'qty'=>1, 'category'=>'Classics'),
	array('id'=>'00412', 'name'=>'Just So Stories', 'brand'=>'Rudyard Kipling', 'price'=>350, 'qty'=>2, 'category'=>'Classics'),
);
$orderId = '0032';
$orderRevenue = 1030;

// склеиваем два массива: event и purchase
$payload = array_merge(
	$ga->event('Ecommerce', 'Event'),
	$ec->purchase($orderId, $orderRevenue, $products)
);

$ga->send($payload);
```

### Похожие библиотеки

[Эта библиотека][similar-lib], также работает с параметрами хитов напрямую.


[php-ga-tools]: https://github.com/harabchuk/php-ga-tools
[ga-pageview]: https://developers.google.com/analytics/devguides/collection/protocol/v1/devguide#page
[ga-transaction]: https://developers.google.com/analytics/devguides/collection/protocol/v1/devguide#ecom
[mp-php-1]: https://github.com/theiconic/php-ga-measurement-protocol
[mp-php-2]: https://github.com/ins0/google-measurement-php-client
[similar-lib]: https://github.com/krizon/php-ga-measurement-protocol
[iconv-encodings]: https://gist.github.com/hakre/4188459
