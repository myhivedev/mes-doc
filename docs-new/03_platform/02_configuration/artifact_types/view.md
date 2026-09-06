---
id: DOC-03-02-AT-VIEW
title: 'Тип конфигурационного артефакта — View'
type: module-spec
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: configuration
holder: '@axelprosoft'
created_at: 2026-08-25 17:20
created_by: '@axelprosoft'
updated_at: 2026-09-03 16:08
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Тип конфигурационного артефакта — View

[schemas]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ViewArtifactSchemas.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ViewArtifactSchemas.cs
[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[template-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ViewTemplateCatalog.cs
[node-builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/ViewNodeBuilder.cs
[configuration-terms]: ../../../11_glossary/configuration_terms.md
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactSchemaRegistryIntegrationTests.cs
[builder-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ViewNodeBuilderIntegrationTests.cs
[editor-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactEditorSectionsIntegrationTests.cs

## 1. Назначение

`View` — корневой конфигурационный артефакт представления. Он описывает
структуру и настройки экрана, списка, карточки, панели показателей или рабочей
области.

В документе используется термин **представление** (`View`). Конкретный результат
может быть экраном, списком, карточкой, панелью или рабочей областью.

`View` не является исполняемой моделью бизнес-объекта. Он не объявляет поля, не
хранит данные, не выполняет действия и не заменяет Object Runtime.

## 2. Идентичность и граница

### 2.1. Идентичность

`View` идентифицируется значением `ArtifactNode.Code`. Значение
`ArtifactNode.ArtifactTypeCode` равно `View`.

`ViewCode` — прикладное название стабильного кода представления. Отдельного
сохраняемого свойства `ViewCode` в schema нет.

### 2.2. Входит в документ

- корневой узел `View` и его свойства;
- выбранный `ViewType` и допустимый для него `TemplateCode`;
- связь с `ObjectTypeCode` и `DatasetCode`;
- дочерние коллекции и их вложенные типы;
- правила связей, применимости и проверки перед публикацией.

### 2.3. Не входит в документ

- бизнес-модель `ObjectType` и свойства её полей;
- Object Runtime, выполнение запросов, repository и конвейер изменения данных;
- обработчики `Action`, переходы `Workflow` и выполнение `Rule`;
- общий frontend shell, routing, session/bootstrap, component library и границы
  приложений;
- реализация рендерер для `TemplateCode`;
- окончательный владелец `Dataset` и `Read Query Capability`.

`View` может ссылаться на эти сущности стабильными кодами. Ссылка не делает их
вложенными узлами `View`.

## 3. Место в Configuration

Реестр схем регистрирует `View` с признаками `Node`, `ObjectBound`,
`RuntimeRenderable`, `ActionHost` и `HasConditions`. `ViewType` является
дискриминатором корневого узла. ([schema][schemas])

`ViewNodeBuilder` и baseline authoring формируют канонический узел `View` и его
вложенные узлы. ([builder][node-builder])

Configuration владеет сохранением, проверкой, публикацией и серверным
контрактом редактора. фронтенд-платформа владеет общей отрисовкой, shell и
приложениями. Конкретное frontend-потребление описывается в
`06_user_experience.md`.

## 4. Структура и схема `View`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств. Индивидуальные свойства приведены в
разделах `4.2–4.18`.

```text
View (ArtifactNode)
├── identity: ArtifactTypeCode = View; Code = <ViewCode>
├── properties: { common properties, ViewType, ... }
└── child collections allowed by ViewType:
    ├── ObjectForm:
    │   ├── FormElements[0..n] → ViewFormElement
    │   ├── Layout[0..n] → ViewLayoutNode
    │   │   (hierarchical; ParentLayoutCode → ViewLayoutNode.Code)
    │   ├── ActionMenuSections[0..n] → ViewActionMenuSection
    │   ├── ActionPlacements[0..n] → ViewActionPlacement
    │   └── LocalBehaviors[0..n] → ViewLocalBehavior
    ├── ObjectList / LookupListView:
    │   ├── Columns[0..n] → ViewColumn
    │   ├── QueryParameters[0..n] → ViewQueryParameter
    │   ├── Filters[0..n] → ViewFilter
    │   ├── FilterPresets[0..n] → ViewFilterPreset
    │   │   └── Values[0..n] → ViewFilterPresetValue
    │   ├── ActionMenuSections[0..n] → ViewActionMenuSection
    │   └── ActionPlacements[0..n] → ViewActionPlacement
    ├── Dashboard:
    │   ├── Widgets[0..n] → ViewWidget
    │   ├── Layout[0..n] → ViewLayoutNode (hierarchical)
    │   ├── ActionMenuSections[0..n] → ViewActionMenuSection
    │   └── ActionPlacements[0..n] → ViewActionPlacement
    └── Workspace:
        ├── Layout[0..n] → ViewLayoutNode (hierarchical)
        ├── ActionMenuSections[0..n] → ViewActionMenuSection
        └── ActionPlacements[0..n] → ViewActionPlacement
```

`ViewType` выбирается один раз и определяет допустимую ветку структуры. Нельзя
считать все перечисленные коллекции доступными одновременно. Иерархия `Layout`
относится к конфигурационным узлам компоновки: родитель задаётся свойством
`ParentLayoutCode`, а не физическим вложением узлов. Отдельно `TreeMode` и
свойства `Tree*` задают отображение иерархии **данных объектов** в списке;
они не добавляют дочернюю коллекцию в schema `View`.

| `ViewType` | Русский смысл | Разрешённые коллекции |
| --- | --- | --- |
| `ObjectForm` | Карточка объекта | `FormElements`, `Layout`, `ActionMenuSections`, `ActionPlacements`, `LocalBehaviors` |
| `ObjectList` | Список объектов | `Columns`, `QueryParameters`, `Filters`, `FilterPresets`, `ActionMenuSections`, `ActionPlacements` |
| `LookupListView` | Список выбора значения | `Columns`, `QueryParameters`, `Filters`, `FilterPresets`, `ActionMenuSections`, `ActionPlacements` |
| `Dashboard` | Панель показателей | `Widgets`, `Layout`, `ActionMenuSections`, `ActionPlacements` |
| `Workspace` | Рабочая область | `Layout`, `ActionMenuSections`, `ActionPlacements` |

### 4.2. Свойства корневого узла

В этой таблице указаны общие свойства корневого `View`. Пометка «статический
каталог вариантов» означает, что допустимые коды заданы в [каталоге статических
вариантов][static-options]. Это не список, который может произвольно расширить
клиентский редактор.

Ниже находятся **таблицы свойств артефакта**. В каждой таблице одна строка
описывает одно свойство конкретного узла схемы. Для всех таблиц используются
одинаковые столбцы:

- `Технический код` — имя свойства в schema и коде;
- `Русский смысл` — понятное русское название свойства;
- `Тип значения` — формат значения, например `Bool`, `Enum`, `String` или
  `ReferenceCode`;
- `Обязательность / значение по умолчанию` — можно ли не задавать свойство и
  какое значение применяется при отсутствии значения;
- `Допустимые значения или источник` — допустимые коды, формат либо каталог,
  из которого выбирается ссылка или значение;
- `Смысл значения и влияние` — что означает значение и на что оно влияет;
- `Когда используется / переопределение` — условие применения и правила
  наследования.

В последнем столбце используются следующие обозначения:

- `Default` — для свойства нет специального ограничения наследования;
- `InheritedOnly` — значение берётся от базового типа и не изменяется в
  производном типе;
- `CreationOnly` — значение задаётся только при создании;
- `OverrideAllowed` — производный тип может заменить унаследованное значение;
- `Schema filter` — сервер предлагает только значения из каталога, прошедшие
  условие, например `View` определённого типа;
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
| `ViewType` | Тип представления | `Enum` | Да | `ObjectForm`, `ObjectList`, `LookupListView`, `Dashboard`, `Workspace`; статический каталог вариантов | Выбирает назначение представления и допустимую структуру дочерних узлов | Дискриминатор; после создания не изменяется |
| `TemplateCode` | Код шаблона представления | `ReferenceCode` | Нет | `Registry.ViewTemplates`; сервер фильтрует по `ViewType` | Выбирает зарегистрированный визуальный каркас; рендерер принадлежит фронтенд-платформа | Все типы; значение проверяется относительно `ViewType` |
| `Title` | Заголовок представления | `LocalizedText` | Нет | Значение автора | Заголовок для редактора и пользователя; schema-структуру не меняет | Все типы |
| `Description` | Описание представления | `LocalizedText` | Нет | Значение автора | Поясняет назначение представления; выполнение не меняет | Все типы |
| `ObjectTypeCode` | Код типа объекта | `ReferenceCode` | Нет | Каталог `ObjectTypes` | Задаёт объектный контекст и доступные поля | Для object-bound представлений; `CreationOnly` |
| `DatasetCode` | Код набора данных | `ReferenceCode` | Нет | Ссылка на `Dataset`; каталог и владелец не заданы schema `View` | Указывает набор данных для чтения; формат и выполнение задаёт владелец Dataset | Если представление использует Dataset; `InheritedOnly` |

Для `TemplateCode` в текущем backend-реестре зарегистрированы
`ObjectFormTemplate`, `ListTemplate`, `TreeListTemplate`, `LookupListTemplate`,
`DashboardTemplate`, `WorkspaceTemplate`, `SplitExplorerTemplate` и
`TreeInspectorTemplate`. `FormTemplate` текущим backend-реестром не
зарегистрирован. ([catalog][template-catalog])

### 4.3. Свойства корневого `View`, зависящие от `ViewType`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `FilterMode` | Режим фильтрации | `Enum` | Нет | `Inline`, `Panel`, `Drawer`, `AdvancedOnly`, `PresetAndUser`; статический каталог вариантов | `Inline` — фильтр в списке; `Panel` — панель; `Drawer` — выдвижная область; `AdvancedOnly` — расширенный режим; `PresetAndUser` — преднастройки и пользовательская фильтрация | `ObjectList`, `LookupListView` |
| `AdministrationSectionMode` | Режим секции администрирования | `Enum` | Нет | `Inherit`, `Auto`, `Hidden`, `Explicit`; статический каталог вариантов | `Inherit` — унаследованный режим; `Auto` — автоматический выбор; `Hidden` — скрыть; `Explicit` — явная конфигурация | Если представление имеет секцию администрирования |
| `DefaultPresetCode` | Преднастройка фильтра по умолчанию | `ReferenceCode` | Нет | `View.FilterPresets` текущего `View` | Выбирает преднастройку, применяемую по умолчанию | `ObjectList`, `LookupListView`; только при наличии `FilterPresets` |
| `TreeMode` | Режим отображения дерева | `Enum` | Нет | `Auto`, `Flat`, `Tree`, `TreeGrid`; статический каталог вариантов | `Auto` — автоматический выбор; `Flat` — плоский список; `Tree` — дерево; `TreeGrid` — дерево с колонками | `ObjectList`, `LookupListView` |
| `TreeNodeIdFieldCode` | Поле идентификатора узла | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле идентификатора узла дерева | `ObjectList`, `LookupListView`; при использовании дерева |
| `TreeParentFieldCode` | Поле родителя узла | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле непосредственного родителя | `ObjectList`, `LookupListView`; при использовании дерева |
| `TreeTitleFieldCode` | Поле заголовка узла | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле заголовка узла | `ObjectList`, `LookupListView`; при использовании дерева |
| `TreeHasChildrenFieldCode` | Поле наличия дочерних узлов | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле, по которому определяется наличие дочерних узлов | `ObjectList`, `LookupListView`; при использовании дерева |
| `TreeDefaultExpandMode` | Начальное состояние дерева | `Enum` | Нет | `Expanded`, `Collapsed`; статический каталог вариантов | `Expanded` — узлы раскрыты; `Collapsed` — узлы свёрнуты | `ObjectList`, `LookupListView`; при использовании дерева |

### 4.4. Вариант `ObjectForm`

```text
ViewType = ObjectForm
├── FormElements[] → ViewFormElement
├── Layout[] → ViewLayoutNode
├── ActionMenuSections[] → ViewActionMenuSection
├── ActionPlacements[] → ViewActionPlacement
└── LocalBehaviors[] → ViewLocalBehavior
```

`ObjectForm` предназначен для карточки объекта. Свойства элементов формы
описаны в `4.9`, компоновка — в `4.12`, действия — в `4.16`, локальные
поведения — в `4.17`.

### 4.5. Вариант `ObjectList`

```text
ViewType = ObjectList
├── Columns[] → ViewColumn
├── QueryParameters[] → ViewQueryParameter
├── Filters[] → ViewFilter
├── FilterPresets[] → ViewFilterPreset
├── ActionMenuSections[] → ViewActionMenuSection
└── ActionPlacements[] → ViewActionPlacement
```

`ObjectList` предназначен для списка объектов. `QueryParameters`, `Filters` и
`FilterPresets` описывают настройки списка, но не исполняют запрос.

### 4.6. Вариант `LookupListView`

```text
ViewType = LookupListView
├── Columns[] → ViewColumn
├── QueryParameters[] → ViewQueryParameter
├── Filters[] → ViewFilter
├── FilterPresets[] → ViewFilterPreset
├── ActionMenuSections[] → ViewActionMenuSection
└── ActionPlacements[] → ViewActionPlacement
```

`LookupListView` предназначен для выбора значения. Его структура совпадает с
`ObjectList`, но ссылки на него из lookup-контрактов должны использовать именно
этот `ViewType`.

### 4.7. Вариант `Dashboard`

```text
ViewType = Dashboard
├── Widgets[] → ViewWidget
├── Layout[] → ViewLayoutNode
├── ActionMenuSections[] → ViewActionMenuSection
└── ActionPlacements[] → ViewActionPlacement
```

`Dashboard` предназначен для панели показателей. Коллекция `Widgets` разрешена
для него, но не для `Workspace`.

### 4.8. Вариант `Workspace`

```text
ViewType = Workspace
├── Layout[] → ViewLayoutNode
├── ActionMenuSections[] → ViewActionMenuSection
└── ActionPlacements[] → ViewActionPlacement
```

`Workspace` предназначен для рабочей области. В текущей schema коллекция
`Widgets` для него не разрешена.

### 4.9. Вложенный тип `ViewFormElement`

Коллекция `FormElements[]` разрешена только для `ObjectForm`. `ElementType`
определяет вид элемента: поле, текст или коллекция.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ElementType` | Вид элемента формы | `Enum` | Да | `Field`, `Text`, `Collection`; статический каталог вариантов | `Field` — поле объекта; `Text` — текст; `Collection` — коллекция объектов | Дискриминатор |
| `Title` | Заголовок элемента | `LocalizedText` | Нет | Локализованные значения | Подпись элемента | Все виды |
| `Description` | Описание элемента | `LocalizedText` | Нет | Локализованные значения | Пояснение элемента | Все виды |
| `Visible` | Видимость элемента | `Bool` | Нет | `true`, `false` | `true` показывает элемент; `false` скрывает его | Все виды |
| `Editable` | Редактируемость элемента | `Bool` | Нет | `true`, `false` | `true` разрешает редактирование при поддержке типа; `false` запрещает | Все виды |
| `Order` | Порядок элемента | `Number` | Нет | Число | Определяет порядок среди соседних элементов | Все виды |
| `FieldCode` | Код поля объекта | `ReferenceCode` | Для `Field` и `Collection` — да | `ObjectType.Members` | Связывает элемент с полем объекта | `Field`, `Collection`; запрещён для `Text` |
| `ValueType` | Тип отображаемого значения | `Enum` | Нет | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid`; статический каталог типов данных | Задаёт базовый тип значения для редактора и отображения | Только `Field` |
| `DisplayType` | Вид отображения | `Enum` | Нет | `text`, `enumText`, `badge`, `link`, `number`, `boolean`, `switch`, `date`, `dateTime`; статический каталог вариантов | Выбирает представление значения | Только `Field` |
| `DisplayFormat` | Формат отображения | `String` | Нет | Строка; синтаксис schema не задаёт | Передаёт формат рендерер | Только `Field` |
| `ComponentCode` | Компонент ввода | `Enum` | Нет | `TextBox`, `TextArea`, `Select`, `MultiSelect`, `Checkbox`, `Switch`, `Number`, `Date`, `DateTime`, `Lookup`, `Collection`, `Badge`; статический каталог вариантов | Выбирает рекомендуемый компонент редактора; рендерер принадлежит фронтенд-платформа | Только `Field` |
| `Text` | Текст элемента | `LocalizedText` | Для `Text` — да | Локализованные значения | Содержимое элемента с `ElementType = Text`; при других типах не является его значением | Только `Text`; запрещает `FieldCode` |
| `TargetViewCode` | Представление элементов коллекции | `ReferenceCode` | Для `Collection` — да | Каталог `Views`, только `ViewType = ObjectList` | Выбирает представление списка элементов коллекции | Только `Collection` |
| `ItemViewCode` | Представление карточки элемента | `ReferenceCode` | Нет | Каталог `Views`, только `ViewType = ObjectForm` | Выбирает представление карточки элемента | Только `Collection` |
| `ContextQueryParameterCode` | Параметр контекста | `ReferenceCode` | Нет | `View.QueryParameters` | Передаёт параметр контекста во вложенное представление | Только `Collection` |
| `ContextSourcePath` | Путь контекстного значения | `String` | Нет | Строковый путь; синтаксис schema не задаёт | Указывает, откуда взять значение контекста | Только `Collection` |
| `ItemOpenMode` | Способ открытия элемента | `Enum` | Нет | `Inline`, `Dialog`, `Drawer`, `Navigate`; статический каталог вариантов | Открывает элемент внутри списка, в диалоге, в выдвижной области или навигацией | Только `Collection` |

Правила: `Field` требует `FieldCode`; `Text` требует `Text` и запрещает
`FieldCode`; `Collection` требует `FieldCode` и `TargetViewCode`.

### 4.10. Вложенный тип `ViewColumn`

Коллекция `Columns[]` разрешена для `ObjectList` и `LookupListView`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ElementType` | Вид элемента списка | `Enum` | Да | `Column`; статический каталог вариантов | Подтверждает, что узел является колонкой | Все экземпляры |
| `Title` | Заголовок колонки | `LocalizedText` | Нет | Локализованные значения | Подпись колонки | Все экземпляры |
| `Description` | Описание колонки | `LocalizedText` | Нет | Локализованные значения | Пояснение колонки | Все экземпляры |
| `Visible` | Видимость колонки | `Bool` | Нет | `true`, `false` | Показывает или скрывает колонку | Все экземпляры |
| `Editable` | Редактируемость колонки | `Bool` | Нет | `true`, `false` | Разрешает или запрещает редактирование при поддержке runtime | Все экземпляры |
| `HideWhenGrouped` | Скрытие при группировке | `Bool` | Нет | `true`, `false` | Скрывает или показывает колонку при группировке | Все экземпляры |
| `IsKey` | Ключевая колонка | `Bool` | Нет | `true`, `false` | Помечает колонку как ключевую для потребителя | Все экземпляры |
| `Searchable` | Участие в поиске | `Bool` | Нет | `true`, `false` | Разрешает или запрещает поиск по колонке | Все экземпляры |
| `InlineEditable` | Редактирование в строке | `Bool` | Нет | `true`, `false` | Разрешает или запрещает редактирование прямо в строке | Все экземпляры |
| `Order` | Порядок колонки | `Number` | Нет | Число | Определяет положение колонки | Все экземпляры |
| `SortIndex` | Порядок сортировки | `Number` | Нет | Число | Определяет приоритет сортировки | При сортировке |
| `GroupIndex` | Порядок группировки | `Number` | Нет | Число | Определяет приоритет группировки | При группировке |
| `FieldCode` | Код поля объекта | `ReferenceCode` | Да | `ObjectType.Members` | Связывает колонку с отображаемым полем | Все экземпляры |
| `ValueType` | Тип значения колонки | `Enum` | Нет | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid`; статический каталог типов данных | Определяет тип значения для отображения, сортировки и запроса | Все экземпляры |
| `DisplayType` | Вид отображения колонки | `Enum` | Нет | `text`, `enumText`, `badge`, `link`, `number`, `boolean`, `switch`, `date`, `dateTime`; статический каталог вариантов | Выбирает рендерер значения | Все экземпляры |
| `DisplayFormat` | Формат отображения колонки | `String` | Нет | Строка; синтаксис schema не задаёт | Передаёт формат рендерер | Все экземпляры |
| `Width` | Ширина колонки | `Number` | Нет | Число; единица измерения schema не задаёт | Задаёт ширину колонки | Все экземпляры |
| `Sortable` | Разрешение сортировки | `Bool` | Нет | `true`, `false` | Разрешает или запрещает сортировку | Все экземпляры |
| `SortDirection` | Направление сортировки | `Enum` | Нет | `Asc`, `Desc`; статический каталог вариантов | `Asc` — по возрастанию; `Desc` — по убыванию | При сортировке |
| `GroupDirection` | Направление группировки | `Enum` | Нет | `Asc`, `Desc`; статический каталог вариантов | `Asc` — по возрастанию; `Desc` — по убыванию | При группировке |
| `InlineEditorType` | Компонент редактирования в строке | `Enum` | Нет | `TextBox`, `TextArea`, `Select`, `MultiSelect`, `Checkbox`, `Switch`, `Number`, `Date`, `DateTime`, `Lookup`, `Badge`; статический каталог вариантов | Выбирает рекомендуемый компонент; рендерер и запись принадлежат потребителям | При `InlineEditable = true` |
| `InlineEditCommitTrigger` | Момент сохранения ввода | `Enum` | Нет | `Blur`, `Enter`, `Change`; статический каталог вариантов | `Blur` — после потери фокуса; `Enter` — после Enter; `Change` — при изменении | При `InlineEditable = true` |

### 4.11. Вложенный тип `ViewWidget`

Коллекция `Widgets[]` разрешена только для `Dashboard`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ElementType` | Вид виджета | `Enum` | Да | `Widget`, `Grid`, `ReportWidget`, `EmbeddedViewHost`; статический каталог вариантов | Выбирает разновидность виджета; рендерер принадлежит фронтенд-платформа | Только `Dashboard` |
| `Title` | Заголовок виджета | `LocalizedText` | Нет | Локализованные значения | Подпись виджета | Все виды виджетов |
| `Description` | Описание виджета | `LocalizedText` | Нет | Локализованные значения | Пояснение виджета | Все виды виджетов |
| `Visible` | Видимость виджета | `Bool` | Нет | `true`, `false` | Показывает или скрывает виджет | Все виды виджетов |
| `Editable` | Редактируемость виджета | `Bool` | Нет | `true`, `false` | Разрешает или запрещает редактирование при поддержке рендерер | Все виды виджетов |
| `Order` | Порядок виджета | `Number` | Нет | Число | Определяет порядок виджетов | Все виды виджетов |
| `FieldCode` | Код поля объекта | `ReferenceCode` | Нет | `ObjectType.Members` | Связывает виджет с полем объекта | Если виджет использует поле |
| `ValueType` | Тип значения виджета | `Enum` | Нет | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid`; статический каталог типов данных | Определяет базовый тип значения | Если виджет использует значение |
| `DisplayType` | Вид отображения | `Enum` | Нет | `text`, `enumText`, `badge`, `link`, `number`, `boolean`, `switch`, `date`, `dateTime`; статический каталог вариантов | Выбирает отображение значения | Если виджет отображает значение |
| `DisplayFormat` | Формат отображения | `String` | Нет | Строка; синтаксис schema не задаёт | Передаёт формат рендерер | Если задан формат |
| `ComponentCode` | Код компонента | `Enum` | Нет | `TextBox`, `TextArea`, `Select`, `MultiSelect`, `Checkbox`, `Switch`, `Number`, `Date`, `DateTime`, `Lookup`, `Collection`, `Badge`; статический каталог вариантов | Выбирает рекомендуемый компонент виджета | Если виджет использует компонент |
| `Text` | Текст виджета | `LocalizedText` | Нет | Локализованные значения | Содержимое текстового виджета | Если виджет показывает текст |

### 4.12. Вложенный тип `ViewLayoutNode`

Коллекция `Layout[]` разрешена для `ObjectForm`, `Dashboard` и `Workspace`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `LayoutType` | Вид узла компоновки | `Enum` | Да | `Summary`, `Tabs`, `Tab`, `Group`, `Container`, `Placement`; статический каталог вариантов | Определяет роль узла компоновки | Дискриминатор |
| `Title` | Заголовок узла | `LocalizedText` | Для `Tab` — да | Локализованные значения | Заголовок вкладки или группы | `Tab` требует заголовок |
| `Order` | Порядок узла | `Number` | Нет | Число | Определяет порядок среди соседних узлов | Все виды |
| `ParentLayoutCode` | Родительский узел | `ReferenceCode` | Нет | `View.LayoutNodes` | Строит иерархию компоновки | Все виды |
| `Mode` | Режим представления | `Enum` | Нет | `View`, `Edit`, `Create`; статический каталог вариантов | Ограничивает узел режимом просмотра, редактирования или создания | Только `Container` |
| `ElementCode` | Размещаемый элемент | `ReferenceCode` | Для `Placement` — да | `View.Elements` | Размещает элемент в layout | Только `Placement`; запрещён для других типов |
| `ContainerDirection` | Направление контейнера | `Enum` | Нет | `Row`, `Column`, `horizontal`, `vertical`; статический каталог вариантов | Задаёт горизонтальное или вертикальное размещение | Только `Container` |
| `ContainerGap` | Промежуток контейнера | `Number` | Нет | Число; единица измерения schema не задаёт | Задаёт расстояние между элементами | Только `Container` |
| `ContainerSize` | Размер контейнера | `Number` | Нет | Число; единица измерения schema не задаёт | Задаёт размер контейнера | `Container` или `Group` |
| `ColumnsCount` | Число колонок | `Number` | Нет | Число | Задаёт количество колонок | Только `Group` |
| `ContainerResponsive` | Адаптивность контейнера | `Bool` | Нет | `true`, `false` | Включает или выключает адаптивное размещение | Только `Container` |
| `ContainerMinSize` | Минимальный размер | `String` | Нет | Строка; формат и единица измерения schema не задаёт | Ограничивает минимальный размер | `Container` или `Group` |
| `ContainerMaxSize` | Максимальный размер | `String` | Нет | Строка; формат и единица измерения schema не задаёт | Ограничивает максимальный размер | `Container` или `Group` |
| `GroupLayout` | Компоновка группы | `Enum` | Нет | `Fields`, `Grid`, `Stack`, `grid`, `stack`; статический каталог вариантов | Выбирает раскладку элементов группы | Только `Group` |
| `FieldLayout` | Компоновка поля | `Enum` | Нет | `Auto`, `Horizontal`, `Vertical`; статический каталог вариантов | Выбирает автоматическую, горизонтальную или вертикальную раскладку | Только `Placement` |
| `LabelWidthMode` | Режим ширины подписи | `Enum` | Нет | `Auto`, `Fixed`, `None`, `contentMax`; статический каталог вариантов | Выбирает автоматическую, фиксированную ширину или отсутствие подписи | Только `Group` |
| `HideTitle` | Скрытие заголовка группы | `Bool` | Нет | `true`, `false` | Показывает или скрывает заголовок группы | Только `Group` |
| `GroupMode` | Режим группы | `Enum` | Нет | `Section`, `Card`, `Inline`, `collapsible`; статический каталог вариантов | Выбирает секцию, карточку, строчное или сворачиваемое представление | Только `Group` |
| `DefaultExpanded` | Начальное раскрытие группы | `Bool` | Нет | `true`, `false` | Определяет, раскрыта ли группа при открытии | Только `Group` |

`ParentLayoutCode` задаёт родителя, а `Order` — порядок внутри родителя. В
каталоге сохранены варианты с различным регистром, например `Row` и
`horizontal`; это разные технические коды и их нельзя молча объединять.

### 4.13. Вложенный тип `ViewQueryParameter`

Коллекция `QueryParameters[]` разрешена для `ObjectList` и `LookupListView`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Code` | Код параметра запроса | `ReferenceCode` | Да | Стабильный код параметра | Идентифицирует параметр внутри `View` | Все экземпляры |
| `Kind` | Назначение параметра | `Enum` | Да | `Property`, `Search`, `Context`, `Constant`; статический каталог вариантов | `Property` — свойство объекта; `Search` — поиск; `Context` — контекст; `Constant` — постоянное значение | Дискриминатор смысла |
| `ValueType` | Тип значения параметра | `Enum` | Нет | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid`; статический каталог типов данных | Определяет тип значения и доступные операции | Все экземпляры |
| `Operator` | Оператор запроса | `Enum` | Нет | `Equals`, `NotEquals`, `Contains`, `StartsWith`, `EndsWith`, `GreaterThan`, `GreaterThanOrEqual`, `LessThan`, `LessThanOrEqual`, `In`, `HasFlag`, `FlagsAny`, `FlagsAll`, `FlagsExact`, `Between`, `IsNull`, `IsNotNull`; статический каталог операторов | Определяет сравнение, поиск, диапазон, проверку flags или null | Если параметр участвует в предикате |
| `PropertyCode` | Код свойства объекта | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле, участвующее в запросе | При `Kind = Property` |
| `PredicateCode` | Код предиката | `ReferenceCode` | Нет | Каталог и владелец не заданы schema `View` | Подключает предикат внешнего query-контракта | Если задан query-контрактом |
| `ContextSourceCode` | Код источника контекста | `ReferenceCode` | Нет | Каталог и владелец не заданы schema `View` | Подключает источник контекстного значения | При `Kind = Context` |
| `SearchFieldCodes` | Поля поиска | `Json` | Нет | JSON-массив кодов; формат schema не задаёт | Ограничивает поля, участвующие в поиске | При `Kind = Search` |
| `AllowedOperators` | Разрешённые операторы | `Json` | Нет | JSON-массив; формат schema не задаёт | Ограничивает операторы для параметра | Если задано ограничение |
| `AllowUserInput` | Пользовательский ввод | `Bool` | Нет | `true`, `false` | `true` разрешает ввод; `false` запрещает его | Для параметров, управляемых пользователем |
| `Required` | Обязательность параметра | `Bool` | Нет | `true`, `false` | `true` требует значение; `false` не требует его | Все экземпляры |
| `Order` | Порядок параметра | `Number` | Нет | Число | Определяет порядок параметра в редакторе и контракте | Все экземпляры |

### 4.14. Вложенный тип `ViewFilter`

Коллекция `Filters[]` разрешена для `ObjectList` и `LookupListView`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Заголовок фильтра | `LocalizedText` | Нет | Локализованные значения | Подпись фильтра | Все экземпляры |
| `Placeholder` | Подсказка ввода | `LocalizedText` | Нет | Локализованные значения | Подсказывает формат ввода | Если компонент поддерживает подсказку |
| `EmptyOptionLabel` | Подпись пустого варианта | `LocalizedText` | Нет | Локализованные значения | Подписывает отсутствие выбранного значения | Для выбора с пустым вариантом |
| `QueryParameterCode` | Параметр запроса | `ReferenceCode` | Да | `View.QueryParameters` | Связывает фильтр с параметром запроса | Все экземпляры |
| `ValueType` | Тип значения фильтра | `Enum` | Нет | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid`; статический каталог типов данных | Определяет тип ввода и проверки | Все экземпляры |
| `Placement` | Место фильтра | `Enum` | Нет | `SearchBar`, `QuickFilter`, `FilterPanel`, `Hidden`, `Quick`, `Advanced`; статический каталог вариантов | Выбирает место и режим отображения | Все экземпляры |
| `EditorType` | Компонент ввода фильтра | `Enum` | Нет | `TextBox`, `TextArea`, `Select`, `MultiSelect`, `Checkbox`, `Number`, `Date`, `DateTime`, `Lookup`; статический каталог вариантов | Выбирает компонент ввода | Все экземпляры |
| `Order` | Порядок фильтра | `Number` | Нет | Число | Определяет порядок фильтров | Все экземпляры |
| `MinLength` | Минимальная длина ввода | `Number` | Нет | Число символов | Не применяет фильтр до достижения длины | Если ввод текстовый |
| `DebounceMs` | Задержка применения | `Number` | Нет | Миллисекунды | Откладывает применение фильтра после ввода | Если фильтр применяет debounce |
| `ValueSetCode` | Набор значений | `ReferenceCode` | Нет | `Catalog.ValueSets` | Выбирает источник вариантов | Взаимоисключим с `SystemEnumCode` и `WorkflowCode` |
| `SystemEnumCode` | Системное перечисление | `ReferenceCode` | Нет | `Catalog.SystemEnums` | Выбирает системный источник вариантов | Взаимоисключимо с `ValueSetCode` и `WorkflowCode` |
| `WorkflowCode` | Код workflow-источника | `ReferenceCode` | Нет | Каталог и владелец не заданы schema | Выбирает внешний источник вариантов | Взаимоисключим с `ValueSetCode` и `SystemEnumCode` |
| `LookupSourceCode` | Клиентский источник lookup | `ReferenceCode` | Нет | `View.LookupSources`; режим `Client` | Выбирает источник вариантов lookup; данные не принадлежат `View` | При использовании lookup; зависит от `LookupViewCode` |
| `LookupViewCode` | Представление выбора | `ReferenceCode` | Нет | Каталог `Views`, только `LookupListView` | Выбирает представление для lookup | При использовании lookup |
| `LookupPrefilter` | Предварительный фильтр lookup | `Json` | Нет | JSON; формат schema не задаёт | Ограничивает исходный набор lookup | При использовании lookup |
| `HiddenWhen` | Условие скрытия | `Json` | Нет | JSON-условие; язык и формат schema не задаёт | Скрывает фильтр при выполнении условия | Если задано условие |
| `VisibleWhen` | Условие видимости | `Json` | Нет | JSON-условие; язык и формат schema не задаёт | Показывает фильтр при выполнении условия | Если задано условие |
| `DisplayTemplateCode` | Шаблон отображения | `ReferenceCode` | Нет | Каталог и владелец не заданы schema `View` | Выбирает шаблон отображения значения | Если рендерер поддерживает шаблон |
| `AllowEmpty` | Разрешение пустого значения | `Bool` | Нет | `true`, `false` | `true` разрешает пустое значение; `false` запрещает его | Все экземпляры |

### 4.15. Вложенные типы `ViewFilterPreset` и `ViewFilterPresetValue`

`ViewFilterPreset` входит в `FilterPresets[]`, а `Values[]` является его
вложенной коллекцией типа `ViewFilterPresetValue`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ViewFilterPreset.Title` | Название преднастройки | `LocalizedText` | Да | Локализованные значения | Подпись преднастройки | Все экземпляры |
| `ViewFilterPreset.Description` | Описание преднастройки | `LocalizedText` | Нет | Локализованные значения | Поясняет преднастройку | Все экземпляры |
| `ViewFilterPreset.Order` | Порядок преднастройки | `Number` | Нет | Число | Определяет порядок преднастроек | Все экземпляры |
| `ViewFilterPreset.VisibleWhen` | Условие видимости | `Json` | Нет | JSON-условие; язык и формат schema не задаёт | Условно показывает преднастройку | Если задано условие |
| `ViewFilterPreset.EnabledWhen` | Условие доступности | `Json` | Нет | JSON-условие; язык и формат schema не задаёт | Условно делает преднастройку доступной | Если задано условие |
| `ViewFilterPreset.IconCode` | Код значка | `ReferenceCode` | Нет | Каталог и рендерер не заданы schema | Выбирает значок преднастройки | Если рендерер поддерживает значок |
| `ViewFilterPreset.Style` | Визуальный стиль | `Enum` | Нет | `Default`, `Primary`, `Success`, `Warning`, `Danger`; статический каталог вариантов | Выбирает визуальный стиль преднастройки | Все экземпляры |
| `ViewFilterPresetValue.QueryParameterCode` | Параметр преднастройки | `ReferenceCode` | Да | `View.QueryParameters` | Связывает значение преднастройки с параметром | Все экземпляры |
| `ViewFilterPresetValue.Operator` | Оператор преднастройки | `Enum` | Нет | Каталог операторов `ViewQueryParameter` | Определяет проверку значения преднастройки | Если задан оператор |
| `ViewFilterPresetValue.Value` | Одиночное значение | `Json` | Нет | JSON; форма зависит от оператора | Хранит одиночное значение | Если оператор принимает одно значение |
| `ViewFilterPresetValue.Values` | Множественные значения | `Json` | Нет | JSON-массив; форма зависит от оператора | Хранит список значений | Если оператор принимает список |
| `ViewFilterPresetValue.IsCleared` | Признак очистки | `Bool` | Нет | `true`, `false` | `true` очищает значение параметра; `false` сохраняет его | Все экземпляры |
| `ViewFilterPresetValue.Order` | Порядок значения | `Number` | Нет | Число | Определяет порядок значений в преднастройке | Все экземпляры |

### 4.16. Вложенные типы действий

`ViewActionMenuSection` и `ViewActionPlacement` размещают существующие действия.
Они не описывают обработчик, параметры, результат или права `Action`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ViewActionMenuSection.Title` | Название секции меню | `LocalizedText` | Да | Локализованные значения | Подпись секции меню | Все экземпляры |
| `ViewActionMenuSection.Description` | Описание секции меню | `LocalizedText` | Нет | Локализованные значения | Пояснение секции | Все экземпляры |
| `ViewActionMenuSection.Order` | Порядок секции | `Number` | Нет | Число | Определяет порядок секций | Все экземпляры |
| `ViewActionMenuSection.Hidden` | Скрытие секции | `Bool` | Нет | `true`, `false` | Скрывает или показывает секцию | Все экземпляры |
| `ViewActionPlacement.ActionCode` | Код действия | `ReferenceCode` | Да | `Catalog.Actions` | Ссылается на существующее действие | Все экземпляры |
| `ViewActionPlacement.Hidden` | Скрытие действия | `Bool` | Нет | `true`, `false` | Скрывает или показывает действие | Все экземпляры |
| `ViewActionPlacement.Placement` | Место действия | `Enum` | Нет | `Toolbar`, `EmbeddedCollectionToolbar`, `Row`, `ActionsMenu`, `ContextMenu`, `CardHeader`, `Footer`; статический каталог вариантов | Определяет место размещения действия; `EmbeddedCollectionToolbar` публикует action только в контексте встроенной коллекции и отображается в её toolbar | Все экземпляры |
| `ViewActionPlacement.MenuSection` | Секция меню | `ReferenceCode` | Нет | `View.ActionMenuSections` | Помещает действие в секцию меню | Если `Placement = ActionsMenu` |
| `ViewActionPlacement.DisplayKind` | Вид элемента действия | `Enum` | Нет | `Button`, `IconButton`, `Menu`; статический каталог вариантов | Выбирает кнопку, иконку-кнопку или меню | Все экземпляры |
| `ViewActionPlacement.Order` | Порядок действия | `Number` | Нет | Число | Определяет порядок действий | Все экземпляры |
| `ViewActionPlacement.VisibleWhen` | Условие видимости | `String` | Нет | Строковое условие; язык и формат schema не задаёт | Условно показывает действие | Если задано условие |
| `ViewActionPlacement.EnabledWhen` | Условие доступности | `String` | Нет | Строковое условие; язык и формат schema не задаёт | Условно делает действие доступным | Если задано условие |

### 4.17. Вложенный тип `ViewLocalBehavior`

Коллекция `LocalBehaviors[]` разрешена только для `ObjectForm`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название поведения | `LocalizedText` | Нет | Локализованные значения | Подпись поведения | Все экземпляры |
| `ConditionExpression` | Условие поведения | `String` | Нет | Строковое выражение; язык и контекст schema не задаёт | Ограничивает применение поведения | Если задано условие |
| `TargetKind` | Уровень цели | `Enum` | Да | `View`, `Element`; статический каталог вариантов | `View` — поведение относится ко всему представлению; `Element` — к элементу | Дискриминатор |
| `TargetCode` | Код элемента | `ReferenceCode` | Для `Element` — да | `View.Elements` | Указывает элемент, к которому применяется поведение | Только `Element`; запрещён для `View` |
| `EffectKind` | Вид эффекта | `Enum` | Да | `Warn`, `SetVisible`, `SetEnabled`, `SetReadOnly`, `SetReadonly`, `SetEditable`, `SetRequired`, `ReloadOptions`, `AllowedValues`; статический каталог вариантов | Определяет предупреждение, видимость, доступность, редактирование, обязательность, обновление вариантов или ограничение значений | Все экземпляры; точный эффект принадлежит frontend-контракту |
| `EffectValue` | Значение эффекта | `Json` | Нет | JSON; форма зависит от `EffectKind`, schema её не задаёт | Параметр эффекта | Если выбранный эффект требует параметр |
| `ReasonCode` | Код причины | `String` | Нет | Строковый код; каталог schema не задаёт | Классифицирует причину поведения | Если потребитель использует классификацию |
| `Order` | Порядок поведения | `Number` | Нет | Число | Определяет порядок применения поведений | Все экземпляры |

### 4.18. Связи и структурные правила

Ключевые связи, которые определяют структуру `View`:

- `ViewType` определяет допустимые корневые коллекции;
- `ElementType` определяет допустимую форму `ViewFormElement`;
- `LayoutType` определяет допустимые свойства `ViewLayoutNode`;
- `TargetKind` определяет обязательность и запрет `TargetCode`;
- `TargetViewCode` и `ItemViewCode` должны ссылаться на представления с
  допустимым `ViewType`;
- `QueryParameterCode`, `DefaultPresetCode`, `MenuSection` и другие ссылки на
  дочерние узлы должны указывать на узел текущего `View`;
- `ValueSetCode`, `SystemEnumCode` и `WorkflowCode` взаимоисключимы;
- `ParentLayoutCode` не должен образовывать цикл.

Локальное условие конкретного свойства указано в последнем столбце его таблицы.
Этот раздел содержит только связи между несколькими свойствами и не является
вторым каталогом свойств.

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

Конкретный допустимый набор и источник указаны в строке соответствующего
свойства в разделе `4`. Серверная часть Configuration является источником
истины для схемы, статических каталогов вариантов и каталогов опубликованных
артефактов. Клиентский редактор получает значения через контракт редактора,
может обновлять зависимый список после изменения исходного свойства, но не
расширяет допустимый набор. Сервер повторно проверяет сохранённые ссылки и
значения.

Этот раздел не является вторым каталогом свойств.

Для текущего контракта `View` подтверждён режим `ServerSnapshot` для серверных
каталогов. `LookupSourceCode` является исключением: он ссылается на клиентский
источник `View.LookupSources`, зависит от `LookupViewCode` и фильтруется на
клиенте. `View` при этом не становится владельцем данных источника.

Зависимый список должен обновляться после изменения исходного свойства, а
сохранённая ссылка повторно проверяется сервером. Например, изменение
`ViewType` ограничивает `TemplateCode`, а изменение `LookupViewCode` ограничивает
источник lookup.

## 6. Ограничения и зависимости

Сервер проверяет следующие ограничения и зависимости:

- соответствие `TemplateCode` допустимому `ViewType`;
- разрешённость коллекции для выбранного `ViewType`;
- обязательные свойства узла по его дискриминатору;
- ссылки на `ObjectType.Members`, `View`, `View.QueryParameters` и другие
  дочерние узлы;
- взаимоисключение источников значений фильтра;
- отсутствие циклов в `ViewLayoutNode`.

Проверка на клиенте может использоваться для удобства редактора, но не является
источником истины и не заменяет серверную проверку.

## 7. Операции над структурой

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| `FormElements`, `Columns`, `Layout`, `QueryParameters`, `Filters`, `FilterPresets`, `ActionMenuSections`, `ActionPlacements`, `LocalBehaviors` | Да | Через общий контракт записи | Поддерево | Да | Только если коллекция разрешена для `ViewType` |
| `Widgets` | Не объявлено схемой | Да | Не объявлено схемой | Не объявлено схемой | Только для `Dashboard` |
| `ViewFilterPreset.Values` | Не объявлено схемой | Не объявлено схемой | Не объявлено схемой | Не объявлено схемой | `allowedOperations` явно не задан; не считать подтверждённой editor-операцией без отдельной проверки |
| Отдельное свойство | Не применяется | Через общий контракт записи | Не применяется | Не применяется | С учётом обязательности, зависимостей и состояния версии |

Фактические API-команды, права и доступность действий редактора описываются в
`03_contracts.md` и `06_user_experience.md`. Эта таблица фиксирует только
schema-level возможности.

## 8. Наследование и рассчитанный результат

Для текущего schema-контракта `View` отдельная модель наследования не
подтверждена. `InheritedOnly` у ссылки или свойства не следует трактовать как
доказательство полной модели наследования `View`; такое поведение требует
отдельного контракта Configuration.

Рассчитанное представление и runtime-модель не являются новым типом
конфигурационного артефакта.

## 9. Создание, проверка и публикация

Перед публикацией проверяются как минимум:

- наличие допустимого `ViewType`;
- допустимость `TemplateCode` для выбранного типа;
- разрешённость каждой дочерней коллекции;
- обязательные свойства по `ElementType`, `LayoutType` и `TargetKind`;
- ссылки на поля объекта, представления, параметры, преднастройки, layout-узлы
  и действия;
- допустимые типы и значения локального поведения;
- отсутствие циклов в layout и недопустимых ссылок на дочерние узлы.

Публикация `View` выполняется общим lifecycle Configuration. Published/effective
представление является входом для потребителя, но Configuration не исполняет
рендерер, frontend-приложение или механизм выполнения запросов.

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `View` и одиннадцать дочерних типов схемы зарегистрированы в реестре схем:
  десять прямых типов и вложенный `ViewFilterPresetValue`;
- `ViewType` используется как дискриминатор;
- ограничения дочерних коллекций и свойств заданы schema;
- канонический builder и Artifact Editor работают с layout, form elements,
  columns, filters, presets, actions и local behaviors;
- `ObjectTypeCode` используется как контекст object-bound представления.

### За пределами текущего schema-контракта `View`

- окончательный владелец и контракт `DatasetCode` / `Read Query Capability`;
- расширение каталога `TemplateCode` и рендерер для каждого нового кода;
- общий runtime-контракт `Dashboard` и `Workspace`;
- production-правила query, paging, sorting и фильтрации;
- полная модель приложений Admin, Studio и Runtime.

Это границы данного schema-контракта, а не утверждение, что перечисленные
возможности не нужны. Configuration регистрирует новый `TemplateCode` и его
допустимые `ViewType`, а фронтенд-платформа регистрирует соответствующий
рендерер для поддерживаемых приложений.

Открытые решения и маршруты переноса фиксируются в [трассировке Configuration](../90_traceability.md).
В частности, там описаны `CFG-DEC-03` для `Dataset` / `Read Query Capability`,
`CFG-DEC-05` для Frontend Studio, `CFG-DEC-09` для `TemplateCode` и рендерер и
`CFG-DEC-10` для запроса `View` и runtime-контракта.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Представление | View | Корневой конфигурационный артефакт отображения. |
| Тип представления | `ViewType` | Дискриминатор, определяющий назначение и допустимую структуру `View`. |
| Шаблон представления | View template / `TemplateCode` | Зарегистрированный код визуального каркаса, отрисовываемого клиентским runtime. |
| Элемент представления | View element | Вложенный узел `View`, например `ViewFormElement`, `ViewColumn` или `ViewWidget`. |
| Компоновка представления | View layout / `Layout` | Иерархическая структура контейнеров и элементов представления. |
| Преднастройка фильтра | Filter preset / `ViewFilterPreset` | Сохранённый набор настроек фильтрации представления. |
| Рендерер представления | View renderer | Клиентская реализация для отрисовки `TemplateCode`; не часть schema `View`. |

Общие термины Configuration ведутся в [тематическом глоссарии][configuration-terms].
Локальная таблица нужна для чтения документа и не является отдельным источником
истины.

## 12. Источники в коде и тестах

- [View schemas][schemas]
- [Статический каталог вариантов View][static-options]
- [Каталог шаблонов View][template-catalog]
- [View node builder][node-builder]
- [Schema registry tests][schema-tests]
- [View builder tests][builder-tests]
- [Artifact Editor tests][editor-tests]

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 16:08 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #63: обновлена проектная документация на ядро платформы + быстрый старт | [PR #63](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/63) |
| 0.2 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | 16. Вложенные типы действий | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
