---
id: DOC-03-02-ART-OUTPUT
title: 'Тип конфигурационного артефакта — Output'
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

# Тип конфигурационного артефакта — Output

[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/OutputArtifactSchemas.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[static-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[baseline-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/OutputBaselineBuilder.cs
[node-builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/OutputNodeBuilder.cs
[editor-session]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Services/Editor/ConfigurationRootArtifactEditorSessionService.cs
[editor-projector]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Services/Editor/CanonicalArtifactEditorReadProjectorBase.cs
[runtime-provider]: ../../../../src/Hosts/DMP.Platform.Api/Composition/RuntimeReportDefinitionProvider.cs
[runtime-materializer]: ../../../../src/Hosts/DMP.Platform.Api/Composition/RuntimeOutputLaunchBindingMaterializer.cs
[runtime-service]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/GenerateOutputService.cs
[runtime-mapping]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/OutputParameterMappingResolver.cs
[runtime-models]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Abstractions/Reports/RuntimeOutputEngineModels.cs
[runtime-request]: ../../../../src/Platform/DMP.Platform.Contracts/Runtime/Requests/GenerateOutputRequest.cs
[runtime-response]: ../../../../src/Platform/DMP.Platform.Contracts/Runtime/Responses/GenerateOutputResponse.cs
[runtime-format]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Services/Reports/ReportOutputFormatMapper.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactSchemaRegistryIntegrationTests.cs
[builder-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/OutputNodeBuilderIntegrationTests.cs
[editor-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactEditorSelectionPolicyIntegrationTests.cs
[traceability]: ../90_traceability.md

## 1. Назначение

`Output` — корневой конфигурационный артефакт, который связывает источник
результата с форматом, способом запуска и базовыми настройками выдачи.
Configuration хранит и публикует эту схему. Исполнение результата выполняет
runtime-владелец Output/Reporting.

В текущем коде runtime-операция `GenerateOutput` получает опубликованный
`Output`, разрешает связанный `Report`, передаёт параметры в отчётный runtime и
возвращает содержимое результата. Это не означает, что все значения schema и
все каналы доставки уже исполняются: фактически поддержаны только
`SourceType = Report` и режимы `Inline`/`Download`.

## 2. Идентичность и граница

### 2.1. Идентичность

`ArtifactNode.Code` является стабильным кодом `Output`. В baseline-поставке
полная идентичность записи включает `ModuleCode`, `ObjectTypeCode` и код
артефакта; они участвуют в `OriginKey` и контексте effective resolution.

Дочерние узлы имеют собственные коды, но не являются корневыми артефактами:
они хранятся в коллекциях принадлежащего им `Output`.

### 2.2. Входит в документ

- schema корневого `Output` и шести дочерних типов;
- свойства источника, формата, режима запуска и доступа;
- места запуска и сопоставление параметров источника;
- настройки каналов, файла, выполнения и безопасности как значения schema;
- операции schema, baseline authoring, effective resolution и editor options;
- фактическая граница между Configuration schema и Output runtime.

### 2.3. Не входит в документ

- query semantics и чтение строк `Dataset`;
- выполнение и design-модель `Report`;
- генерация PDF/Excel/CSV, хранение файла и внешний рендерер;
- отправка электронной почты, storage, печать и фоновые задания;
- общий frontend shell и реализация кнопок запуска;
- окончательная runtime-семантика ещё неисполняемых полей и каналов.

## 3. Место в Configuration

`Output` принадлежит Configuration как каноническая schema-модель и часть
versioned/effective configuration. Runtime получает опубликованную проекцию,
но не изменяет конфигурационный артефакт.

| Владелец или слой | Ответственность | Где описано |
| --- | --- | --- |
| Configuration | Schema, authoring, baseline import, versioning, publication и effective resolution `Output` | Этот документ, [schema][schema], [baseline builder][baseline-builder] |
| Reporting / Output runtime | Разрешение опубликованного `Output`, выполнение связанного `Report`, форматирование результата и возврат response | [runtime provider][runtime-provider], [runtime service][runtime-service] |
| фронтенд-платформа | Общая кнопка/панель запуска и обработка runtime response | Отдельная frontend-документация |
| Tenant/Security | Проверка permission и контекста пользователя/tenant/site | Runtime boundary и Tenant/Security |
| Domain module | Регистрация `Output`, `Report` и предметных источников в baseline package | [baseline builder][baseline-builder] и документ модуля |

`Output.SourceCode` связывает артефакт с источником по `SourceType`. Ссылка не
переносит модель источника в `Output`: `Report`, `View` и Dataset/Read Query
описаны своими владельцами.

## 4. Структура и схема `Output`

### 4.1. Состав схемы

Ниже показаны реальные узлы и коллекции schema. `properties: { ... }` — это
словарь свойств узла, а не дочерняя коллекция.

```text
Output (ArtifactNode)
├── identity: ArtifactTypeCode = Output; Code = <OutputCode>
├── properties: { Title, Description, SourceType, SourceCode, Format,
│                DeliveryMode, ObjectTypeCode, IsDiscoverable, CategoryCode,
│                PermissionCode, FeatureProviderCode }
└── child collections:
    ├── LaunchBindings[0..n] -> OutputLaunchBinding
    ├── ParameterMappings[0..n] -> OutputParameterMapping
    ├── DeliveryChannels[0..n] -> OutputDeliveryChannel
    ├── FileOptions[0..1] -> OutputFileOptions
    ├── ExecutionOptions[0..1] -> OutputExecutionOptions
    └── SecurityOptions[0..1] -> OutputSecurityOptions
```

`FileOptions`, `ExecutionOptions` и `SecurityOptions` являются singleton-
коллекциями: в одном `Output` допускается не более одного узла каждого типа.
Остальные коллекции допускают несколько дочерних узлов и имеют код порядка,
если он предусмотрен schema.

### 4.2. Свойства корневого узла

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название выдачи | `LocalizedText` | Нет | Локализованные значения | Показывает `Output` в каталоге и интерфейсе запуска | Обычное локализуемое свойство |
| `Description` | Описание выдачи | `LocalizedText` | Нет | Локализованные значения | Поясняет назначение выдачи | Обычное локализуемое свойство |
| `SourceType` | Тип источника | `Enum` | Да; create-dialog default `Report` | `Report`, `Dataset`, `View`; статический каталог Configuration | Выбирает модель, из которой берётся результат | Задаётся при создании; определяет источник options для `SourceCode` |
| `SourceCode` | Код источника | `ReferenceCode` | Да | Для `Report` и `View` options формируются сервером из effective catalog и выдаются клиентскому editor; для `Dataset` текущий projector options не возвращает | Определяет конкретный `Report`, `View` или иной источник | Зависит от `SourceType`; текущий runtime исполняет только ссылку на `Report` |
| `Format` | Формат результата | `Enum` | Да; create-dialog default `Preview` | `Preview`, `Pdf`, `Excel`, `Csv`, `Print`; статический каталог Configuration | Выбирает формат, который запрашивается у рендерер | Задаётся при создании; фактическая поддержка определяется рендерер runtime |
| `DeliveryMode` | Режим возврата результата | `Enum` | Нет; create-dialog: `Inline` для `Preview`/`Print`, иначе `Download` | `Inline`, `Download`, `Background`, `External`; статический каталог Configuration | Определяет, как результат должен быть возвращён потребителю | Может редактироваться; runtime MVP принимает только `Inline` и `Download` |
| `ObjectTypeCode` | Код типа контекстного объекта | `ReferenceCode` | Нет; creation-only | `Catalog.ObjectTypes`; options из серверного каталога | Ограничивает контекст запуска типом объекта | Задаётся при создании и не меняется обычным editor |
| `IsDiscoverable` | Показывать в каталоге доступных выдач | `Bool` | Нет; builder задаёт только при явном вызове | `true` — доступен для обнаружения; `false` — не должен показываться как обнаруживаемый | Влияет на обнаружение артефакта в каталоге | Обычное свойство; отдельный runtime-фильтр не подтверждён |
| `CategoryCode` | Код категории выдачи | `ReferenceCode` | Нет | Источник options schema не задан | Группирует выдачу в каталоге, если потребитель поддерживает категории | Обычное свойство |
| `PermissionCode` | Код права запуска | `ReferenceCode` | Нет | Источник options schema не задан; код должен быть зарегистрирован в security manifest | Ограничивает запуск выдачи проверкой permission | Обычное свойство; runtime проверяет только при наличии кода |
| `FeatureProviderCode` | Код поставщика функции | `ReferenceCode` | Нет | Источник options schema не задан | Связывает выдачу с владельцем feature registration | Обычно задаётся baseline/provider и не определяет рендерер |

`Required` здесь означает обязательность сохранённого свойства в schema. Default
create-dialog или builder — это начальное значение authoring flow, а не общий
default effective-модели.

### 4.3. Вложенный тип `OutputLaunchBinding`

`OutputLaunchBinding` описывает контекст и отображаемые сведения места запуска.
Он не реализует кнопку и не задаёт общий frontend component.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ViewCode` | Код представления запуска | `ReferenceCode` | Нет | Код `View`; источник options schema не задан | Ограничивает binding конкретным представлением | Используется при materialization runtime View |
| `Placement` | Место размещения | `Enum` | Нет; runtime fallback `Toolbar` | `Toolbar`, `HeaderActions`, `ReportCenter`; статический каталог Configuration | Выбирает место, в котором потребитель может показать запуск | Интерпретируется frontend/runtime-потребителем |
| `ContextMode` | Контекст запуска | `Enum` | Да; runtime fallback `List` | `None`, `List`, `CurrentObject`, `Selection`, `ReportCenter`; статический каталог Configuration | Определяет, какие данные контекста могут быть переданы | Задаётся при создании binding |
| `SelectionMode` | Режим выбора объектов | `Enum` | Нет | `None`, `Optional`, `Single`, `Multiple`, `Required`; статический каталог Configuration | Определяет требования к выбранным строкам при запуске | Применяется для list/selection context |
| `Label` | Подпись запуска | `LocalizedText` | Нет; runtime использует название `Output`, если подпись отсутствует | Локализованные значения | Показывает текст команды запуска | Обычное локализуемое свойство |
| `Tooltip` | Подсказка запуска | `LocalizedText` | Нет | Локализованные значения | Показывает дополнительное пояснение | Обычное локализуемое свойство |
| `Icon` | Код значка | `String` | Нет | Код, который понимает рендерер фронтенда; каталог schema не задан | Передаёт визуальный идентификатор запуска | Интерпретируется frontend-потребителем |
| `Order` | Порядок запуска | `Number` | Нет; runtime fallback по позиции | Целое число порядка | Сортирует launch bindings | Используется при materialization runtime View |
| `VisibleWhen` | Условие видимости | `Json` | Нет | JSON, формат общего условия в schema не ограничен этим документом | Даёт потребителю условие показа запуска | Runtime evaluation не подтверждён текущей реализацией |
| `EnabledWhen` | Условие доступности | `Json` | Нет | JSON, формат общего условия в schema не ограничен этим документом | Даёт потребителю условие блокировки/доступности | Runtime evaluation не подтверждён текущей реализацией |

### 4.4. Вложенный тип `OutputParameterMapping`

`OutputParameterMapping` связывает параметр источника с данными контекста или
входом запуска. Текущая runtime-проекция использует только часть полей.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ParameterCode` | Код параметра источника | `ReferenceCode` | Да | Для `Report` options формируются сервером по `SourceType` + `SourceCode`; текущий editor возвращает параметры только для `Report` | Выбирает параметр, которому передаётся значение | Зависит от корневого источника |
| `Source` | Источник значения | `Enum` | Да | `CurrentObject`, `Selection`, `ViewFilter`, `ViewState`, `Constant`, `UserInput`, `Expression`; статический каталог Configuration | Выбирает контекст, откуда берётся значение | Определяет применимость `SourceCode` и `DefaultValue` |
| `SourceCode` | Код поля или ключа источника | `String` | Нет | Ключ поля/фильтра/входного параметра; отдельный options catalog не задан | Уточняет, какое значение читать из выбранного источника | Условно нужен для контекстных источников; runtime `Expression` и `ViewState` пока не поддерживает |
| `DefaultValue` | Значение по умолчанию | `String` | Нет | Строка в формате, который принимает параметр отчёта | Используется для `Constant` и как fallback в mapping/runtime-потребителе | Условно используется по типу `Source` |
| `Required` | Значение обязательно | `Bool` | Нет; runtime fallback `false` | `true` — отсутствие значения вызывает ошибку; `false` — mapping можно пропустить | Определяет, можно ли продолжить запуск без значения | Проверяется runtime mapping resolver |
| `Transform` | Преобразование значения | `String` | Нет | Формат преобразования schema не задан | Должно описывать преобразование перед передачей параметру | Сохраняется schema/baseline, но текущая runtime-проекция его не использует |

Static `Source` catalog и фактический runtime mapping пока расходятся: runtime
также знает исторические `ListFilter` и `Manual`, но не обрабатывает schema-
значения `Selection`, `ViewState` и `Expression`. Это зафиксировано в
[трассировке][traceability].

### 4.5. Вложенный тип `OutputDeliveryChannel`

`OutputDeliveryChannel` задаёт дополнительный канал выдачи в schema. Сам
Configuration не отправляет сообщение и не сохраняет файл в канал.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ChannelType` | Тип канала | `Enum` | Да | `Download`, `Inline`, `Email`, `AttachToObject`, `Storage`, `Print`; статический каталог Configuration | Выбирает способ выдачи результата | Требует реализации и контракта владельца канала |
| `RecipientSource` | Источник получателя | `Enum` | Нет | `User`, `Role`, `Email`, `Expression`, `CurrentObject`; статический каталог Configuration | Определяет, как найти получателя | Применяется для каналов, которым нужен получатель |
| `RecipientCode` | Код получателя | `String` | Нет | Код пользователя/роли/объекта или адрес в формате владельца канала | Уточняет получателя | Условно нужен по `RecipientSource` |
| `Subject` | Тема сообщения | `LocalizedText` | Нет | Локализованные значения | Задаёт тему сообщения для канала, например email | Условно применяется для message channel |
| `Body` | Тело сообщения | `LocalizedText` | Нет | Локализованные значения | Задаёт текст сообщения | Условно применяется для message channel |
| `Order` | Порядок канала | `Number` | Нет | Целое число порядка | Определяет порядок обработки/показа каналов | Schema поддерживает порядок; runtime Output MVP каналы пока не проектирует |

### 4.6. Вложенный тип `OutputFileOptions`

В одном `Output` допускается не более одного узла `FileOptions`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `FileNamePattern` | Шаблон имени файла | `String` | Нет | Строка имени; формат tokens не задан schema | Задаёт основу имени результата | Runtime применяет при построении имени файла |
| `Extension` | Расширение файла | `String` | Нет | Расширение без обязательной точки | Переопределяет расширение результата | Runtime применяет при построении имени файла |
| `MimeType` | MIME-тип файла | `String` | Нет | MIME-тип, например `application/pdf` | Описывает тип содержимого для потребителя | Runtime сейчас не использует при формировании response |
| `Encoding` | Кодировка файла | `String` | Нет | Кодировка в формате, принятом владельцем рендерер | Описывает кодировку результата | Текущая runtime-проекция `FileOptions` это поле не переносит |
| `IncludeTimestamp` | Добавлять время в имя | `Bool` | Нет; runtime fallback `false` | `true` — добавить UTC timestamp; `false` — не добавлять | Влияет на итоговое имя файла | Runtime применяет при построении имени файла |

### 4.7. Вложенный тип `OutputExecutionOptions`

В одном `Output` допускается не более одного узла `ExecutionOptions`. Эти
поля зарегистрированы schema, но текущий `GenerateOutput` не читает их как
отдельный runtime-контракт.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ExecutionMode` | Режим выполнения | `Enum` | Нет | `Sync`, `Async`, `Auto`; статический каталог Configuration | Задаёт желаемую синхронность выполнения | Будущая runtime-семантика; текущий endpoint отвечает синхронно |
| `TimeoutSeconds` | Лимит времени | `Number` | Нет | Число секунд; положительность отдельно schema не задана | Ограничивает длительность выполнения | Будущая runtime-семантика |
| `CachePolicy` | Политика кэширования | `Enum` | Нет | `None`, `PerUser`, `Shared`; статический каталог Configuration | Определяет область повторного использования результата | Будущая runtime-семантика |
| `MaxRows` | Максимальное число строк | `Number` | Нет | Число строк; диапазон отдельно schema не задан | Ограничивает объём результата | Будущая runtime-семантика |
| `AllowBackground` | Разрешить фоновой запуск | `Bool` | Нет; fallback не задан schema | `true` — фоновой запуск допустим; `false` — не допустим | Ограничивает выбор фонового режима | Текущая runtime-реализация не использует поле |

### 4.8. Вложенный тип `OutputSecurityOptions`

В одном `Output` допускается не более одного узла `SecurityOptions`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `RequireCurrentObjectAccess` | Требовать доступ к текущему объекту | `Bool` | Нет; fallback не задан schema | `true` — перед выдачей нужна проверка доступа к объекту; `false` — это правило не включено | Ограничивает выдачу в current-object context | Текущая runtime-служба не читает это поле; общая permission проверка использует `Output.PermissionCode` |
| `AuditMode` | Режим аудита выдачи | `Enum` | Нет | `None`, `Start`, `Complete`, `StartAndComplete`; статический каталог Configuration | Задаёт ожидаемые точки аудита операции | Реальный audit contract и обработчик принадлежат runtime/Audit History; текущая `GenerateOutput` не читает поле |

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

Статические enum-значения задаются кодом Configuration и доступны editor как
каталог вариантов. Русские подписи для них могут формироваться metadata/UI
слоем, но технические коды остаются неизменными.

| Группа | Технические значения |
| --- | --- |
| `Output.SourceType` | `Report`, `Dataset`, `View` |
| `Output.Format` | `Preview`, `Pdf`, `Excel`, `Csv`, `Print` |
| `Output.DeliveryMode` | `Inline`, `Download`, `Background`, `External` |
| `OutputLaunchBinding.ContextMode` | `None`, `List`, `CurrentObject`, `Selection`, `ReportCenter` |
| `OutputLaunchBinding.Placement` | `Toolbar`, `HeaderActions`, `ReportCenter` |
| `OutputLaunchBinding.SelectionMode` | `None`, `Optional`, `Single`, `Multiple`, `Required` |
| `OutputParameterMapping.Source` | `CurrentObject`, `Selection`, `ViewFilter`, `ViewState`, `Constant`, `UserInput`, `Expression` |
| `OutputDeliveryChannel.ChannelType` | `Download`, `Inline`, `Email`, `AttachToObject`, `Storage`, `Print` |
| `OutputDeliveryChannel.RecipientSource` | `User`, `Role`, `Email`, `Expression`, `CurrentObject` |
| `OutputExecutionOptions.ExecutionMode` | `Sync`, `Async`, `Auto` |
| `OutputExecutionOptions.CachePolicy` | `None`, `PerUser`, `Shared` |
| `OutputSecurityOptions.AuditMode` | `None`, `Start`, `Complete`, `StartAndComplete` |

Источник этих списков — [static catalog][static-catalog] и [static codes][static-codes].

### 5.2. Динамические значения

`SourceCode` и `ParameterCode` зависят от других свойств. Editor получает
options с сервера:

- для `SourceCode` при `SourceType = Report` выбираются зарегистрированные
  `Report`, при `SourceType = View` — зарегистрированные `View`;
- при `SourceType = Dataset` текущий editor не возвращает список options;
- для `ParameterCode` текущий editor получает параметры выбранного `Report`;
- `ObjectTypeCode` получает options из каталога типов объектов.

Это client-facing options, но источником истины остаются effective catalog и
серверные сервисы Configuration. Статические enum-каталоги также формируются
на сервере и передаются editor-контракту.

## 6. Ограничения и зависимости

- `SourceType` определяет, как интерпретировать `SourceCode`.
- В текущем `GenerateOutput` допустим только `SourceType = Report`; ссылки на
  `Dataset` и `View` сохраняются schema, но не дают исполняемого результата.
- Для runtime `Report` должен существовать в опубликованной effective
  конфигурации и содержать доступные design/schema content references.
- `ParameterCode` должен соответствовать параметру выбранного источника;
  schema сама не перечисляет параметры источника.
- `PermissionCode`, если задан, проверяется runtime перед выполнением.
- `LaunchBindings` материализуются в runtime View response только для
  подходящего `ViewCode`; общий frontend решает, как отобразить response.
- Singleton-коллекции не могут содержать более одного дочернего узла.
- `VisibleWhen`, `EnabledWhen`, `Transform`, `ExecutionOptions`,
  `SecurityOptions` и `DeliveryChannels` не становятся исполняемыми только из-за
  наличия в schema.
- Effective inheritance и scoped override применяются к узлам и свойствам по
  общим правилам Configuration; отдельная бизнес-семантика переопределения
  результата принадлежит runtime-владельцу.

## 7. Операции над структурой

Schema разрешает следующие операции над дочерними коллекциями:

| Коллекция | Допустимые операции | Ограничение |
| --- | --- | --- |
| `LaunchBindings` | `Create`, `DeleteSubtree`, `Reorder` | Упорядоченная коллекция |
| `ParameterMappings` | `Create`, `DeleteSubtree`, `Reorder` | Упорядоченная по операциям редактора коллекция |
| `DeliveryChannels` | `Update` | Текущая schema не разрешает create/delete через эту policy |
| `FileOptions` | `Create`, `Update`, `DeleteSubtree` | Не более одного узла |
| `ExecutionOptions` | `Update` | Не более одного узла |
| `SecurityOptions` | `Create`, `Update`, `DeleteSubtree` | Не более одного узла |

Операции root `Output` и сохранение draft выполняются общим Configuration
editor/lifecycle. Builder-ы [baseline builder][baseline-builder] и [node
builder][node-builder] создают структуру, но не заменяют validation и publication.

## 8. Наследование и рассчитанный результат

`Output` входит в effective configuration по общей цепочке scope/version.
Runtime получает только опубликованный effective результат через
[runtime provider][runtime-provider]. При merge singleton-узлов сохраняется
property-level patch semantics Configuration; фактическая интерпретация
полученной комбинации остаётся за runtime.

Для launch binding runtime-проекция использует `ViewCode`, placement, context,
selection, label, tooltip, icon, order и условия видимости/доступности. Для
генерации результата runtime-проекция использует root source/format/delivery,
permission, parameter mappings и file options.

## 9. Создание, проверка и публикация

При создании editor предлагает значения `SourceType`, `Format` и
`DeliveryMode` из допустимых каталогов и применяет начальные defaults, описанные
в таблице root properties. Builder требует root `OutputCode`, `SourceType`,
`SourceCode` и `Format`.

Generic `ArtifactValidator` проверяет наличие обязательных schema properties,
типы значений и структуру дочерних коллекций. Релевантные проверки подтверждены
в [schema tests][schema-tests] и [builder tests][builder-tests]. Editor options
и зависимые списки проверены в [editor tests][editor-tests].

После публикации runtime ищет только опубликованный effective `Output`,
проверяет permission, проверяет, что источник имеет тип `Report`, разрешает
опубликованный `Report`, передаёт параметры, вызывает рендерер отчётов и
возвращает [runtime response][runtime-response]. Ручной HTTP-контракт запуска
описан в [runtime request][runtime-request]; он не является Configuration
authoring-контрактом.

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `Output` и шесть дочерних типов зарегистрированы в schema registry;
- root properties, child collections, singleton limits и schema operations
  подтверждены кодом;
- static catalogs для enum-полей существуют;
- baseline и canonical builders создают root/child структуру;
- editor выдаёт зависимые options для `Report`/`View` sources и `Report`
  parameters;
- runtime разрешает опубликованный `Output`, выполняет связанный `Report` и
  возвращает content response для `Inline`/`Download`;
- launch bindings могут проектироваться в runtime View response;
- file name pattern, extension и timestamp участвуют в построении имени файла.

### За пределами текущего schema-контракта `Output`

- исполнение `SourceType = Dataset` или `SourceType = View`;
- runtime-поддержка `DeliveryMode = Background` или `External`;
- фактическая обработка `DeliveryChannels` для email, storage, attach и print;
- применение `ExecutionOptions` и `SecurityOptions.AuditMode`;
- применение `OutputParameterMapping.Transform`, `ViewState` и `Expression`;
- полноценная проверка `VisibleWhen` и `EnabledWhen`;
- кэширование, фоновые задания, расписание и хранение доставленных файлов;
- согласованный контракт доступа к текущему объекту по
  `RequireCurrentObjectAccess`.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Выдача результата | Output | Корневой конфигурационный артефакт формата, запуска и доставки результата |
| Источник результата | Source / `SourceType`, `SourceCode` | Тип и код отчёта, представления или другого источника |
| Место запуска | Launch binding / OutputLaunchBinding | Описание контекста и размещения запуска |
| Сопоставление параметра | Parameter mapping / OutputParameterMapping | Правило получения значения параметра источника |
| Канал выдачи | Delivery channel / OutputDeliveryChannel | Описание дополнительного способа передачи результата |
| Настройки файла | File options / OutputFileOptions | Правила имени, расширения и признака timestamp |
| Настройки выполнения | Execution options / OutputExecutionOptions | Желаемый режим выполнения, лимиты и cache policy |
| Настройки безопасности | Security options / OutputSecurityOptions | Требование object access и ожидаемый audit mode |
| Режим доставки | Delivery mode / `DeliveryMode` | Способ возврата результата потребителю |

## 12. Источники в коде и тестах

Основные подтверждения: [artifact codes][artifact-codes], [property codes][property-codes],
[collection codes][collection-codes], [schema][schema], [static catalog][static-catalog],
[static codes][static-codes], [baseline builder][baseline-builder], [node builder][node-builder],
[editor session][editor-session] и [editor projector][editor-projector].

Runtime-граница подтверждена через [runtime provider][runtime-provider],
[runtime materializer][runtime-materializer], [runtime service][runtime-service],
[runtime mapping][runtime-mapping], [runtime models][runtime-models] и
[runtime format mapper][runtime-format].

Релевантные тесты: [schema tests][schema-tests], [builder tests][builder-tests]
и [editor tests][editor-tests]. Расхождения между schema, editor и runtime
перечислены в [трассировке Configuration][traceability].

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
