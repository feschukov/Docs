# Несколько полей с рейтингом

## Введение

**Статья актуальна для easyComm версии 1.6.0-pl и выше.**

В компоненте easyComm имеется поле rating (Оценка), которое позволяет посетителям сайта проставить оценку (в виде звездочек) чего-либо. Типовой сценарий применения – отзывы о товарах или услугах.

Но бывают ситуации, когда необходимо добавить несколько таких полей.

Например, рассмотрим ситуацию, что у нас сайт автомобильной тематики, где посетители оставляют отзывы об автомобилях.
Каждый отзыв включает в себя общую оценку,  оценку внешнего вида, оценку надежности.

![Мультирейтинг easyComm](https://file.modx.pro/files/1/0/c/10ca9042f318b7990b41379fe2e760e5.png)

Рассмотрим, как заставить easyComm работать с этими 3-мя полями.

Итак, у нас 3 поля, для простоты в этом примере мы присвоим им просто порядковые номера:

- общая оценка – rating;
- внешний вид – rating2;
- надежность – rating3.

Поле rating в системе уже имеется, с ним мы не производим никаких действий, нам нужно добавить rating2 и rating3.

Нам потребуется:

- добавить соответствующие поля в базу данных;
- добавить поля к объектам, чтобы MODX про них знал;
- настроить вывод полей в панели управления сайтом;
- прописать записи в словарях системы;
- настроить форму добавления отзыва и отображение рейтинга в списке отзывов.

Данная статья пересекается с заметкой «Плагины и кастомизация» из документации и является частным случаем.

## Шаг 1. Добавляем поля в базу данных

Удобнее всего это сделать через phpMyAdmin.

1.1. Требуется добавить поля в 2 таблицы:

- **modx\_ec\_messages**. Добавляем 2 поля **rating2** и **rating3**, по аналогии с существующим полем **rating** (Type – tinyint(1), Null – false, Default – 0).

![messages](https://file.modx.pro/files/1/4/4/144e8dcb39d0bf273fffd1accd42d83b.png)

- **modx\_ec\_threads**. Здесь уже добавляем в 2 раза больше полей, т.к. для каждого рейтинга компонент будет считать 2 вида оценок, среднюю и по Вильсону. Соответственно у них постфикс **\_simple** и **\_wilson**. Имена полей **rating2\_simple**, **rating2\_wilson**, **rating3\_simple**, **rating3\_wilson** (Type – decimal, Length - 12,6, Null – false, Default – 0).

![threads](https://file.modx.pro/files/6/6/3/663b6a6fbcdb4979ae074bc633ac1e47.png)

## Шаг 2. Создаем файлы плагина для расширения объектов

2.1. Создать папку "multirating" (можете задать любое имя) в 2 каталогах:

- `/core/components/easycomm/plugins/multirating/`
- `/assets/components/easycomm/plugins/multirating/`

2.2. В каталоге `/core/components/easycomm/plugins/multirating/` создать файлы:

::: code-group

```php [index.php]
<?php

return array(
  'xpdo_meta_map' => array(
    'ecMessage' => require_once dirname(__FILE__) .'/ecmessage.map.inc.php',
    'ecThread' => require_once dirname(__FILE__) .'/ecthread.map.inc.php'
  ),
  'manager' => array(
    'ecMessage' => MODX_ASSETS_URL . 'components/easycomm/plugins/multirating/ecmessage.js',
    'ecThread' => MODX_ASSETS_URL . 'components/easycomm/plugins/multirating/ecthread.js'
  )
);
```

```php [ecmessage.map.inc.php]
<?php

return array(
  'fields' => array(
    'rating2' => NULL,
    'rating3' => NULL
  ),
  'fieldMeta' => array(
    'rating2' => array(
      'dbtype' => 'tinyint',
      'precision' => '1',
      'phptype' => 'integer',
      'null' => false,
      'default' => 0
    ),
    'rating3' => array(
      'dbtype' => 'tinyint',
      'precision' => '1',
      'phptype' => 'integer',
      'null' => false,
      'default' => 0
    ),
  ),
  'indexes' => array(

  )
);
```

```php [ecthread.map.inc.php]
<?php

return array(
  'fields' => array(
    'rating2_simple' => 0,
    'rating2_wilson' => 0,
    'rating3_simple' => 0,
    'rating3_wilson' => 0,
  ),
  'fieldMeta' => array(
    'rating2_simple' => array(
      'dbtype' => 'decimal',
      'precision' => '12,6',
      'phptype' => 'float',
      'null' => false,
      'default' => 0,
    ),
    'rating2_wilson' => array(
      'dbtype' => 'decimal',
      'precision' => '12,6',
      'phptype' => 'float',
      'null' => false,
      'default' => 0,
    ),
    'rating3_simple' => array(
      'dbtype' => 'decimal',
      'precision' => '12,6',
      'phptype' => 'float',
      'null' => false,
      'default' => 0,
    ),
    'rating3_wilson' => array(
      'dbtype' => 'decimal',
      'precision' => '12,6',
      'phptype' => 'float',
      'null' => false,
      'default' => 0,
    ),
  ),
  'indexes' => array(

  )
);
```

:::

Первый файл описывает, какие файлы содержит наш плагин, во втором мы расширяем объект ecMessages, в третьем расширяем объект ecThread.

2.3. В каталоге `/assets/components/easycomm/plugins/multirating/` создать файлы:

::: code-group

```js [ecmessage.js]
easyComm.plugin.multirating = {
  getFields: function (config) {
    return {
      rating2: { xtype: 'numberfield', allowBlank: false, allowNegative: false, allowDecimals: false, fieldLabel: _('ec_message_rating2'), anchor: '99%' },
      rating3: { xtype: 'numberfield', allowBlank: false, allowNegative: false, allowDecimals: false, fieldLabel: _('ec_message_rating3'), anchor: '99%' }
    }
  }
  ,getColumns: function () {
    return {
      rating2: { width:90, sortable:true, name: 'rating2', renderer: easyComm.utils.renderRating },
      rating3: { width:90, sortable:true, name: 'rating3', renderer: easyComm.utils.renderRating }
    }
  }
};
```

```js [ecthread.js]
easyComm.pluginThread.myplugin = {
  getFields: function (config) {
    return {
      rating2_simple: { xtype: 'displayfield', fieldLabel: _('ec_message_rating2'), anchor: '99%' },
      rating2_wilson: { xtype: 'displayfield', fieldLabel: _('ec_message_rating2'), anchor: '99%' },
      rating3_simple: { xtype: 'displayfield', fieldLabel: _('ec_message_rating3'), anchor: '99%' },
      rating3_wilson: { xtype: 'displayfield', fieldLabel: _('ec_message_rating3'), anchor: '99%' }
    }
  }
  ,getColumns: function () {
    return {
      rating2_simple: { width:90, sortable:true, name: 'rating2', renderer: easyComm.utils.renderRating },
      rating2_wilson: { width:90, sortable:true, name: 'rating2', renderer: easyComm.utils.renderRating },
      rating3_simple: { width:90, sortable:true, name: 'rating3', renderer: easyComm.utils.renderRating },
      rating3_wilson: { width:90, sortable:true, name: 'rating3', renderer: easyComm.utils.renderRating }
    }
  }
};
```

:::

Теперь MODX знает про новые поля, но мы их нигде не видим. Что ж, добавим их отображение на сайте.

## Шаг 3. Отображение полей в панели управления

easyComm содержит специальные системные настройки, которыми можно гибко управлять отображением полей списке и окне редактирования как сообщений, так и цепочек.

Это системные настройки `ec_message_grid_fields`, `ec_message_window_layout`, `ec_thread_grid_fields`, `ec_thread_window_fields`.

Необходимо в них добавить наши новые поля. Подробнее про это написано в разделе документации "Плагины и кастомизация".

## Шаг 4. Добавить записи в лексикон

Если вы все сделали правильно, то теперь можно открыть в админке раздел Приложения -> Сообщения (easyComm), где мы увидим в списке сообщений новые колонки. Однако заголовки таблицы будут пустые.

Исправляем. Переходим в "Управление словарями", выбираем Пространство имен easycomm, нужный нам язык и добавляем недостающие записи:

- `ec_message_rating2`
- `ec_message_rating3`
- `ec_thread_rating2_simple`
- `ec_thread_rating2_wilson`
- `ec_thread_rating3_simple`
- `ec_thread_rating3_wilson`

Позже нам нужно будет в этом же разделе добавить еще записи для фронтэнда, они начинаются с префикса "ec\_fe\_...".

## Шаг 5. Фронтэнд, форма добавления отзывов

Как известно, за добавление отзывов отвечает сниппет **ecForm**.

Для обеспечания работы с новыми полями потребуется явно задать параметры:

- **requiredFields**, куда дописать новые поля, например user_name,rating,rating2,rating3
- **allowedFields**, куда дописать новые поля, например user_name,user_email,rating,rating2,rating3,text

::: warning Важно
В этих параметрах должны быть явно перечислены все поля вашей формы, которые вам нужны! Посмотрите в документации на сниппет значения по-умолчанию для параметров!
:::

Кроме того, потребуется внести изменения в чанк формы, за которую отвечает параметр **tplForm**.
Нужно или создать копию стандартного чанка tpl.ecForm и указать его в параметре **tplForm**, либо менять родной чанк, но следить, чтобы при обновлении он не перезаписался. Я рекомендую первый вариант.

Чтобы добавить новые "звездочки" для полей rating2 и rating3, скопируйте html код, отвечающий за поле rating и замените в нем все упоминания rating на rating2 и rating3 соответственно, ИСКЛЮЧАЯ названия css классов и аттрибута data-description.

Родной js-скрипт не привязан к полю rating по id, а работает, используя имена классов, поэтому корректно подхватит новые поля.

## Шаг 6. Фронтэнд, вывод списка отзывов, общего рейтинга

За вывод отвечают сниппеты ecMessages и ecThreadRating. Здесь все просто, смотрим родные чанки tpl.ecMessages.Row и tpl.ecThreadRating, по аналогии с существующим полем rating добавляем наши новые поля. Либо в родные чанки, либо создаем их копии.

## Шаг 7. Окончательная настройка

Итак, всю обвязку мы создали, осталось дело за малым, как сказать компоненту, что наши новые поля - это именно рейтинг?
Для этого имеется специальная системная настройка **ec\_rating\_fields**, в которой нужно через запятую указать список всех полей типа "рейтинг", в нашем случае это будет "rating,rating2,rating3".

Настройка после установки компонента не создается, для ее указания нужно перейти в раздел "Настройки системы", выбрать пространство имен "easycomm", далее нажать кнопку "Создать новый параметр".

::: danger
Без указания этой настройки рейтинг работать не будет!
:::

Для полей, указанных здесь, компонент будет считать средние показатели, а также проверять, что вводимые пользователями значения находятся в пределах от 0 до **ec\_rating\_max**.
