# Ищем заказ в базе RetailCRM

Для поиска заказа в API retailCRM существует метод ***ordersList***, в который передаются три параметра

1. Фильтр. Это обязатально массив каких-то данных (номер, заказа, email клиента, дата)
2. page - пагинация, номер страницы с результатами. Если результатов много (за один раз возвращается не больше 20 заказов) - параметр page позволяет обратиться с одинм запросом несколько раз, как в случае с пагинацией
3. limit - количество отдаваемых за один раз заказов. Не больше 20.

## Пример поиска по номеру заказа externalId

externalIds - должен быть массивом, даже если номер заказа один. Естественно можно указать несколько номеров

```php
$orders = $modRetailCrm->request->ordersList(array('externalIds' => [100]), 1, 20);
```

## Пример поиска заказа по email клиента

Таким образом можем получить все заказы одного клиента.

```php
$orders = $modRetailCrm->request->ordersList(array('email' => 'info@site.ru'), 1, 20);
```

## Пример поиска заказа по дате

Таким образом можем получить все заказы за конкретный промежуток времени. В данном случае нужно обязательно указать дату начала промежутка и дату окончания промежутка в формате Y-m-d

```php
$orders = $modRetailCrm->request->ordersList(array('createdAtTo' => '2019-01-06', 'createdAtFrom' => '2019-01-06'), 1, 20);
```

Больше информации по доступным фильтрам можно посмотреть в [документации](https://help.retailcrm.ru/Developers/ApiVersion5#get--api-v5-orders)
