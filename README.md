# Настройки истории данных

Обработка для настройки механизма истории данных. Написана для платформы 8.3.16; поддерживаются планы обмена, константы, справочники, документы, планы видов характеристик, планы счетов, планы видов расчета, регистры сведений, бизнес-процессы и задачи.

![Data History Settings (FirstBIT ERP)](Images/DataHistorySettings.png "Data History Settings (FirstBIT ERP)")

Есть русский и английский интерфейс. Код — на английском языке. Логика не опирается на БСП или SSLi (то есть будет работать в любой самописке без дополнительной адаптации).

## Как пользоваться

В левой части — дерево метаданных. Отображаются только те объекты метаданных, для которых платформа может вести историю.

Обработка читает настройки сначала из метаданных, потом — из информационной базы. Затем она определяет, для каких объектов история данных не ведется, и выделяет их в дереве серым цветом.

При выборе объекта настройки для него выводятся в панели справа. В неё можно включить или отключить историю для всего объекта, а также для каждого реквизита в отдельности, если они есть у объекта (у констант, например, нет).

Доступные команды:

- `Обновить настройки` строит дерево объектов метаданных и определяет их настройки. Эта операция выполняется при каждом запуске обработки. Может занимать приличное время (чем больше объектов в конфигурации — тем больше времени нужно).
- `Включить историю данных` и `Выключить историю данных` работают для выделенных объектов с учетом их иерархии. Например, можно включить историю для всех объектов в ветках «Справочники» и «Документы», выделив их и нажав «Включить историю данных».
- `Установить настройки метаданных по умолчанию` удаляет настройки механизма истории данных из информационной базы. После этого начнут работать стандартные настройки конфигурации, определенными её разработчиками. Как и включение-выключение, эта команда работает для выделенных объектов.

## Скрытые возможности

На форме обработки есть скрытые элементы, которые я посчитал не особенно полезными для пользователя. Вы можете включить их стандартной командой `Изменить форму`. 

- `Картинка настроек информационной базы`. Колонка дерева объектов метаданных. В ней будет отображаться шестеренка для тех объектов, у которых в информационной базе есть хоть какие-то настройки истории данных. Удобно, если вам нужно понять, для каких объектов поведение истории данных отличается от того, что заложил разработчик.
- `Показывать имена объектов метаданных`. Чекбокс в подвале формы. Если включен — вместо синонимов объектов (и их реквизитов) будут выводиться наименования. Удобно, если у вас бардак в синонимах и вы не можете понять, историю чего включаете или выключаете.
- `Показывать количество записей в объектах метаданных`. Чекбокс в подвале формы. Если включен — для каждого объекта метаданных будет выведено количество записей. Удобно, если вам нужно быстро оценить накладные расходы на ведение истории данных.

## Возможные проблемы

### Недостаток прав

Обработка предполагает, что у пользователя есть право «Изменение настроек истории данных» для всех объектов конфигурации, которые она поддерживает. Если это не так — будет глючить.

### Старая версия платформы

Если у вас устаревшая версия платформы, удалите из процедуры DefineMetadataObjectsCollections() определение объектов, которые ваша платформа не поддерживает. 

Например, для 8.3.12 нужно удалить определение планов обмена и констант. Если этого не сделать, обработка будет сыпать ошибками при каждом запуске.

### Доступны служебные объекты

По умолчанию обработка отображает все объекты, для которых можно вести историю. Однако многие из них нет никакого смысла версионировать: например, БСП'шный `ИдентификаторыОбъектовМетаданных`, целый ворох регистров для RLS и другие служебные таблицы. Если не хотите забивать базу мусором — лучше не давать пользователям возможности включать историю для таких таблиц. Кроме того, если ваша конфигурация может работать в режиме сервиса, то скрыть из обработки все неразделенные объекты — исключительно хорошая идея.

Скрыть часть объектов можно, добавив их в реквизит формы MetadataObjectsToIgnore. Это список значений — его можно заполнять, например, при создании формы:

```
MetadataObjectsToIgnore.Add(Metadata.InformationRegisters.AccessGroupTables.FullName());
MetadataObjectsToIgnore.Add(Metadata.InformationRegisters.AccessGroupValues.FullName());
```    

### Не поддерживаются стандартные реквизиты

Нельзя включить историю данных для:

1. Реквизита «Порядок» любого плана счетов;
2. Реквизита «НомерСтроки» табличной части любого бизнес-процесса.

Почему — неизвестно. В документации про это ни слова, а платформа просто молча генерит исключение. Скорее всего, разработчики платформы это вскоре исправят.

Пока что я отключил работу с этими реквизитами в коде — обработка игнорирует их и не выводит в дереве реквизитов. Если вам нужно отменить это решение, удалите из кода блоки If / EndIf, в условиях которых вызывается функция IsStandardAttributeWithName().