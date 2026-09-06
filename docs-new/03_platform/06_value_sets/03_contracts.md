---
id: DOC-03-06-03
title: 'Контракты — Value Sets'
type: contract
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: value_sets
holder: '@axelprosoft'
created_at: 2026-08-26 18:00
created_by: '@codex'
updated_at: 2026-08-27 01:17
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Контракты — Value Sets

## 1. Назначение и границы

Документ описывает публичные HTTP-контракты, DTO и application-порты области
Value Sets. Он не дублирует схему `ValueSet`/`SystemEnum`, материализацию данных
для runtime или конкретную реализацию компонента фронтенда.

## 2. Источники истины и владельцы

| Контракт | Источник структуры | Владелец |
| --- | --- | --- |
| HTTP API Value Sets | Контроллер и типы contracts | Value Sets |
| Порт чтения runtime | `IValueSetsQueryService` | Value Sets |
| Адаптер runtime | `IRuntimeValueSetResolver` | Runtime / API host |
| Проекция опубликованной схемы набора | `IConfigurationPublishedValueSetProjectionWriter` | Граница Configuration |
| Начальные данные baseline | `IConfigurationBaselineValueSetDataSeedWriter` | Граница Configuration |

## 3. Карта контрактов

| Контракт | Вид | Владелец | Потребитель | Статус сведения | Подробное описание |
| --- | --- | --- | --- | --- | --- |
| `GET /api/platform/value-sets/items` | HTTP | Value Sets | Потребители runtime/API | Подтверждено в MVP | 4.1 |
| `GET /api/platform/value-sets/data/definitions` | HTTP | Value Sets | Studio и редактор данных | Подтверждено в MVP | 4.2 |
| `GET/POST/PUT /api/platform/value-sets/data/{valueSetCode}/items...` | HTTP | Value Sets | Studio и редактор данных | Подтверждено в MVP | 4.3 |
| `IValueSetsQueryService` | C#-порт | Value Sets | Runtime host | Подтверждено в MVP | 6.1 |
| `IValueSetDataEditorService` | C#-порт | Value Sets | API-контроллер | Подтверждено в MVP | 6.2 |
| Порты проекции и начальных данных `ValueSetDataSet` | C#-порт | Граница Configuration | Хранилище Value Sets | Подтверждено, но зависит от API host | 6.3 |

## 4. HTTP-контракты

### 4.1. Чтение эффективных элементов

Операция возвращает элементы набора, выбранные для текущего контекста с учётом
приоритета областей данных. `ValueSetCode` задаёт набор, а `IncludeInactive`
разрешает включить неактивные элементы.

| Метод и маршрут | Запрос | Ответ | Права | Идемпотентность | Конкурентность |
| --- | --- | --- | --- | --- | --- |
| `GET /api/platform/value-sets/items` | `ValueSetCode`, `IncludeInactive` | `ValueSetItemsResponse` | `Permission:Configuration.Scope.View` | Да | Не применимо |

Ответ содержит `ValueSetCode`, `Items`, `Structure` (`Flat`/`Hierarchical`) и
`RootBehavior` (`MultipleRoots`). Элемент содержит стабильный `Code`, заголовок,
признак активности, порядок, необязательные код и путь родителя, а также признак
переопределения на уровне Tenant. По умолчанию неактивные элементы не возвращаются.

### 4.2. Список наборов данных для редактора

`GET /api/platform/value-sets/data/definitions` принимает `Query`, `IncludeFixed`,
`Page`, `PageSize`, `Sort` и возвращает постраничный список
`ValueSetDataDefinitionResponse`.

Здесь «набор данных» означает хранимый `ValueSetDataSet` с фактическими
`ValueSetItem`. Метод API возвращает список таких наборов и сохранённые в них
метаданные исходного `ValueSet`: `ValueSetCode`, `Title`, `Structure`, `Policy`
и `AllowTenantOverrides`. Это не отдельный вид конфигурационного артефакта.
По умолчанию наборы с `Policy = Fixed` исключаются. Поля `ItemCount`, `CanEdit`
и `DisabledReason*` показывают количество элементов и доступность изменения в
текущем контексте.

### 4.3. Операции над элементами данных

| Операция | Маршрут | Права |
| --- | --- | --- |
| Эффективный или точный список | `GET /{valueSetCode}/items` | `ValueSets.Data.View`; для явно выбранной области также `.ManageScope` и назначенная область `Global`/`Corporate` |
| Кандидаты в родители | `GET /{valueSetCode}/items/parent-candidates` | `.View`, проверки изменения и области |
| Получить значения по умолчанию для новой строки | `GET /{valueSetCode}/items/create-defaults` | `.Edit` |
| Прочитать по идентификатору | `GET /{valueSetCode}/items/by-id/{itemId}` | `.View` |
| Создать строку | `POST /{valueSetCode}/items` | `.Edit` |
| Изменить по коду или идентификатору | `PUT /{valueSetCode}/items/{itemCode}` или `/by-id/{itemId}` | `.Edit` |
| Активировать или деактивировать | `POST .../activate` или `.../deactivate` | `.Edit` |

`CreateValueSetDataItemRequest` передаёт `Code`, `Title`, `IsActive`, `Order`,
необязательные `TenantId`/`SiteId` и `ParentItemId` или `ParentItemCode`. Запрос
обновления не меняет область существующей строки. `ParentItemId` и
`ParentItemCode`, если переданы вместе, должны указывать на один элемент.

## 5. Общие типы и DTO

| Тип | Назначение |
| --- | --- |
| `ValueSetDataSourceScope` | Область данных строки: `Global`, `Tenant`, `Site` |
| `ValueSetDataListMode` | Режим списка редактора: `EffectiveCurrent`, `ExactRows`, `EffectiveSelectedScope` |
| `ValueSetDataItemResponse` | Элемент, область данных, признаки включения в эффективный результат и переопределения, возможности изменения и отображаемые ссылки |
| `ValueSetOptionResponse` | Вариант для выбора во время выполнения: `Value`, `Label`, родитель и путь |

`ValueSetDataSetId` и `ValueSetItem.Id` могут присутствовать в ответе редактора,
но не являются стабильными бизнес-кодами.

### 5.1. Перечисления

| Тип | Допустимые значения | Смысл |
| --- | --- | --- |
| `ValueSetDataSourceScope` | `Global`, `Tenant`, `Site` | Область, к которой относятся физические строки данных |
| `ValueSetDataListMode` | `EffectiveCurrent`, `ExactRows`, `EffectiveSelectedScope` | Способ выбрать строки для ответа редактора |

### 5.2. `ValueSetItemsResponse` и `ValueSetItemResponse`

`ValueSetItemsResponse` возвращает эффективные элементы для чтения во время
выполнения. Структура подтверждена [контрактами Value Sets][contracts].

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ValueSetItemsResponse.ValueSetCode` | `string` | Нет | Обязательно | Непустой код набора | Код набора значений | Стабильный код, не GUID | [ValueSetItemsResponse][items-response] |
| `ValueSetItemsResponse.Items` | `IReadOnlyCollection<ValueSetItemResponse>` | Нет | Обязательно | Может быть пустой | Элементы ответа | Неактивные исключаются без `IncludeInactive` | [ValueSetItemsResponse][items-response] |
| `ValueSetItemsResponse.Structure` | `string` | Нет | `Flat` | `Flat`, `Hierarchical` | Структура набора | Значение приходит из схемы `ValueSet` | [ValueSetItemsResponse][items-response] |
| `ValueSetItemsResponse.RootBehavior` | `string` | Нет | `MultipleRoots` | `MultipleRoots` в текущем контракте | Правило корневых элементов | Другие значения кодом не подтверждены | [ValueSetItemsResponse][items-response] |
| `ValueSetItemResponse.Code` | `string` | Нет | Обязательно | Непустой код элемента | Стабильный код элемента | Уникален в эффективном наборе | [ValueSetItemResponse][item-response] |
| `ValueSetItemResponse.Title` | `string` | Нет | Обязательно | Любой непустой заголовок | Основной текст элемента | Возвращается как сохранённое значение | [ValueSetItemResponse][item-response] |
| `ValueSetItemResponse.Order` | `int?` | Да | Необязательно; `null` | Целое или `null` | Порядок сортировки | `null` сортируется после заданных значений | [ValueSetItemResponse][item-response] |
| `ValueSetItemResponse.IsActive` | `bool` | Нет | Обязательно | `true`/`false` | Доступен ли элемент для обычного чтения | `false` скрывается по умолчанию | [ValueSetItemResponse][item-response] |
| `ValueSetItemResponse.IsTenantOverride` | `bool` | Нет | Обязательно | `true`/`false` | Есть ли переопределение в области Tenant или Site | В текущем коде `true` для любой не-Global строки; имя поля задано существующим контрактом | [ValueSetItemResponse][item-response] |
| `ValueSetItemResponse.ParentItemCode` | `string?` | Да | `null` | Код родителя или `null` | Родитель элемента | Используется только для иерархического набора | [ValueSetItemResponse][item-response] |
| `ValueSetItemResponse.DisplayTitle` | `string?` | Да | `null` | Отображаемый заголовок | Подготовленное отображаемое значение | Не заменяет `Title` без правила потребителя | [ValueSetItemResponse][item-response] |
| `ValueSetItemResponse.DisplayPath` | `string?` | Да | `null` | Отображаемый путь или `null` | Полный путь в иерархии | Применим к иерархическому набору | [ValueSetItemResponse][item-response] |
| `ValueSetItemResponse.Level` | `int?` | Да | `null` | Уровень или `null` | Глубина в иерархии | Применим к иерархическому набору | [ValueSetItemResponse][item-response] |

### 5.3. `ValueSetOptionsResponse` и `ValueSetOptionResponse`

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ValueSetOptionsResponse.ValueSetCode` | `string` | Нет | Обязательно | Непустой код набора | Код набора значений | Стабильный код | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionsResponse.Items` | `IReadOnlyCollection<ValueSetOptionResponse>` | Нет | Обязательно | Может быть пустой | Варианты выбора | Формируются с учётом фильтра и области данных | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionsResponse.TotalCount` | `int` | Нет | Обязательно | `0` или больше | Число элементов результата | Относится к применённым фильтрам | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionsResponse.Page` | `int` | Нет | Обязательно | Положительное число | Номер страницы | Нормализуется сервисом | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionsResponse.PageSize` | `int` | Нет | Обязательно | Положительное число | Размер страницы | Нормализуется сервисом | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionsResponse.Structure` | `string` | Нет | `Flat` | `Flat`, `Hierarchical` | Структура набора | Значение из схемы `ValueSet` | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionsResponse.RootBehavior` | `string` | Нет | `MultipleRoots` | `MultipleRoots` в текущем контракте | Правило корневых элементов | Другие значения не подтверждены | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.Value` | `string` | Нет | Обязательно | Значение элемента | Значение, передаваемое потребителю | Не является внутренним GUID | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.Label` | `string` | Нет | Обязательно | Отображаемая строка | Текст варианта выбора | Формируется для потребителя | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.Code` | `string` | Нет | Обязательно | Код элемента | Стабильный код элемента | Не меняется при смене области данных | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.Title` | `string` | Нет | Обязательно | Заголовок элемента | Исходный заголовок | Хранится у элемента | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.Subtitle` | `string?` | Да | `null` | Дополнительный текст или `null` | Дополнительная подпись | Не обязательна | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.ParentValue` | `string?` | Да | `null` | Значение родителя или `null` | Родитель варианта | Для иерархических наборов | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.Path` | `string?` | Да | `null` | Путь или `null` | Путь в иерархии | Для иерархических наборов | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.Level` | `int?` | Да | `null` | Уровень или `null` | Глубина в иерархии | Для иерархических наборов | [ValueSetOptionsResponse][options-response] |
| `ValueSetOptionResponse.HasChildren` | `bool` | Нет | `false` | `true`/`false` | Есть ли дочерние элементы | Для иерархических наборов | [ValueSetOptionsResponse][options-response] |

### 5.4. Ответы редактора данных

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ValueSetDataDefinitionsResponse.Items` | `IReadOnlyCollection<ValueSetDataDefinitionResponse>` | Нет | Обязательно | Может быть пустой | Страница наборов данных, доступных редактору | `Fixed` исключается по умолчанию | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionsResponse.TotalCount` | `int` | Нет | Обязательно | `0` или больше | Общее число найденных наборов данных | С учётом фильтра | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionsResponse.Page` | `int` | Нет | Обязательно | Положительное число | Номер страницы | Нормализуется сервисом | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionsResponse.PageSize` | `int` | Нет | Обязательно | Положительное число | Размер страницы | Нормализуется сервисом | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.ValueSetCode` | `string` | Нет | Обязательно | Непустой код | Код набора значений | Связывает Configuration и данные | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.Title` | `string` | Нет | Обязательно | Заголовок | Снимок заголовка набора значений | Снимок, а не схема | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.Structure` | `string` | Нет | Обязательно | `Flat`, `Hierarchical` | Структура набора | Из сохранённого снимка схемы `ValueSet` | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.Policy` | `string` | Нет | Обязательно | `Configurable`, `Overrideable`, `Fixed` по текущей модели | Политика изменения данных | По текущему коду отдельное различие первых двух политик не раскрыто | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.AllowTenantOverrides` | `bool` | Нет | Обязательно | `true`/`false` | Разрешены ли tenant-переопределения | Проверяется при изменении данных | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.ItemCount` | `int` | Нет | Обязательно | `0` или больше | Число элементов | Значение ответа редактора | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.CanEdit` | `bool` | Нет | Обязательно | `true`/`false` | Можно ли изменить набор в текущем контексте | Зависит от политики и прав | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.DisabledReasonCode` | `string?` | Да | `null` при `CanEdit=true` | Код причины или `null` | Машинная причина запрета | Должен быть согласован с кодами ошибок | [ValueSetDataDefinitionsResponse][definitions-response] |
| `ValueSetDataDefinitionResponse.DisabledReasonMessage` | `string?` | Да | `null` при `CanEdit=true` | Текст причины или `null` | Человекочитаемое объяснение запрета | Не используется как стабильный код | [ValueSetDataDefinitionsResponse][definitions-response] |

### 5.5. Элемент данных редактора

| Путь или поле | Тип | Nullable | Обязательность/default | Допустимые значения | Смысл | Ограничения | Источник |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `ValueSetDataItemsResponse.Items` | `IReadOnlyCollection<ValueSetDataItemResponse>` | Нет | Обязательно | Может быть пустой | Страница физических или эффективных строк | Режим задаётся запросом | [ValueSetDataItemsResponse][data-items-response] |
| `ValueSetDataItemsResponse.TotalCount` | `int` | Нет | Обязательно | `0` или больше | Число строк результата | С учётом фильтров | [ValueSetDataItemsResponse][data-items-response] |
| `ValueSetDataItemsResponse.Page` | `int` | Нет | Обязательно | Положительное число | Номер страницы | Нормализуется сервисом | [ValueSetDataItemsResponse][data-items-response] |
| `ValueSetDataItemsResponse.PageSize` | `int` | Нет | Обязательно | Положительное число | Размер страницы | Нормализуется сервисом | [ValueSetDataItemsResponse][data-items-response] |
| `ValueSetDataItemsResponse.EffectiveOnly` | `bool` | Нет | `true` | `true`/`false` | Признак эффективного режима ответа | Не заменяет `Mode` запроса | [ValueSetDataItemsResponse][data-items-response] |
| `ValueSetDataItemResponse.Id` | `Guid?` | Да | `null` для значений по умолчанию или результата без физической строки | GUID или `null` | Внутренний идентификатор физической строки | Не использовать как бизнес-код | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.Code` | `string` | Нет | Обязательно | Непустой код | Стабильная идентичность элемента | Уникален внутри набора данных и области | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.Title` | `string` | Нет | Обязательно | Непустой заголовок | Значение для отображения | Сохраняется в строке | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.IsActive` | `bool` | Нет | Обязательно | `true`/`false` | Активен ли элемент | Неактивный элемент скрывается по умолчанию | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.Order` | `int?` | Да | `null` | Целое или `null` | Порядок сортировки | `null` после заданных значений | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.TenantId` | `Guid?` | Да | `null` для Global | GUID Tenant или `null` | Tenant, которому принадлежит строка | Site требует Tenant | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.SiteId` | `Guid?` | Да | `null` для Global/Tenant | GUID Site или `null` | Site, которому принадлежит строка | Нельзя без `TenantId` | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.ParentItemId` | `Guid?` | Да | `null` для корня | GUID или `null` | Физический родитель | Только для `Structure = Hierarchical` | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.ParentItemCode` | `string?` | Да | `null` для корня | Код родителя или `null` | Код родителя для чтения | Сервер проверяет соответствие ID и кода | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.HasChildren` | `bool` | Нет | Обязательно | `true`/`false` | Есть ли дочерние элементы | Для `Structure = Hierarchical` | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.DisplayValues` | `IReadOnlyDictionary<string, ValueSetDataReferenceDisplayResponse>?` | Да | `null` | Отображаемые значения по ключу ссылки | Подготовленные подписи ссылочных полей | Формируются механизмом разрешения ссылок | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.LookupDisplay` | `ValueSetDataReferenceDisplayResponse?` | Да | `null` | Отображаемое значение или `null` | Подпись элемента для lookup | Не заменяет `Code` | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.SourceScope` | `ValueSetDataSourceScope` | Нет | Обязательно | `Global`, `Tenant`, `Site` | Область физической или эффективной строки | Не путать с областью безопасности | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.IsEffective` | `bool` | Нет | Обязательно | `true`/`false` | Вошла ли строка в эффективный результат | Зависит от режима запроса | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.IsOverride` | `bool` | Нет | Обязательно | `true`/`false` | Является ли строка переопределением области | Та же бизнес-идентичность, более узкая область | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.CanEdit` | `bool` | Нет | Обязательно | `true`/`false` | Можно ли изменить строку | Зависит от прав, политики и области | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.CanCreateOverride` | `bool` | Нет | Обязательно | `true`/`false` | Можно ли создать переопределение | Не означает возможность изменить унаследованную строку | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.DisabledReasonCode` | `string?` | Да | `null` при доступном действии | Код причины или `null` | Машинная причина блокировки | Не заменяет код ошибки HTTP/result | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataItemResponse.DisabledReasonMessage` | `string?` | Да | `null` при доступном действии | Текст причины или `null` | Объяснение блокировки | Не является стабильным ключом | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataReferenceDisplayResponse.Id` | `string` | Нет | Обязательно | Идентификатор ссылки | Идентификатор отображаемой сущности | Не объявляется бизнес-кодом без отдельного контракта | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataReferenceDisplayResponse.Code` | `string?` | Да | `null` | Код или `null` | Дополнительный код ссылки | Не у всех ссылок есть code | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataReferenceDisplayResponse.Title` | `string` | Нет | Обязательно | Заголовок | Читаемое имя ссылки | Для UI | [ValueSetDataItemResponse][data-item-response] |
| `ValueSetDataReferenceDisplayResponse.Subtitle` | `string?` | Да | `null` | Дополнительный текст или `null` | Дополнительное описание ссылки | Не обязательна | [ValueSetDataItemResponse][data-item-response] |

### 5.6. Запросы редактора данных

В запросах ниже `Page` и `PageSize` имеют положительные значения и нормализуются
сервисом. `TenantId` и `SiteId` задают область данных, а не область безопасности.

| Запрос | Поля | Правила |
| --- | --- | --- |
| `GetValueSetDataDefinitionsRequest` | `Query: string?`, `IncludeFixed: bool = false`, `Page: int = 1`, `PageSize: int = 50`, `Sort: string?` | `IncludeFixed=false` исключает `Fixed`; пустой `Query` не фильтрует |
| `GetValueSetDataItemsRequest` | `IncludeInactive: bool = false`, `Query: string?`, `Page: int = 1`, `PageSize: int = 50`, `Sort: string?`, `Scope: ValueSetDataSourceScope?`, `Mode: ValueSetDataListMode = EffectiveCurrent`, `TenantId: Guid?`, `SiteId: Guid?`, `ParentNodeId: string?` | Явно выбранная область требует `ExactRows` или `EffectiveSelectedScope`; `SiteId` требует `TenantId` |
| `GetValueSetDataParentCandidatesRequest` | `IncludeInactive: bool = false`, `Query: string?`, `Page: int = 1`, `PageSize: int = 50`, `Sort: string?`, `TenantId: Guid?`, `SiteId: Guid?`, `CurrentItemId: Guid?`, `ParentNodeId: string?` | Только для `Structure = Hierarchical`; текущий элемент и его потомки исключаются |
| `GetValueSetItemsRequest` | `ValueSetCode: string`, `IncludeInactive: bool = false` | `ValueSetCode` обязателен; неактивные элементы включаются только явно |
| `GetValueSetOptionsRequest` | `ValueSetCode: string`, `Query: string?`, `ParentValue: string?`, `IncludeInactive: bool = false`, `Page: int = 1`, `PageSize: int = 50`, `SelectedValues: IReadOnlyCollection<string>?` | `ValueSetCode` обязателен; `ParentValue` применяется к иерархическому набору |
| `CreateValueSetDataItemRequest` | `Code: string`, `Title: string`, `IsActive: bool = true`, `Order: int?`, `TenantId: Guid?`, `SiteId: Guid?`, `ParentItemId: Guid?`, `ParentItemCode: string?` | `Code` и `Title` обязательны; область и родитель проходят серверную проверку |
| `UpdateValueSetDataItemRequest` | `Title: string`, `IsActive: bool`, `Order: int?`, `TenantId: Guid?`, `SiteId: Guid?`, `ParentItemId: Guid?`, `ParentItemCode: string?` | `Title` и `IsActive` обязательны; изменение области существующей строки запрещено |

## 6. C#-контракты и точки расширения

### 6.1. `IValueSetsQueryService`

| Метод | Результат | Предусловия | Побочные эффекты |
| --- | --- | --- | --- |
| `GetItemsAsync(code, includeInactive)` | `ValueSetItemsResponse` | Непустой `ValueSetCode` | Нет |
| `GetOptionsAsync(request)` | `ValueSetOptionsResponse` | Непустой `ValueSetCode`, корректные параметры постраничной выдачи | Нет |
| `ExistsAsync(code)` | `bool` | Непустой `ValueSetCode` | Нет |

### 6.2. `IValueSetDataEditorService`

Это [`IValueSetDataEditorService`][editor-port] — application-порт между
API-контроллером и сервисом редактора данных. Он
описывает операции области, но не раскрывает внутренние запросы к базе данных.

| Метод | Результат | Предусловия | Побочные эффекты |
| --- | --- | --- | --- |
| `GetDefinitionsAsync(request)` | `ValueSetDataDefinitionsResponse` | Валидные параметры страницы | Нет |
| `GetItemsAsync(valueSetCode, request)` | `ValueSetDataItemsResponse` | Непустой `ValueSetCode`; допустимые режим и область | Нет |
| `GetParentCandidatesAsync(valueSetCode, request)` | `ValueSetDataItemsResponse` | `Structure = Hierarchical`; корректная область | Нет |
| `GetItemCreateDefaultsAsync(valueSetCode)` | `ValueSetDataItemResponse` | Непустой `ValueSetCode` | Нет |
| `GetItemByIdAsync(valueSetCode, itemId)` | `ValueSetDataItemResponse` | Непустые `ValueSetCode` и `itemId` | Нет |
| `CreateItemAsync(valueSetCode, request)` | `ValueSetDataItemResponse` | Права, политика, область и родитель проходят проверку | Создание строки |
| `UpdateItemAsync(valueSetCode, itemCode, request)` | `ValueSetDataItemResponse` | Строка доступна для изменения; область не меняется | Изменение строки |
| `UpdateItemByIdAsync(valueSetCode, itemId, request)` | `ValueSetDataItemResponse` | Непустой `itemId`; строка доступна для изменения | Изменение строки |
| `SetItemActiveStateAsync(valueSetCode, itemCode, isActive)` | `ValueSetDataItemResponse` | Права и политика разрешают изменение | Изменение `IsActive` |
| `SetItemActiveStateByIdAsync(valueSetCode, itemId, isActive)` | `ValueSetDataItemResponse` | Непустой `itemId`; права и политика разрешают изменение | Изменение `IsActive` |

### 6.3. Проекция опубликованной схемы набора и начальные данные

Configuration передаёт опубликованные метаданные и начальные элементы через
`IConfigurationPublishedValueSetProjectionWriter` и
`IConfigurationBaselineValueSetDataSeedWriter`. Адаптер приложения записывает
их в хранилище Value Sets; конкретная транзакционная композиция определяется
приложением, в котором работает API.

## 7. Контракты событий

Публичное событие Value Sets в текущем коде не подтверждено. Инвалидация кэша и
уведомление потребителей не объявляются контрактом событий этого документа.

## 8. Ошибки и отказоустойчивость

| Код ошибки | Условие |
| --- | --- |
| `VALUE_SET_SELECTED_SCOPE_MODE_REQUIRED` | Выбрана область Tenant/Site без режима `ExactRows` или `EffectiveSelectedScope` |
| `VALUE_SET_MANAGE_SCOPE_REQUIRED` | Для выбранной области нужен `ValueSets.Data.ManageScope` |
| `VALUE_SET_SCOPE_ACCESS_FORBIDDEN` | Нет назначения `Global`/`Corporate` для доступа к выбранной области |
| `VALUE_SET_INVALID_SCOPE` | Site задан без Tenant |
| `VALUE_SET_PARENT_NODE_NOT_ALLOWED` | `ParentNodeId` задан для плоского набора |
| `VALUE_SET_DATASET_NOT_FOUND` | Набор данных отсутствует или недоступен редактору |
| `VALUE_SET_ITEM_ID_REQUIRED` | Для операции по идентификатору не передан `itemId` |
| `VALUE_SET_ITEM_NOT_FOUND` | Элемент не найден или не входит в эффективный результат для текущего контекста |
| `VALUE_SET_FOREIGN_SCOPE_FORBIDDEN` | Операция выходит за разрешённую область |
| `VALUE_SET_UNKNOWN_TENANT_SCOPE` | Выбранный tenant не найден |
| `VALUE_SET_UNKNOWN_SITE_SCOPE` | Выбранный site не найден в tenant |
| `VALUE_SET_PARENT_NOT_ALLOWED` | Родитель задан для плоского набора |
| `VALUE_SET_PARENT_NOT_FOUND` | Родитель не найден в выбранном наборе или области |
| `VALUE_SET_PARENT_MISMATCH` | `ParentItemId` и `ParentItemCode` указывают на разные элементы |
| `VALUE_SET_PARENT_SCOPE_INCOMPATIBLE` | Родитель недоступен в выбранной области данных |
| `VALUE_SET_PARENT_CYCLE` | Родитель создаёт цикл |
| `VALUE_SET_ITEM_DUPLICATE_SCOPE_CODE` | Код уже есть в той же области |
| `VALUE_SET_POLICY_FIXED` | Набор имеет политику `Fixed` и не редактируется |
| `VALUE_SET_TENANT_OVERRIDE_FORBIDDEN` | Набор не разрешает переопределения на уровне Tenant |
| `VALUE_SET_INVALID_REQUEST` | Обязательное поле запроса отсутствует или пусто |
| `VALUE_SET_EDIT_FORBIDDEN` | Редактирование недоступно в текущем контексте |
| `VALUE_SET_SCOPE_IDENTITY_CHANGE_NOT_SUPPORTED` | Нельзя изменить область существующей строки |
| `VALUE_SET_OVERRIDE_REQUIRED` | Для изменения унаследованной строки сначала нужно создать переопределение области (override) |
| `VALUE_SET_INHERITED_ROW_MUTATION_REJECTED` | Попытка изменить унаследованную строку без переопределения (override) |

## 9. Совместимость и изменение контрактов

Стабильные коды и значения enum DTO являются контрактом. Внутренние GUID,
таблицы хранения и реализация repository не являются внешним контрактом. Изменение
режима области, структуры ответа или кода ошибки требует проверки потребителей.

## 10. Границы с другими владельцами

Схема набора значений принадлежит Configuration; общие контексты и права —
Foundation и Tenant/Security; адаптер runtime resolver — Runtime/API host.
Value Sets описывает только собственные API данных и их семантику.

## 11. Источники и тесты

Структура подтверждена [контрактами Value Sets][contracts],
[ValueSetsController][values-controller] и
[ValueSetDataController][data-controller].

[contracts]: ../../../src/Platform/DMP.Platform.Contracts/ValueSets/
[items-response]: ../../../src/Platform/DMP.Platform.Contracts/ValueSets/Responses/ValueSetItemsResponse.cs
[item-response]: ../../../src/Platform/DMP.Platform.Contracts/ValueSets/Responses/ValueSetItemResponse.cs
[options-response]: ../../../src/Platform/DMP.Platform.Contracts/ValueSets/Responses/ValueSetOptionsResponse.cs
[definitions-response]: ../../../src/Platform/DMP.Platform.Contracts/ValueSets/Responses/ValueSetDataDefinitionsResponse.cs
[data-items-response]: ../../../src/Platform/DMP.Platform.Contracts/ValueSets/Responses/ValueSetDataItemsResponse.cs
[data-item-response]: ../../../src/Platform/DMP.Platform.Contracts/ValueSets/Responses/ValueSetDataItemResponse.cs
[values-controller]: ../../../src/Platform/DMP.Platform.ValueSets/Api/Controllers/ValueSetsController.cs
[data-controller]: ../../../src/Platform/DMP.Platform.ValueSets/Api/Controllers/ValueSetDataController.cs
[editor-port]: ../../../src/Platform/DMP.Platform.ValueSets/Application/Abstractions/IValueSetDataEditorService.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 01:17 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | модули ядра 05, 06 подготовлены для передачи на ревью. изменения в связанных документах. | [6f7684b3](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/6f7684b3dff526120cfead5984de36e0427a18d1) |
