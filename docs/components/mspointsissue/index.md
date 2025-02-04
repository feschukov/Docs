---
title: msPointsIssue
description: Пункты выдачи заказов для miniShop2
logo: https://modstore.pro/assets/extras/mspointsissue/logo-lg.jpg
author: vgrish
modstore: https://modstore.pro/packages/delivery/mspointsissue

items: [
  {
    text: 'Сниппеты',
    link: 'snippets/',
    items: [
      { text: 'msPointsIssue.Order', link: 'snippets/mspointsissue-order' },
    ],
  },
  {
    text: 'Интерфейс',
    link: 'interface/',
    items: [
      { text: 'Заказы', link: 'interface/orders' },
      { text: 'Настройка', link: 'interface/settings' },
    ],
  },
]

dependencies: miniShop2
---

# msPointsIssue

Для работы вам нужен MODX не ниже **2.3** и PHP не ниже **5.4**.

## Описание

**msPointsIssue** - компонент реализует фунцкионал Пункты Выдачи Заказов для магазина MiniShop2

![msPointsIssue](https://file.modx.pro/files/4/f/d/4fd49fd13aea3c258b83d37597d5b0bc.png)

## Особенности

- работа только с новым miniShop2 (version =>2.4.0-beta2)
- сниппет для расчета стоимости заказа
- адаптация с [geonames](http://www.geonames.org) для облегчения первичного наполнения ПВЗ

## Демо сайт

Доступен демо сайт `http://s13938.h10.modhost.pro/`
Логин и пароль для входа в [админку](http://s13938.h10.modhost.pro/manager/): `test`

## Установка

- [Подключите наш репозиторий](https://modstore.com)
- Установите **pdoTools** - это библиотека для быстрой работы с БД и оформлением, необходима для многих компонентов
- Установите **Theme.Bootstrap** - это тема оформления Twitter Bootstrap для MODX, под неё заточены стандартные чанки
- Установите **miniShop2** - это магазин на основе которого реализован функционал заказа c оплатой
- Установите **msPointsIssue**

Для тестирования можно использовать [наш хостинг](https://modhost.pro), на нём эти дополнения можно выбрать прямо при создании сайта.
После установки компонента необходимо:

- активировать нужные вариант доставки
- назначить необходимые варианты оплаты
- наполнить ПВЗ

Доступно 3 метод доставки:

- Своя доставка. Обычная доставки не обрабатывается компонентом, прописаны лишь свойства в `properties` для корректного отображения формы заказа.
- Курьер. Доставка обрабатывается компонентом. Режим - `point`;
- Терминал. Доставка обрабатывается компонентом. Режим - `terminal`;

Вы можете создать свои собственные типы доставки.

## Настройка

Все сниппеты msPointsIssue работают при помощи pdoTools и рассчитывают на использование [Fenom][010103] в чанках.

Это позволяет:

- сократить общее количество чанков
- повысить удобство работы
- ускорить работу
- делать более сложные чанки, за счёт продвинутой проверки условий через функции Fenom

Необходимо включить следующие настройки:

- `pdotools_fenom_modx`
- `pdotools_fenom_parser`
- `pdotools_fenom_php`

![Настройка](https://file.modx.pro/files/6/1/c/61c556239adbb2d257654c68ec07f9a5.png)

Далее смотрите рекомендации из раздела [Быстрый старт MiniShop2][010200]

Отличия только при создании страницы заказа, нужно указать сниппет `msPointsIssue.Order`

```modx
[[!msCart]] <!-- Вывод корзины заказа -->
[[!msPointsIssue.Order]] <!-- Форма оформления заказа, скрывается после его создания -->
[[!msGetOrder]] <!-- Вывод информации о заказе, показывается после его создания -->
```

[010103]: /components/pdotools/parser
[010200]: /components/minishop2/quick-start
