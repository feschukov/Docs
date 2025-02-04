# Логика работы

Компонент **Office** - это модульная система, в которой может быть сколько угодно частей (контроллеров).
В стандартном комплекте их 4:

- [Auth][0] - авторизация через email
- [Profile][1] - работа с профилем пользователя
- [miniShop2][2] - вывод личного кабинета MS2
- [RemoteAuth][3] - авторизация на один сайт через другой

В самом компоненте есть сниппет **Office**, который вызывает нужный контроллер и передает ему все указанные параметры.
Таким образом, возможные настройки, чанки, и прочие свойства зависят от контроллера и в сниппете не прописаны.

Например, вот вызов сразу всех 3х контроллеров на одной странице.

```modx
[[!Office? &action=`Auth`]]
[[!Office? &action=`Profile`]]
[[!Office? &action=`miniShop2`]]
```

Для стандартных контроллеров есть и специальные сниппеты с прописанными параметрами - просто для удобства. Внутри все они всё равно вызывают сниппет Office.

[![](https://file.modx.pro/files/7/a/6/7a691dcfa2bf7915716c61a3450e487cs.jpg)](https://file.modx.pro/files/7/a/6/7a691dcfa2bf7915716c61a3450e487c.png)

## Настройка контроллеров

В контроллер передаётся всё, что вы указали сниппету, и он сам решает, что ему нужно.
Все стандартные скрипты и стили необходимые для работы регистрируются через системные настройки, как у miniShop2.

[![](https://file.modx.pro/files/4/4/b/44b3499d03c306d34342bc1e9eb5808ds.jpg)](https://file.modx.pro/files/4/4/b/44b3499d03c306d34342bc1e9eb5808d.png)

Например, через системную настройку **office_extjs_css** можно изменить внешний вид личного кабинета miniShop2

Старый вид (`[[++assets_url]]components/office/css/main/lib/xtheme-modx.old.css`)

[![](https://file.modx.pro/files/9/6/4/9640c1d8fe2742274dba1c0238491001s.jpg)](https://file.modx.pro/files/9/6/4/9640c1d8fe2742274dba1c0238491001.png)

[![](https://file.modx.pro/files/e/d/6/ed6b56bc39dffbb68c8c9425399e17aas.jpg)](https://file.modx.pro/files/e/d/6/ed6b56bc39dffbb68c8c9425399e17aa.png)

Новый вид (`[[++assets_url]]components/office/css/main/lib/xtheme-modx.new.css`)

[![](https://file.modx.pro/files/5/a/b/5ab2fdf1b80cac13a660e07e319b57ees.jpg)](https://file.modx.pro/files/5/a/b/5ab2fdf1b80cac13a660e07e319b57ee.png)

[![](https://file.modx.pro/files/e/c/4/ec40dca2f9e8e2d620cf2a47ea5a4befs.jpg)](https://file.modx.pro/files/e/c/4/ec40dca2f9e8e2d620cf2a47ea5a4bef.png)

По умолчанию внешний вид выбирается в зависимости от установленной версии MODX - 2.2 или старше.

Вообще, контроллер - это обычный php класс, который наследует стандартный класс из Office.
Эти классы лежат в директории `/core/components/office/controllers/` и, благодаря модульной архитектуре, вы легко можете изменить любой из них.

Нужно просто сделать копию, переименовать и вызвать:

```modx
[[!Office?
  &action=`AuthCopy`
]]
```

Также в Office можно регистрировать контроллеры из устанавливаемых дополнений.

### Расширение сторонними компонентами

Сторонние дополнения могу регистрировать свои контроллеры путём добавлением пути к ним в системном параметре **office_controllers_paths**.

Для упрощения этой работы лучше всего использовать методы Office::**addExtenstion**() и Office::**removeExtension**() - принцип такой же, как и при регистрации моделей компонентов в MODX.

Вы можете посмотреть пример в [заготовке для разработки дополнений modExtra][6]:

Работа с записями modExtra в админке

[![](https://file.modx.pro/files/0/9/a/09acd54474eac1da1a18a45ef417b0c6s.jpg)](https://file.modx.pro/files/0/9/a/09acd54474eac1da1a18a45ef417b0c6.png)

[![](https://file.modx.pro/files/9/9/f/99f389219e64d198d80cf34de3bcc359s.jpg)](https://file.modx.pro/files/9/9/f/99f389219e64d198d80cf34de3bcc359.png)

Работа с записями modExtra снаружи сайта

[![](https://file.modx.pro/files/d/6/c/d6c064323f14c85809a852decd09b8a9s.jpg)](https://file.modx.pro/files/d/6/c/d6c064323f14c85809a852decd09b8a9.png)

[![](https://file.modx.pro/files/8/5/5/855490e75c5c93d364af3756d8d2bb92s.jpg)](https://file.modx.pro/files/8/5/5/855490e75c5c93d364af3756d8d2bb92.png)

Зарегистрированный контроллер выводится по своему имени. В данном случае это:

```modx
[[!Office? &action=`modExtra`]]
```

*Вам вовсе не обязательно использовать Ext JS, это просто пример возможностей.*

Вот, всё необходимое для реализации редактирования записей modExtra на фронтенде сайта, [одним коммитом][7].

После регистрации пути к контроллеру Office будет загружать его из указанной директории для всех действий, с ним связанных.
Вам не нужно ничего никуда копировать, вы можете поставлять и обновлять свой виджет внутри своего пакета.
По этому принципу уже работает [управление ключами в modstore.pro][8] и вывод [статистики продаж авторов][9].

Если же вы планируете использовать Ext JS в своих виджетах, то обратите внимание, что по умолчанию уже подгружаются кое-какие [улучшенные компоненты Office][10], типа таблицы со встроенным поиском - можно свободно их расширять.

Так как [modExtra][11] предназначен для разработки дополнений для MODX и уже поддерживает работу с Office, советую использовать в качестве примера именно его.

[0]: /components/office/controllers/auth
[1]: /components/office/controllers/profile
[2]: /components/office/controllers/orders-history-minishop2
[3]: /components/office/controllers/auth-remote
[6]: https://github.com/bezumkin/modExtra/blob/7b238647746fdd3443941a78fccc96ca9e96d76c/_build/resolvers/resolve.office.php
[7]: https://github.com/bezumkin/modExtra/commit/7b238647746fdd3443941a78fccc96ca9e96d76c
[8]: https://modstore.pro/cabinet/keys/
[9]: https://modx.pro/store/5343-statistics-for-authors-supplements/
[10]: https://github.com/bezumkin/Office/tree/master/assets/components/office/js/main/extjs
[11]: https://github.com/bezumkin/modExtra/
