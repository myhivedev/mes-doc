---
id: DOC-03-02-AT-SYSTEMENUM
title: 'Тип конфигурационного артефакта — SystemEnum'
type: module-spec
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: configuration
holder: '@axelprosoft'
created_at: 2026-08-26 00:00
created_by: '@axelprosoft'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Тип конфигурационного артефакта — SystemEnum

[schemas]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/SystemEnumArtifactSchemas.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/SystemEnumArtifactSchemas.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[node-builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/SystemEnumNodeBuilder.cs
[baseline-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/SystemEnumBaselineBuilder.cs
[query-service]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Services/Core/ConfigurationQueryService.cs
[configuration-controller]: ../../../../src/Platform/DMP.Platform.Configuration/Api/Controllers/ConfigurationController.cs
[runtime-materializer]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Services/Materializers/RuntimeValueSetMaterializer.cs
[system-enum-response]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/Responses/ConfigurationSystemEnumItemsResponse.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ConfigurationBaselineAuthoringDslIntegrationTests.cs
[configuration-terms]: ../../../11_glossary/configuration_terms.md
[traceability]: ../90_traceability.md

## 1. Назначение

`SystemEnum` — корневой артефакт для перечисления, которое регистрируется
кодом платформы или прикладного модуля. Код задаёт состав перечисления,
технические коды значений и их смысл для runtime.

Configuration нужен здесь как слой опубликованного описания и локализации:
редактор Configuration может переопределить локализованный `SystemEnumValue.Title`, не
изменяя состав перечисления и его технические свойства.

В текущем MVP значения передаются в Configuration через canonical/baseline
builders, сохраняются как дочерние узлы `Values[]` и читаются из опубликованной
конфигурации. Отдельный механизм получения значений из внешнего источника не
нужен и этим документом не вводится.

## 2. Идентичность и граница

### 2.1. Идентичность

Корневой узел идентифицируется полем `ArtifactNode.Code`. В контексте этого
артефакта код является `SystemEnumCode`; поле
`ArtifactNode.ArtifactTypeCode` имеет значение `SystemEnum`.

Дочерний узел идентифицируется собственным `ArtifactNode.Code` внутри
коллекции `Values`. В контексте коллекции этот код является `ValueCode`.

`SystemEnumCode` и `ValueCode` — технические коды узлов, а не schema-свойства.
Они стабильны и не зависят от локализованного текста.

### 2.2. Входит в документ

- schema и структура `SystemEnum`;
- коллекция `Values[]` и schema `SystemEnumValue`;
- код регистрации перечисления и значений;
- единственное разрешённое configuration-переопределение — локализованный
  `SystemEnumValue.Title`;
- получение опубликованных значений через Configuration API;
- границы Configuration, runtime и frontend.

### 2.3. Не входит в документ

- создание новых системных значений через редактор Configuration;
- удаление, переименование или изменение `ValueCode`;
- изменение `IsFlags`, `NumericValue`, `Order` и `IsActive` через редактор Configuration;
- правила использования enum в бизнес-объекте;
- flags-операции и хранение числовой маски в Object Runtime;
- компонент выбора и рендерер конкретного frontend-приложения;
- внешний механизм получения значений.

Ссылка потребителя на `SystemEnumCode` не делает перечисление частью его
структуры. Потребитель получает значения по серверному контракту.

## 3. Место в Configuration

`SystemEnum` зарегистрирован в schema registry как корневой артефакт. Его schema
регистрирует коллекцию `Values`, элементы которой имеют тип
`SystemEnumValue`. ([schema][schemas])

`SystemEnumNodeBuilder` и `SystemEnumBaselineBuilder` создают перечисление и
 его значения в коде или в поставляемом базовом пакете конфигурации. Это не означает, что
пользователь может создавать произвольные значения в Configuration Editor.
([канонический сборщик][node-builder], [сборщик baseline][baseline-builder])

### 3.1. Владение по слоям

| Слой | Ответственность | Где описывается |
| --- | --- | --- |
| Код модуля или платформы | Регистрация `SystemEnum`, состав `Values[]`, `ValueCode`, `IsFlags`, `NumericValue`, `Order` и `IsActive` | Baseline/registration code и документ владельца модуля |
| Configuration | Сохранение effective-дерева и локализованного переопределения `SystemEnumValue.Title` | Этот документ и документы Configuration |
| Configuration API | Чтение опубликованных значений, локализация, фильтрация неактивных и сортировка | `ConfigurationController`, `ConfigurationQueryService` |
| Object Runtime | Получение значений для runtime-контракта и применение flags-смысла к данным объекта | Документы Object Runtime |
| Consumer-артефакт | Ссылка на `SystemEnumCode` и тип использования | `ObjectType`, `View`, `Action`, `Rule` |
| фронтенд-платформа и приложение | Отображение уже полученных кода и заголовка | фронтенд-платформа и владелец приложения |

## 4. Структура и схема `SystemEnum`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств.

```text
SystemEnum (ArtifactNode)
├── identity: ArtifactTypeCode = SystemEnum; Code = <SystemEnumCode>
├── properties: { Title, Description, IsFlags }
└── child collections:
    └── Values[0..n] → SystemEnumValue (Code = <ValueCode>)
        └── properties: { Title, Order, IsActive, NumericValue }
```

`SystemEnumValue` — дочерний узел, а не самостоятельный корневой артефакт.
Группы `metadata`, `ordering/state` и `runtime` не являются полями или
коллекциями; это только смысловая группировка свойств в объяснении.

В таблицах свойств ниже приведён полный текущий schema-контракт. Поля,
которые зарегистрированы в коде, но не входят в принятую модель, помечены как
«технический остаток». Они не должны появляться как доступные настройки
Configuration Editor.

### 4.2. Свойства корневого узла

Ниже приведён полный каталог свойств, зарегистрированных в текущей схеме
`SystemEnum`. `Required` означает обязательность свойства в схеме; это не
всегда означает обязательность пользовательского ввода в UI. Источники
значений и правила обновления описаны в разделе 5. Смысл значений и их
влияние указаны в этой же строке таблицы, а не в отдельном повторном каталоге.

Ниже находятся таблицы свойств артефакта. В каждой таблице одна строка
описывает одно свойство конкретного узла. Для всех таблиц используются
одинаковые столбцы:

- `Технический код` — имя свойства в schema и коде;
- `Русский смысл` — понятное русское название свойства;
- `Тип значения` — формат значения;
- `Обязательность / значение по умолчанию` — можно ли не задавать свойство и
  какое значение применяется при отсутствии значения;
- `Допустимые значения или источник` — допустимый формат или кодовый источник;
- `Смысл значения и влияние` — что означает значение и на что оно влияет;
- `Когда используется / переопределение` — владелец значения и возможность
  переопределения в Configuration.

Пометка «статический каталог вариантов» означает, что допустимые коды заданы в
[каталоге статических вариантов][static-options]. Это не список, который может
произвольно расширить клиентский редактор.

В последнем столбце используются следующие обозначения:

- `Default` — для свойства нет специального ограничения наследования;
- `InheritedOnly` — значение берётся от базового типа и не изменяется в
  производном типе;
- `CreationOnly` — значение задаётся только при создании;
- `OverrideAllowed` — производный тип может заменить унаследованное значение;
- `Schema filter` — сервер предлагает только значения из каталога, прошедшие
  условие;
- `Дискриминатор` — значение определяет допустимую форму или вид вложенного
  узла;
- `Все виды` — свойство применяется при любом допустимом значении условия;
- `Только <условие>` — свойство разрешено только при указанном условии;
- `Все экземпляры` — отдельного ограничения применения нет.

В столбце обязательности используются `Да`, `Нет` и `Условно`. `Условно`
означает, что обязательность зависит от другого свойства. Если после `Нет`
указано `/ значение`, это значение применяется по умолчанию.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название перечисления | `LocalizedText` | Нет; default задаёт код регистрации | Локализованные строки из кода или baseline | Отображаемое имя всего перечисления | Задаётся кодом; изменение через редактор Configuration не разрешено в принятой модели |
| `Description` | Описание перечисления | `LocalizedText` | Нет; default не задан | Локализованные строки из кода или baseline | Поясняет назначение перечисления | Задаётся кодом; изменение через редактор Configuration не разрешено |
| `SourceType` | Тип источника значений | `Enum` | Нет; default не задан | Допустимые коды в текущей schema не определены | Поле присутствует в текущей schema, но принятой модели не требуется | Технический остаток текущей schema; изменение через редактор Configuration не разрешено |
| `SourceCode` | Код источника значений | `ReferenceCode` | Нет; default не задан | Формат и каталог источника в текущем контракте не определены | Поле присутствует в текущей schema, но отдельный источник значений не используется | Технический остаток текущей schema; изменение через редактор Configuration не разрешено |
| `Policy` | Политика перечисления | `Json` | Нет; default не задан | Структура JSON в текущей schema не определена | Поле присутствует в текущей schema, но исполняемая policy для `SystemEnum` не используется | Технический остаток текущей schema; изменение через редактор Configuration не разрешено |
| `DisplayType` | Способ отображения значений | `Enum` | Нет; default не задан | `text`, `enumText`, `badge`, `link`, `number`, `boolean`, `switch`, `date`, `dateTime`; статический серверный каталог вариантов | Поле присутствует в текущей schema, но способ отображения задаётся рендерер-ом потребителя | Технический остаток текущей schema; изменение через редактор Configuration не разрешено |
| `IsFlags` | Перечисление допускает комбинацию значений | `Bool` | Нет; default `false` | `true` — значения образуют битовую маску; `false` — обычное перечисление | Определяет, может ли runtime объединять значения через числовую маску | Задаётся кодом; `InheritedOnly`; изменение через редактор Configuration не разрешено |

### 4.3. Вложенный тип `SystemEnumValue`

`SystemEnumValue` имеет собственный технический код `ValueCode` в
`ArtifactNode.Code`. В schema нет отдельного свойства `ValueCode`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название значения | `LocalizedText` | Нет; fallback API — `ValueCode` | Локализованные строки; переопределение из Configuration | Текст, который видит пользователь для данного кода | Единственное разрешённое переопределение редактора Configuration |
| `Order` | Порядок значения | `Number` | Нет; код или baseline обычно задаёт порядок | Число, обычно целое | Порядок значения в серверном ответе и списке выбора | Задаётся кодом; изменение через редактор Configuration не разрешено |
| `IsActive` | Значение активно | `Bool` | Нет; fallback API — `true` | `true` — значение возвращается обычно; `false` — только при `IncludeInactive=true` | Скрывает неактивное значение из обычного ответа, не удаляя его | Задаётся кодом; изменение через редактор Configuration не разрешено |
| `Tone` | Семантический тон отображения | `Enum` | Нет; default не задан | Каталог кодов в текущей schema не определён | Поле присутствует в текущей schema, но не входит в принятую модель | Технический остаток текущей schema; изменение через редактор Configuration не разрешено |
| `IconCode` | Код иконки значения | `ReferenceCode` | Нет; default не задан | Каталог иконок в текущем контракте не определён | Поле присутствует в текущей schema, но не входит в принятую модель | Технический остаток текущей schema; изменение через редактор Configuration не разрешено |
| `Variant` | Вариант отображения значения | `Enum` | Нет; default не задан | Каталог кодов в текущей schema не определён | Поле присутствует в текущей schema, но не входит в принятую модель | Технический остаток текущей schema; изменение через редактор Configuration не разрешено |
| `NumericValue` | Числовое значение flags | `Number` | Нет; требуется для значений, участвующих в битовой маске | Целое число из кода или baseline; для flags обычно отдельный бит | Сопоставляет код значения с битом числовой маски | Задаётся кодом; `InheritedOnly`; изменение через редактор Configuration не разрешено |

### 4.4. Структурные ограничения коллекции `Values[]`

- `Values` допускает ноль или более дочерних узлов.
- `ValueCode` должен быть уникален внутри одного `SystemEnum`.
- Набор значений и технические свойства поступают из code/baseline.
- Configuration может добавить локализованный `Title` к существующему
  значению, но не создаёт новый `ValueCode`.
- `SystemEnumValue` не используется как самостоятельный корневой артефакт.

## 5. Значения и источники

Таблицы ниже являются читаемым представлением текущей schema и её каталогов,
а не вторым источником истины. Типы, свойства и коллекции сверяются с
`<Artifact>ArtifactSchemas`, `ConfigurationSchemaPropertyCodes` и
`ConfigurationSchemaCollectionCodes` ([schema][schema], [коды свойств][property-codes],
[коды коллекций][collection-codes]). Технические значения сверяются с классом
`ConfigurationStaticValueCodes` и `ArtifactStaticOptionCatalog`
([коды статических значений][static-value-codes], [каталог вариантов][static-options]).
Динамические источники, их зависимости и режим обновления сверяются с
`ArtifactValueSourceCodes` и `ArtifactValueSourceDefinition`
([коды источников][value-source-codes], [описания источников][value-source-definition]).
Русский и английский смысл сверяется с `ConfigurationSchemaMetadataCatalog`
([каталог метаданных редактора][metadata-catalog]).

| Источник | Что предоставляет | Канал и обновление | Кто проверяет |
| --- | --- | --- | --- |
| Регистрация в коде или baseline | `SystemEnumCode`, `ValueCode`, состав значений, технические свойства и исходные заголовки | Поставка версии модуля или платформы | Code/baseline builder и Configuration import |
| Переопределение в Configuration | Локализованный `SystemEnumValue.Title` для существующего кода | Draft и опубликованная effective configuration | Editor policy и validation |
| Configuration API | Эффективные опубликованные значения, заголовки, порядок и активность | Серверный запрос; `IncludeInactive=true` добавляет неактивные значения | `ConfigurationQueryService` |
| рендерер фронтенда | Отображение полученного `Code` и `Title` | Клиентский rendering contract | фронтенд-платформа и приложение |

Frontend не является источником истины для состава значений. Изменение
`ValueCode`, `IsFlags`, `NumericValue`, `Order` или `IsActive` требует новой
регистрации/поставки кода, а не изменения через редактор Configuration.

## 6. Ограничения и зависимости

- Потребитель ссылается на стабильный `SystemEnumCode`.
- `ValueCode` уникален внутри родительского перечисления и используется как
  значение данных; `Title` не используется для сравнения и хранения.
- `IsActive=false` скрывает значение из обычного API-ответа, но не удаляет
  его из effective configuration.
- `Order` определяет порядок ответа API; при отсутствии порядка применяется
  сортировка по коду.
- При `IsFlags=true` runtime использует `NumericValue`; правила маски должны
  оставаться согласованными с code registration.
- Изменение состава системного перечисления поставляется через код и должно
  пройти обычный lifecycle baseline/import и публикации.
- `ValueSet` остаётся отдельным источником configurable data и не заменяет
  `SystemEnum`.

## 7. Операции над структурой

Операции ниже показывают границу между поставкой кода и Configuration Editor.

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| `SystemEnum` | Да, регистрация в коде или baseline; нет, произвольное создание в редакторе | Только поставка новой версии кода | Только поставка новой версии кода | Не применяется в редакторе | Код перечисления стабилен; публикация проверяет ссылки потребителей |
| `Values[] / SystemEnumValue` | Да, регистрация в коде или baseline; нет, создание нового значения в редакторе | Только `Title` в Configuration | Нет в редакторе; удаление поставкой кода | Только кодом или baseline через `Order` | `ValueCode`, `Order`, `IsActive`, `NumericValue` и `IsFlags` принадлежат коду |

## 8. Наследование и рассчитанный результат

`SystemEnum` участвует в effective configuration как корневой артефакт и
разрешается по общей цепочке версий и scope.

Для принятой модели эффективная конфигурация строится так:

1. берётся набор значений и технические свойства из кода или baseline;
2. для существующего `ValueCode` применяется локализованный
   `SystemEnumValue.Title` из Configuration, если он задан;
3. при отсутствии переопределения используется исходный заголовок, а затем
   fallback на `ValueCode`;
4. API исключает неактивные значения без `IncludeInactive` и сортирует ответ
   по `Order`, затем по коду.

`SystemEnumCode`, `ValueCode`, `IsFlags`, `NumericValue`, `Order` и `IsActive`
не являются полями для переопределения в Configuration.

## 9. Создание, проверка и публикация

### 9.1. Создание

Перечисление и его значения регистрируются кодом модуля или платформы и
попадают в Configuration через baseline/import. В Configuration Editor
допускается только создание локализованного override для `Title` существующего
`SystemEnumValue`.

### 9.2. Проверка

Проверяются:

- зарегистрированные типы `SystemEnum` и `SystemEnumValue`;
- типы schema-свойств;
- уникальность `SystemEnumCode` в module scope;
- уникальность `ValueCode` внутри одного перечисления;
- ссылки потребителей на существующий `SystemEnumCode`;
- наличие согласованных `NumericValue` для flags-сценариев.

Проверка редактора должна отклонять попытки изменить технические свойства или
создать новый `ValueCode`. В текущем общем редакторе это специальное правило
ещё не выделено отдельной политикой; расхождение описано в разделе 10 и
трассировке.

### 9.3. Публикация и чтение

После публикации Configuration API возвращает значения по запросу
`GET /api/platform/configuration/system-enums/items?systemEnumCode=...`.
Текущий response-контракт содержит `SystemEnumCode`, `Code`, `Title`, `Order`,
`IsActive`, `NumericValue` и `IsFlags`.
([контроллер][configuration-controller], [ответ][system-enum-response])

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `SystemEnum` и `SystemEnumValue` зарегистрированы в schema registry;
- `SystemEnum` имеет коллекцию `Values`;
- builders кода и baseline создают корневой и дочерние узлы;
- Configuration API читает опубликованные значения, локализует их, фильтрует
  неактивные и сортирует;
- `IsFlags` и `NumericValue` используются в текущих baseline/test-сценариях.

### За пределами текущего schema-контракта `SystemEnum`

В целевую модель не входят внешний механизм получения значений и изменение
технических свойств через Configuration Editor. Код владеет составом
перечисления и его техническими свойствами, а редактор Configuration может переопределять
только локализованный `SystemEnumValue.Title`. В текущей schema при этом
сохраняются дополнительные поля `SourceType`, `SourceCode`, `Policy`,
`DisplayType`, `Tone`, `IconCode` и `Variant`. Они не являются разрешёнными
настройками принятой модели и остаются техническим долгом: schema и editor
policy нужно привести к правилу из этого документа отдельной задачей.

## 11. Термины

| Русский термин | English term | Технический алиас | Краткое определение |
| --- | --- | --- | --- |
| Системное перечисление | System enum | `SystemEnum` | Зарегистрированное кодом перечисление со стабильным набором значений |
| Значение системного перечисления | System enum value | `SystemEnumValue` | Дочерний узел `Values[]` с кодом и заголовком |
| Код перечисления | Enum code | `SystemEnumCode` / `ArtifactNode.Code` | Стабильный код корневого узла |
| Код значения | Value code | `ValueCode` / `ArtifactNode.Code` | Стабильный код значения внутри перечисления |
| Флаговое перечисление | Flags enum | `IsFlags` | Перечисление, значения которого могут образовывать числовую маску |
| Числовое значение | Numeric value | `NumericValue` | Число, связывающее значение с битом flags-маски |
| Заголовок значения | Value title | `SystemEnumValue.Title` | Локализованный текст, который можно переопределить в Configuration |

Термины должны согласовываться с общим [глоссарием Configuration][configuration-terms].

## 12. Источники в коде и тестах

- [schema `SystemEnum` и `SystemEnumValue`][schemas];
- [коды типов][artifact-codes] и [коды свойств][property-codes];
- [canonical builder][node-builder] и [baseline builder][baseline-builder];
- [сервис чтения системных перечислений][query-service] и
  [Configuration API][configuration-controller];
- [ответ API][system-enum-response] и [runtime materializer][runtime-materializer];
- [интеграционный сценарий baseline и flags][schema-tests];
- [трассировка расхождения editor policy][traceability].

Источником истины для текущей schema является код schema registry. Принятая
политика владения и разрешённого override зафиксирована в разделе 10; до её
реализации общего редактора нельзя считать соответствующим этой политике.

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
