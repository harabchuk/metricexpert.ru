---
layout: post
title: ClientID в Google Tag Manager
date: '2016-06-06 10:23'
---

Настраивая коллтрекинг для одного из клиентов потребовалось получить Client ID из
Google Analytics. При обычной установке Google Analytics это делается так:

```javascript
ga(function(tracker)){
  var clientId = tracker.get('clientId');
  // что-то делаем с clientId
  ...
}
```

### Правильный способ получения ClientID в GTM

В предыдущем примере clientId получается через функцию обратного вызова.
Это значит, что в GTM должен появится триггер, который срабатывает когда ClientID доступен.

#### План действий:

1. Создать тэг получающий clientId и создающий событие ClientIdReady
2. Создать триггер на основе события ClientIdReady
3. Создать тэг с вашим кодом срабатывюащий по триггеру

В результате, ваш код, в котором нужен ClientID запускается по триггеру и гарантированно
получает ClientID.

#### 1. Создаем тэг получающий clientId

Создайте тэг с javascript

```html
<script>
ga(function(){
  var tracker = ga.getAll()[0];
  dataLayer.push({
    'event' : 'ClientIdReady',
    'client_id' : tracker.get('clientId')
  });
});
</script>
```

Имя события ClientIdReady я сам придумал, можно использовать любое имя.

Назначаем запуск тэга только после тэга Google Analytics

#### 2. Создаем триггер на основе события ClientIdReady

#### 3. Создаем тэг с вашим кодом

Я создам код, который выводит в консоль значение clientId

```html
<script>
 console.log('{% raw %}{{ClientID}}{% endraw %}');
</script>
```

Назначаем запуск тэга при срабатывании триггера ClientIdReady
