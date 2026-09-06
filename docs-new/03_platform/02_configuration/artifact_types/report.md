---
id: DOC-03-02-ART-REPORT
title: 'Тип конфигурационного артефакта — Report'
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

# Тип конфигурационного артефакта — Report

[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ReportArtifactSchemas.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[static-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/ReportNodeBuilder.cs
[validator]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Services/ArtifactValidator.cs
[publish-validator]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Validation/Services/ReportPublishValidator.cs
[contract-service]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Services/Reports/ReportContractApplicationService.cs
[design-service]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Services/Reports/ReportDesignOperationsApplicationService.cs
[content-ref]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Content/ConfigurationContentRef.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ReportContractGeneratorIntegrationTests.cs
[publish-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ReportPublishValidationIntegrationTests.cs
[design-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ReportDesignOperationsIntegrationTests.cs
[traceability]: ../90_traceability.md

## 1. Назначение

`Report` — корневой конфигурационный артефакт, который описывает источник и
структуру отчётного результата: профиль представления, dataset, поля,
параметры, фильтры и design-контракт.

В текущем Configuration MVP `Report` хранит design-time описание и ссылки на
содержимое design/schema/sample в configuration blob store. Выполнение запроса,
рендеринг и доставка результата не являются частью schema `Report` и принадлежат
соответствующим runtime-владельцам.

## 2. Идентичность и граница

### 2.1. Идентичность

`ArtifactNode.Code` является стабильным кодом отчёта. В module context он
используется как `ReportCode`, участвует в ссылках и в `OriginKey`.

`ReportDesign` имеет код `Design`, но не является самостоятельным root artifact:
он хранится в коллекции `Report.Design`.

### 2.2. Входит в документ

- schema-контракт корневого `Report`;
- дочерний `ReportDesign` и его content references;
- коллекции `ReportField`, `ReportParameter`, `ReportFilter`;
- значения `PresentationKind` и `DesignEngine`, подтверждённые кодом;
- генерация schema/sample/default design и design operations Configuration;
- проверки ссылок, design blob и совместимости schema version при публикации.

### 2.3. Не входит в документ

- исполнение dataset/query и получение строк данных;
- рендеринг через BIRT или другой внешний движок;
- доставка результата, формат output и фоновые задания;
- UI shell, кнопки запуска и привязки `Output`;
- layout и стили внутри содержимого `.rptdesign`;
- полный HTTP-контракт Reporting/Output runtime.

## 3. Место в Configuration

`Report` принадлежит Configuration как schema, authoring artifact и набор
design-time операций. Configuration может сгенерировать или сохранить
контракт данных и design-файлы, но не выполняет сам отчёт.

| Владелец или слой | Ответственность | Где описано |
| --- | --- | --- |
| Configuration | Schema `Report`, authoring, versioning, publication, fields/parameters/filters, design refs и генерация design-time contracts | Этот документ, [schema][schema], [publish validator][publish-validator] |
| Dataset / Read Query Capability | Источник строк, чтение данных и query semantics | Отдельная область или владелец capability; не `Report` |
| Reporting runtime | Выполнение отчёта и интерпретация design/data contract | Будущая или отдельная Reporting Output area |
| Output | Формат, запуск, доставка и output audit | `Output` и владелец Output runtime |
| фронтенд-платформа | Общий shell, editor integration и preview viewer | фронтенд-платформа |
| Domain module | Регистрация report/dataset и предметный набор полей | Baseline package и документ прикладного модуля |

`DatasetCode` связывает `Report` с источником данных, но не переносит query
модель в Configuration schema. `TemplateCode` в текущем schema сохранён как
read-only совместимость и не заменяет `Report.Design`.

## 4. Структура и схема `Report`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств.

```text
Report (ArtifactNode)
├── identity: ArtifactTypeCode = Report; Code = <ReportCode>
├── properties: { PresentationKind, DesignEngine, SchemaVersion, Title,
│                Description, ObjectTypeCode, DatasetCode, TemplateCode,
│                Purpose, FeatureProviderCode }
└── child collections:
    ├── Design[0..1] → ReportDesign (Code = Design)
    ├── Fields[0..n] → ReportField
    ├── Parameters[0..n] → ReportParameter
    └── Filters[0..n] → ReportFilter
```

`Design` — singleton-коллекция: для одного `Report` допускается не более одного
`ReportDesign`. В текущей schema нет иерархических дочерних коллекций.

### 4.2. Свойства корневого узла

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `PresentationKind` | Профиль представления отчёта | `Enum` | Да; default create-dialog `Tabular` | `Tabular`, `Card`; статический каталог Configuration | Выбирает профиль структуры данных для генерации report contract | Creation-only; влияет на VODT profile и generated schema |
| `DesignEngine` | Движок design | `Enum` | Да; builder default `Birt` | `Birt`; статический каталог Configuration | Определяет обработчик design-контракта | Creation-only; для `Birt` требуется `Design` |
| `SchemaVersion` | Версия контракта данных | `String` | Нет в schema; create-dialog задаёт `v1` | Непустая строка версии | Идентифицирует версию generated schema/sample | Creation-only; изменение после создания относится к design/schema operations |
| `Title` | Название отчёта | `LocalizedText` | Нет | Локализованные значения | Показывает отчёт в каталоге и редакторе | Обычное локализуемое свойство, если не заблокировано policy |
| `Description` | Описание отчёта | `LocalizedText` | Нет | Локализованные значения | Объясняет назначение отчёта | Обычное локализуемое свойство |
| `ObjectTypeCode` | Код типа объекта | `ReferenceCode` | Нет; creation-only | `Catalog.ObjectTypes` | Задаёт object context отчёта | Задаётся при создании; не является источником query-модели |
| `DatasetCode` | Код набора данных | `ReferenceCode` | Да; creation-only | Зарегистрированный dataset или read capability | Определяет источник данных отчёта | Задаётся при создании; существование проверяется владельцем dataset/publish validation |
| `TemplateCode` | Устаревший код шаблона | `ReferenceCode` | Нет; read-only | Значение совместимости, если используется | Сохраняет старую ссылку; не определяет текущий BIRT design | Не редактируется обычным editor; основной design — `Design` |
| `Purpose` | Назначение отчёта | `String` | Нет; read-only | Непустая строка, если задана | Дополнительное описание назначения | Только если поставлено registry/baseline или другим доверенным источником |
| `FeatureProviderCode` | Код поставщика функции | `ReferenceCode` | Нет; read-only | Зарегистрированный provider, если используется | Связывает отчёт с владельцем feature registration | Не является design engine и не задаёт рендерер |

`Required` в schema означает обязательность сохранённого свойства. Значение
`Tabular`, `Birt` или `v1`, заданное create-dialog/builder, является default
authoring flow и не заменяет правило schema.

### 4.3. Вложенный тип `ReportDesign`

`ReportDesign` — принадлежащий `Report` дочерний узел. Его свойства описывают
метаданные файлов и ссылки на blob store; содержимое самих файлов не является
значением schema property.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Engine` | Движок design | `Enum` | Нет в schema; для BIRT должен совпадать с `Report.DesignEngine` | `Birt` | Показывает движок принадлежащего design | Только `Design`; проверяется при публикации |
| `DesignFileName` | Имя файла design | `String` | Нет в schema; обязательно для BIRT publish | Имя `.rptdesign` файла | Идентифицирует design content | Заполняется design operations или baseline |
| `ContentRef` | Ссылка на design content | `String` | Нет в schema; обязательно для BIRT publish | Разбираемый `ConfigurationContentRef` | Указывает на blob `.rptdesign` | Проверяется в blob store и по hash |
| `ContentHash` | Хэш design content | `String` | Нет | Хэш содержимого blob | Позволяет проверить целостность | Заполняется при import/generation |
| `SchemaFileName` | Имя schema-файла | `String` | Нет в schema; обязательно для BIRT publish | Имя XSD-файла | Идентифицирует generated data schema | Заполняется contract operations |
| `SchemaContentRef` | Ссылка на schema content | `String` | Нет в schema; обязательно для BIRT publish | Разбираемый `ConfigurationContentRef` | Указывает на blob XSD | Проверяется в blob store и по hash |
| `SchemaContentHash` | Хэш schema content | `String` | Нет | Хэш содержимого blob | Позволяет проверить целостность XSD | Заполняется contract operations |
| `SampleFileName` | Имя sample-файла | `String` | Нет в schema; обязательно для BIRT publish | Имя XML-файла | Идентифицирует sample data | Может быть выведено из `SchemaFileName` |
| `SampleContentRef` | Ссылка на sample content | `String` | Нет в schema; обязательно для BIRT publish | Разбираемый `ConfigurationContentRef` | Указывает на blob sample XML | Проверяется в blob store и по hash |
| `SampleContentHash` | Хэш sample content | `String` | Нет | Хэш содержимого blob | Позволяет проверить целостность sample | Заполняется contract operations |
| `SchemaVersion` | Версия design-контракта | `String` | Нет в schema; обязательно для BIRT publish | Непустая строка версии | Связывает design с XSD/sample version | Должна соответствовать текущему report contract |
| `GeneratedFromBaseline` | Сгенерирован из baseline | `Bool` | Нет | `true` — сгенерирован baseline; `false` — импортирован или изменён отдельно | Показывает происхождение design | Устанавливается generation/import operations |
| `UpdatedAt` | Время изменения design | `String` | Нет | Строковое представление UTC timestamp | Фиксирует последнюю операцию design | Заполняется design operations |
| `UpdatedBy` | Кто изменил design | `String` | Нет | Идентификатор или код субъекта операции | Фиксирует источник изменения | Заполняется design operations |

### 4.4. Вложенный тип `ReportField`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название поля отчёта | `LocalizedText` | Нет | Локализованные значения | Заголовок поля в report contract/design | Для каждого `ReportField` |
| `Description` | Описание поля | `LocalizedText` | Нет | Локализованные значения | Поясняет назначение поля | Для каждого `ReportField` |
| `MemberCode` | Код поля объекта | `ReferenceCode` | Нет; вместе с `Expression` требуется ровно один источник | `ObjectType.Members` | Берёт значение поля объекта или dataset row | Взаимоисключимо с `Expression` |
| `Expression` | Выражение поля | `String` | Нет; вместе с `MemberCode` требуется ровно один источник | Поддержанный format выражений report generator | Вычисляет значение поля | Взаимоисключимо с `MemberCode`; полный expression engine не принадлежит schema |
| `ValueType` | Тип значения поля | `Enum` | Нет | Значения `ConfigurationDataTypeCodes`, если заданы | Определяет тип generated contract value | Используется генератором и design; конкретная совместимость с dataset проверяется отдельно |
| `Format` | Формат значения поля | `String` | Нет | Формат, поддержанный генератором/design engine | Задаёт форматирование значения | Не определяет общий рендерер runtime |
| `Order` | Порядок поля | `Number` | Нет | Число порядка | Определяет порядок полей в contract/design | В пределах `Fields[]` |

### 4.5. Вложенный тип `ReportParameter`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название параметра | `LocalizedText` | Нет | Локализованные значения | Имя параметра для editor/design | Для каждого параметра |
| `Description` | Описание параметра | `LocalizedText` | Нет | Локализованные значения | Поясняет назначение параметра | Для каждого параметра |
| `DataType` | Тип параметра | `Enum` | Да | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid` | Определяет формат входного значения | Проверяется generated contract |
| `Required` | Параметр обязателен | `Bool` | Нет; schema default не задан | `true` — значение требуется; `false` — значение необязательно | Влияет на обязательность входа отчёта | Runtime enforcement принадлежит report/output consumer |
| `DefaultValue` | Значение по умолчанию | `String` | Нет | Строковое представление значения `DataType` | Используется при отсутствии входного значения | Должно быть совместимо с `DataType` |
| `ValueSetCode` | Код набора значений | `ReferenceCode` | Нет | `Catalog.ValueSets` | Ограничивает или предоставляет варианты параметра | Взаимоисключимо с `SystemEnumCode` |
| `SystemEnumCode` | Код системного перечисления | `ReferenceCode` | Нет | `Catalog.SystemEnums` | Предоставляет системные варианты параметра | Взаимоисключимо с `ValueSetCode` |
| `Order` | Порядок параметра | `Number` | Нет | Число порядка | Определяет порядок параметров | В пределах `Parameters[]` |

### 4.6. Вложенный тип `ReportFilter`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название фильтра | `LocalizedText` | Нет | Локализованные значения | Показывает фильтр в editor/design | Для каждого фильтра |
| `FieldCode` | Код поля фильтра | `ReferenceCode` | Да | `Report.Fields` или разрешённое поле object type | Определяет поле, к которому применяется условие | Проверяется после построения набора допустимых полей |
| `Operator` | Оператор фильтра | `Enum` | Да | `Equals`, `NotEquals`, `Contains`, `StartsWith`, `EndsWith`, `GreaterThan`, `GreaterThanOrEqual`, `LessThan`, `LessThanOrEqual`, `In`, `HasFlag`, `FlagsAny`, `FlagsAll`, `FlagsExact`, `Between`; статический каталог | Определяет способ сравнения | Должен поддерживаться генератором/consumer; значения каталога не означают готовность каждого runtime |
| `ParameterCode` | Код параметра фильтра | `ReferenceCode` | Нет | `Report.Parameters` | Делает значение фильтра входным параметром | Взаимоисключимо с `Value` |
| `Value` | Постоянное значение фильтра | `String` | Нет | Строковое значение, совместимое с полем | Задаёт фиксированное условие | Взаимоисключимо с `ParameterCode` |
| `Required` | Фильтр обязателен | `Bool` | Нет; schema default не задан | `true`, `false` | Показывает обязательность значения фильтра | Enforcement зависит от report consumer |
| `Order` | Порядок фильтра | `Number` | Нет | Число порядка | Определяет порядок фильтров | В пределах `Filters[]` |

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

### 5.1. Статические каталоги

| Свойство | Источник | Допустимые значения | Получение и обновление |
| --- | --- | --- | --- |
| `PresentationKind` | Backend static option catalog | `Tabular`, `Card` | Server catalog; обновляется изменением backend-контракта |
| `DesignEngine` | Backend static option catalog | `Birt` | Server catalog; в текущем MVP другого engine нет |
| `ReportFilter.Operator` | Backend static option catalog | Коды операторов из таблицы `4.6` | Server catalog; генератор и consumer обязаны проверять фактическую поддержку |
| `ReportParameter.DataType` / `ReportField.ValueType` | Backend configuration data type catalog | Коды `ConfigurationDataTypeCodes` | Server/static contract; клиент не расширяет список самостоятельно |
| `ValueSetCode`, `SystemEnumCode`, `MemberCode`, `FieldCode`, `DatasetCode` | Catalog или текущий `Report` context | Зарегистрированные коды | Server-provided options; сохранённая ссылка повторно проверяется server validation |

### 5.2. Generated schema и sample XML

Configuration может сгенерировать schema и sample XML для текущего `Report`.
Генератор использует `PresentationKind`, `Fields[]`, `Parameters[]` и
`Filters[]`; результат возвращается как design-time artifact и может быть
сохранён в blob store через `ReportDesign`.

Sample values могут быть переданы отдельно для system attributes, parameters,
rows, object и nested collections. Это значения для генерации sample, а не новые
свойства `Report`.

### 5.3. Design content

Default design генерируется Configuration как содержимое `.rptdesign` и
сохраняется через `ReportDesign.ContentRef`. Импортированный design проходит
проверку XML и соответствия колонок `Report.Fields[]`. Blob content не следует
дублировать в markdown-документе.

### 5.4. Локализация

`Title` и `Description` корневого и дочерних узлов имеют тип `LocalizedText`.
Локализация хранится общим механизмом Configuration properties; этот документ
фиксирует только применимость локализуемого значения, а не persistence-модель
локализаций.

## 6. Ограничения и зависимости

- `Report` не исполняет dataset и не выбирает query semantics самостоятельно.
- Для `DesignEngine = Birt` нужен один `Report.Design`, а при публикации должны
  быть заполнены design/schema/sample metadata и существовать соответствующие
  blobs.
- `Report.Design.Engine`, если задан, должен совпадать с корневым
  `Report.DesignEngine`.
- `ReportField` должен задавать `MemberCode` или `Expression`; одновременное
  задание обоих источников запрещено schema rule.
- `ReportFilter.FieldCode` должен ссылаться на допустимое поле; `ParameterCode`
  должен ссылаться на существующий `ReportParameter`.
- `ReportParameter.ValueSetCode` и `SystemEnumCode`, а также `ReportFilter`
  `ParameterCode` и `Value` не должны задаваться одновременно.
- Изменение полей отчёта может потребовать новой `SchemaVersion`; publish
  validator предупреждает о breaking change без изменения версии.
- `TemplateCode` не является текущим механизмом визуального шаблона отчёта;
  основной design хранится в `Report.Design`.

## 7. Операции над структурой

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| Корневой `Report` | Да, через root editor/builder | Да, с учётом schema и edit policy | По общим правилам Configuration | Не применяется | `PresentationKind`, `DesignEngine`, `SchemaVersion`, `ObjectTypeCode`, `DatasetCode` имеют ограничения создания/изменения |
| `Design[0..1]` | Через builder/design operations | Update owned singleton | Поддерживается общим удалением поддерева | Не применяется | Максимум один узел; для BIRT publish обязателен |
| `Fields[]` | Да | Да | Да, subtree | Да | Поле должно иметь `MemberCode` или `Expression` |
| `Parameters[]` | Да | Да | Да, subtree | Да | `DataType` обязателен; ValueSet/SystemEnum взаимно исключаются |
| `Filters[]` | Да | Да | Да, subtree | Да | `FieldCode` и `Operator` обязательны; ссылки проверяются сервером |
| Свойства узлов | Да, если разрешены schema | Да, если нет `ReadOnly`/`CreationOnly` | Сброс или удаление по общей property policy | Не применяется | Изменение design file refs выполняют design operations |

## 8. Наследование и рассчитанный результат

`Report` может быть прочитан из effective Configuration chain вместе с его
`Fields`, `Parameters`, `Filters` и `Design`. Если `Report.Design` унаследован,
design operations требуют локальный override в текущем draft перед импортом или
публикацией изменённого design.

Сгенерированные schema, sample XML и default design являются производными
результатами operations. Они не являются дополнительными schema properties
корневого `Report`.

## 9. Создание, проверка и публикация

### 9.1. Создание

Root editor создаёт минимальный `Report` с `ReportCode`, `PresentationKind`,
`DesignEngine`, `DatasetCode` и, если задан, `ObjectTypeCode`. Текущий create
flow может создать начальный `Report.Design`, но не обязан создавать все
`Fields`, `Parameters`, `Filters` и blobs design-time content.

### 9.2. Проверка

Проверка выполняется на нескольких уровнях:

1. Schema/`ArtifactValidator` проверяет типы, обязательные свойства,
   взаимное исключение ссылок и базовую структуру `Report`.
2. `ReportPublishValidator` проверяет object/field references, BIRT design,
   content refs, наличие blobs, hashes и совместимость schema version.
3. Design validator проверяет импортированный `.rptdesign` и его соответствие
   полям отчёта.

### 9.3. Публикация и design operations

Configuration предоставляет операции генерации schema, sample XML и default
design, их публикации, export/import design bundle, validation design и создания
preview session. Эти операции защищены permission `Configuration.Entry.Edit`.

Публикация отчёта не означает выполнение отчёта. Она означает, что
конфигурация и необходимые design-time content refs прошли проверки и могут
быть использованы внешним report/output runtime.

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `Report` зарегистрирован как root artifact;
- зарегистрированы `ReportDesign`, `ReportField`, `ReportParameter` и
  `ReportFilter`;
- `PresentationKind` имеет значения `Tabular` и `Card`;
- `DesignEngine` текущего каталога — `Birt`;
- существует builder, root editor, schema/publish validation и design operations;
- schema, sample XML и default design могут генерироваться Configuration;
- design/schema/sample content хранится через configuration blob references;
- релевантные сценарии покрыты integration tests.

### За пределами текущего schema-контракта `Report`

- исполнение dataset/query и общий `Read Query Capability`;
- runtime rendering и delivery результата;
- полный контракт `Output` и его launch bindings;
- редактирование `.rptdesign` внутри Configuration Editor;
- другие значения `DesignEngine` кроме `Birt`;
- автоматическая синхронизация каждого изменения `Fields[]` с вручную изменённым
  design без отдельной design operation.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Отчёт | Report | Корневой конфигурационный артефакт описания отчётного результата |
| Поле отчёта | ReportField | Поле или выражение, включённое в contract отчёта |
| Параметр отчёта | ReportParameter | Входное значение, используемое отчётом или фильтром |
| Фильтр отчёта | ReportFilter | Условие ограничения данных отчёта |
| Design отчёта | ReportDesign | Дочерний узел с метаданными и ссылками на design content |
| Профиль представления | PresentationKind | `Tabular` или `Card` |
| Движок design | DesignEngine | Технический обработчик design; текущий код — `Birt` |
| Ссылка на содержимое | ContentRef | Ссылка Configuration на blob design/schema/sample |

## 12. Источники в коде и тестах

Основные подтверждения: [artifact type codes][artifact-codes], [property codes][property-codes],
[collection codes][collection-codes], [schema][schema], [static catalog][static-catalog],
[static codes][static-codes], [builder][builder], [validator][validator],
[publish validator][publish-validator], [contract service][contract-service],
[design service][design-service] и [content ref][content-ref].

Релевантные тесты: [contract generator tests][schema-tests],
[publish validation tests][publish-tests] и [design operations tests][design-tests].
Источники переходного материала и нерешённые границы перечислены в
[трассировке Configuration][traceability].

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
