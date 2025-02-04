# Загрузка файлов

На хостингах существует ограничение на максимальный размер данных в post-запросе, как правило, это 20МБ. При необходимости загрузить на сервер больший объем данных, их
нужно дробить. **SendIt** сделает это за вас. Чтобы это работало, сделайте такую разметку

```html:line-numbers
<form enctype="multipart/form-data" data-si-form="fileForm" data-si-preset="fileupload">
  <label data-fu-wrap>
    <input type="file" name="files" data-fu-field multiple placeholder="Выберите файл">
    <p data-si-error="file"></p>
  </label>
  <button type="submit">Отправить</button>
</form>
```

## Описание атрибутов

* **data-fu-wrap** - атрибут блока-обёртки.
* **data-fu-field** - атрибут поля загрузки.
* **data-si-error** - атрибут блока вывода ошибок валидации (необязательный).

## Порядок работы

Пользователь выбирает файлы, которые хочет загрузить.

::: tip
Атрибут **multiple** позволяет загружать сразу несколько файлов, зажав клавишу [[Ctrl]]. Загрузка будет последовательной.
:::

После этого файлы по очереди отправляются порциями на сервер. При этом сервер при получении первой порции, до начала
загрузки, проводит проверку файла на размер, тип и количество. Если хотя бы одна проверка не пройдена, файл не будет загружен, пользователь увидит уведомление об ошибке,
скрипт начнёт обработку следующего файла, если он есть. На время загрузки, кнопка отправки формы блокируется.

::: danger
Поле загрузки файлов и кнопка отправки формы ОБЯЗАТЕЛЬНО должны быть внутри формы с атрибутом **data-si-form**.
:::

После успешной загрузки путь к файлу будет добавлен в список загруженных файлов - скрытый input, который будет создан автоматически. Кроме того появятся кнопки с именем
файла, или другим заданным вами текстом, по нажатию на которые пользователь сможет удалить загруженный файл. На сервере файл помещается в папку с именем равным id сессии.

::: warning
Пользователь сможет удалить только свои файлы, так как перед удалением проверяется соответствие его session_id и пути к файлу.
:::

::: warning
Загруженные файлы хранятся на сервере до перезагрузки или закрытия страницы.
:::

## Параметры пресета

```php:line-numbers
'fileupload' => [
  'attachFilesToEmail' => 'files',
  'allowFiles' => 'filelist',
  'maxSize' => 6,
  'maxCount' => 2,
  'allowExt' => 'jpg,png',
  'portion' => 0.1,
]
```

Чтобы прикрепить файлы к письму, добавьте тэгу form атрибут **enctype="multipart/form-data"**, а так же добавьте в пресет такие параметры

* **attachFilesToEmail** - содержит имя поля для загрузки файлов (создаёт разработчик).
* **allowFiles** - содержит имя поля со списком загруженных файлов (добавляется автоматически); именно из этого поля вы сможете получить список файлов для дальнейшей
  обработки, например для перемещения в другую папку или пересылки в облачное хранилище.
* **maxSize** - максимально допустимый размер одного файла в мегабайтах.
* **maxCount** - максимально допустимое для загрузки количество файлов.
* **allowExt** - разрешённые форматы файлов.
* **portion** - размер одной порции файла в мегабайтах.

::: warning
Размер порции не должен превышать максимально разрешённый на вашем хостинге разме post-запроса
:::

::: danger
Все перечисленный параметры, кроме **attachFilesToEmail**, являются обязательными.
:::

### Уведомления

Тексты уведомлений берутся из словаря (см. *"Управление словарями"*) компонента по следующим ключам:

* **si_msg_file_open_err** - текст ошибки при которой невозможно сохранить файл на сервере, например когда закончилось свободное место на диске.
* **si_msg_loaded** - текст уведомления об успешном окончании загрузки.
* **si_msg_files_count_err** - текст ошибки о превышении количества файлов.
* **si_msg_file_size_err** - текст ошибки о превышении максимального размера файла.
* **si_msg_file_extention_err** - текст ошибки о недопустимом расширении файла.

## Конфигурация JavaScript

::: details Конфигурация по умолчанию

```js:line-numbers{3-42}
export default function returnConfigs() {
  return {
    FileUploader:{
      pathToScripts: './modules/fileuploader.js',
      formSelector: '[data-si-form]',
      rootSelector: '[data-fu-wrap]',
      fieldSelector: '[data-fu-field]',
      rootKey: 'fuWrap',
      presetKey: 'siPreset',
      sendEvent: 'si:send:after',
      pathKey: 'fuPath',
      pathAttr: 'data-fu-path',
      actionUrl: 'assets/components/sendit/web/action.php',
      layout: {
        list: {
          tagName: 'ul',
          classNames: ['file-list', 'list_unstyled', 'd_flex', 'flex_wrap', 'gap_col-10', 'pt-20'],
          selector: '.file-list'
        },
        item: {
          tagName: 'li',
          classNames: ['file-list__item'],
          parentSelector: '.file-list',
          selector: '.file-list__item'
        },
        btn: {
          tagName: 'button',
          classNames: ['file-list__btn', 'btn', 'py-5', 'px-20', 'ta_center', 'border-1', 'border_error', 'hover_bg_error', 'radius_pill', 'hover_color_light'],
          parentSelector: '.file-list__item',
          selector: '[data-fu-path="${filepath}"]',
          type: 'button',
          text: '${filename}&nbsp;X'
        },
        input: {
          classNames: ['file-list__input'],
          tagName: 'input',
          type: 'hidden',
          selector: '.file-list__input'
        }
      }
    }
  }
}
```

:::

|      Ключ       |                              Описание                               |                         Значение                          |
|:---------------:|:-------------------------------------------------------------------:|:---------------------------------------------------------:|
| `pathToScripts` |                 **./modules&nbsp;/fileuploader.js**                 | путь к модулю, указывается относительно файла *sendit.js* |
| `formSelector`  |                         **[data-si-form]**                          |                      селектор формы                       |
| `rootSelector`  |                         **[data-fu-wrap]**                          |        селектор блока-обёртки поля загрузки фалов         |
| `fieldSelector` |                         **[data-fu-field]**                         |               селектор поля загрузки файлов               |
|    `rootKey`    |                             **fuWrap**                              |      ключ свойства *dataset* селектора блока-обёртки      |
|   `presetKey`   |                            **siPreset**                             |         ключ свойства *dataset* с именем пресета          |
|   `sendEvent`   |                          **si:send:after**                          |                события завершения отправки                |
|    `pathKey`    |                             **fuPath**                              | ключ свойства *dataset* атрибута для записи пути к файлу  |
|   `pathAttr`    |                          **data-fu-path**                           |              атрибут для записи пути к файлу              |
|   `actionUrl`   | **assets&nbsp;/components&nbsp;/sendit&nbsp;/web&nbsp;/action.php** |              путь к файлу-приемнику запроса               |
|    `layout`     |                            ***объект***                             |   объект описывающий верстку списка загруженных файлов    |

## Вёрстка списка файлов

В параметре JS конфигурации `layout` есть 4 дочерних объекта

* **list** - описывает корневой элемент списка.
* **item** - описывает элемент списка, внутри которого будет располагаться кнопка удаления файла.
* **btn** - описывает кнопку удаления файла, параметр `text` содержит плейсхолдер *${filename}*, заменяемый на имя файла.
* **input** - описывает скрытый input для записи списка загруженных файлов, имя этого поля задаётся в пресете через параметр **allowFiles**
