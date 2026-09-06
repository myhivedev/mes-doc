---
id: DOC-03-02-AT-MENU
title: 'Тип конфигурационного артефакта — Menu'
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
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Тип конфигурационного артефакта — Menu

[schemas]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/NavigationArtifactSchemas.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/NavigationArtifactSchemas.cs
[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/MenuNodeBuilder.cs
[editor]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Services/Editor/MenuArtifactEditorReadProjector.cs
[editor-command]: ../../../../src/Platform/DMP.Platform.Configuration/Application/Services/Editor/ConfigurationArtifactEditorCommandService.cs
[validator]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Services/ArtifactValidator.cs
[materializer]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Services/Materializers/RuntimeMenuMaterializer.cs
[runtime-contract]: ../../../../src/Platform/DMP.Platform.Contracts/Runtime/Responses/RuntimeViewResolveResponse.cs
[icon-registry]: ../../../../src/Frontend/packages/ui/src/components.tsx
[navigation-icons]: ../../../../src/Frontend/packages/ui/src/icons/navigation/README.md
[runtime-navigation-adapter]: ../../../../src/Frontend/packages/runtime-react/src/model/runtimeNavigationAdapter.tsx
[configuration-terms]: ../../../11_glossary/configuration_terms.md
[tests]: ../../../../tests/DMP.Platform.IntegrationTests/DemoProductDefinition/DemoProductDefinitionNavigationIntegrationTests.cs
[traceability]: ../90_traceability.md

## 1. Назначение

`Menu` — корневой конфигурационный артефакт дерева навигации приложения.
Артефакт хранит название, описание и единую коллекцию `NavigationItems[]`.
Коллекция содержит как структурные группы, так и пункты перехода.

Этот документ описывает только schema-контракт `Menu` и его вложенного типа
`NavigationItem`. Shell, маршрутизация приложения, отрисовка интерфейса,
проверка прав и выполнение `Action` описываются у соответствующих владельцев.

## 2. Идентичность и граница

Идентичность корневого `Menu` задаётся техническим полем
`ArtifactNode.Code`. Для корня значение `ArtifactNode.ArtifactTypeCode` равно
`Menu`. Отдельного schema-свойства `MenuCode` нет.

Идентичность `NavigationItem` также задаётся `ArtifactNode.Code`, а тип узла
равен `NavigationItem`. `NavigationItem` не является отдельным корневым
артефактом: он хранится внутри коллекции `Menu.NavigationItems`.

`ModuleCode` относится к владельцу baseline package или contribution-контексту.
Он не является свойством schema `Menu` и не изменяет идентичность корня в
каноническом документе.

В документ входят:

- `Menu` и его свойства `Title`, `Description`;
- коллекция `NavigationItems[]`;
- общие свойства `NavigationItem`;
- условные свойства пункта `NavigationItem` для его `TargetType`;
- schema-правила и операции над иерархической коллекцией.

В документ не входят shell state, layout приложения, runtime router, icon
registry, permission catalog, action handler и workflow execution.

## 3. Место в Configuration

| Владелец или слой | Ответственность | Где описано |
| --- | --- | --- |
| Configuration | Schema `Menu`, effective configuration, authoring и редактор дерева | Этот документ и документы Configuration |
| Baseline или application owner | Поставка корневого меню, его `Title` и `Description` в baseline package | Baseline manifest соответствующего владельца |
| Feature module | Поставка принадлежащих модулю `NavigationItem` в согласованный menu-контекст | Baseline package прикладного модуля |
| Object Runtime | Потребление опубликованного меню при разрешении runtime view | Документы Object Runtime |
| фронтенд-платформа и приложения | Shell, routing, рендерер, icon registry и отображение меню | Документы фронтенд-платформа и приложений |
| Tenant Security | Runtime authorization и permission decision | Документы Tenant Security |

`Menu` может ссылаться на `View` и `Action`, но не включает их в свою схему.
Ссылка на другой артефакт не меняет владельца или границу этого артефакта.

## 4. Структура и схема `Menu`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств.

```text
Menu (ArtifactNode)
├── identity: ArtifactTypeCode = Menu; Code = <MenuCode>
├── properties: { Title, Description }
└── child collections:
    └── NavigationItems[0..n] → NavigationItem
        (hierarchical; ParentCode → NavigationItem.Code)
        ├── identity: Code = <ItemCode>
        ├── properties: { ItemKind, Title, IconCode, ParentCode, Order, Visible,
        │                TargetType, target-specific properties }
        ├── ItemKind = Group: target-specific properties are not allowed
        └── ItemKind = Item:
            ├── TargetType = View: TargetViewCode, ResolutionMode, ResolutionSource
            ├── TargetType = ExternalUrl: Url
            ├── TargetType = InternalRoute: Route
            └── TargetType = Action: ActionCode
```

`NavigationItems[]` является одной иерархической коллекцией. `Group` и `Item`
различаются значением discriminator-свойства `ItemKind`; отдельной коллекции
для каждого типа нет. Иерархия не записывается физическим вложением узлов:
родитель определяется ссылкой `ParentCode`, а `Order` задаёт порядок среди
элементов одного родителя. ([schema][schemas]; [collection codes][collection-codes])

### 4.2. Свойства корневого узла

В таблицах свойств используется единый набор столбцов. `Не задано` означает,
что schema не задаёт значение или каталог; это не означает автоматически
`null` и не означает, что runtime обязан принять любое значение.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название меню | `LocalizedText` | Нет; schema default не задан | Локализованные значения; язык выбирается потребителем | Название, которое возвращается в runtime-описании меню | Для всех `Menu`; может быть переопределено в разрешённом configuration layer |
| `Description` | Описание меню | `LocalizedText` | Нет; schema default не задан | Локализованные значения | Пояснение назначения меню; входит в authoring-модель, но не возвращается текущим runtime menu response | Для всех `Menu`; может быть переопределено в разрешённом configuration layer |

`MenuCode`/`Code` не является schema property: это технический код корневого
узла. `ModuleCode` также не является свойством этой таблицы. ([builder][builder];
[runtime contract][runtime-contract])

### 4.3. Общие свойства `NavigationItem`

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ItemKind` | Тип пункта | `Enum` | Да; default не задан | `Group`, `Item`; статический каталог backend | Выбирает структурную группу или пункт перехода и определяет допустимую ветку схемы | Для каждого `NavigationItem`; discriminator, обычное изменение после создания не подтверждено |
| `Title` | Название пункта | `LocalizedText` | Нет; schema default не задан | Локализованные значения | Подпись группы или пункта; runtime использует `Code` как fallback при отсутствии пригодного заголовка | Для `Group` и `Item`; может быть переопределено |
| `IconCode` | Код иконки | `String` | Нет; default не задан | Строковый код, зарегистрированный в frontend-реестре `ActionIcon`; навигационные коды перечислены в [каталоге навигационных иконок][navigation-icons] | Передаёт потребителю смысловую ссылку на иконку; путь к SVG, имя файла или имя frontend-компонента не является значением schema | Для `Group` и `Item`; рендерер/реестр значков принадлежат frontend |
| `ParentCode` | Код родительской группы | `ReferenceCode` | Нет; `null` означает корень дерева | `Menu.NavigationItems`; существующий `NavigationItem` типа `Group` в том же `Menu` | Строит иерархию; при наличии значения элемент становится дочерним узлом группы | Для `Group` и `Item`; validator не допускает self-reference, цикл и parent не-группу |
| `Order` | Порядок пункта | `Number` | Нет; runtime использует вычисленный fallback порядка | Число; отдельный диапазон schema не задаёт | Определяет порядок элементов среди соседей | Для `Group` и `Item`; используется при сортировке effective/runtime дерева |
| `Visible` | Базовая видимость | `Bool` | Нет; schema default не задан, runtime fallback — `true` | `true` — пункт видим; `false` — пункт помечен невидимым | Передаёт базовое состояние видимости потребителю; это не заменяет runtime permission decision | Для `Group` и `Item`; может быть переопределено в effective configuration |

Технический код `NavigationItem` равен `ArtifactNode.Code`; отдельного schema
свойства `Code` нет. `ParentCode` является ссылкой на код соседнего дочернего
узла, а не на отдельный корневой артефакт. ([schema][schemas]; [source codes][source-codes])

### 4.4. Ветка `ItemKind = Group`

`Group` — структурный контейнер дерева. У него нет target. Свойства
`TargetType`, `TargetViewCode`, `ResolutionMode`, `ResolutionSource`, `Url`,
`Route` и `ActionCode` запрещены schema-правилами.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `ItemKind` | Тип пункта | `Enum` | Да; значение `Group` | `Group` | Делает узел контейнером и запрещает target-specific properties | При создании группы |
| `Title` | Название группы | `LocalizedText` | Нет; schema default не задан | Локализованные значения | Подпись структурной группы | Для каждой группы |
| `IconCode` | Код иконки группы | `String` | Нет; default не задан | Код, зарегистрированный в frontend-реестре `ActionIcon`; навигационные коды — в [каталоге навигационных иконок][navigation-icons] | Передаёт иконку группы потребителю; неизвестный для рендерер код не даёт иконку | Для группы, если рендерер поддерживает иконку |
| `ParentCode` | Код родительской группы | `ReferenceCode` | Нет; `null` для корневой группы | `Menu.NavigationItems`, только группа того же `Menu` | Вкладывает группу в другую группу | При построении дерева |
| `Order` | Порядок группы | `Number` | Нет; default не задан | Число | Определяет порядок группы среди соседей | При сортировке коллекции |
| `Visible` | Видимость группы | `Bool` | Нет; runtime fallback — `true` | `true` или `false` | Передаёт базовую видимость группы | При отображении assembled menu |

### 4.5. Ветка `ItemKind = Item`

`Item` — пункт, который задаёт один тип target. `TargetType` обязателен для
`Item`; в зависимости от его значения schema разрешает и требует ровно
соответствующее target-specific property.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `TargetType` | Тип цели | `Enum` | Да для `Item`; запрещён для `Group` | `View`, `ExternalUrl`, `InternalRoute`, `Action`; статический каталог backend | Выбирает ветку target и набор обязательных свойств | Только для `Item`; задаётся при создании или guarded edit |
| `TargetViewCode` | Код целевого представления | `ReferenceCode` | Да при `TargetType = View` | `Catalog.AllViews` | Выбирает `View`, который должен открыть пункт | Только `Item + View`; cross-module code может быть квалифицирован `ModuleCode:Code` |
| `ResolutionMode` | Режим разрешения цели | `Enum` | Нет; schema default не задан | `None`, `Context`, `Query`, `CreateNew`; статический каталог backend | Определяет, нужно ли разрешать объект или контекст при открытии `View` | Только `Item + View`; текущий schema не требует значение, runtime-контракт уточняется владельцем |
| `ResolutionSource` | Источник разрешения цели | `String` | Нет; schema default не задан | Строка; каталог и формат schema не заданы | Передаёт дополнительный источник контекста для `ResolutionMode` | Только `Item + View`; допустимость зависит от runtime/frontend-контракта |
| `Url` | Внешний URL | `String` | Да при `TargetType = ExternalUrl` | Строка; schema проверяет только условную допустимость, формат URL отдельно не задан | Задаёт внешний адрес перехода | Только `Item + ExternalUrl` |
| `Route` | Внутренний маршрут | `String` | Да при `TargetType = InternalRoute` | Строка; route registry schema не задан | Задаёт маршрут приложения | Только `Item + InternalRoute`; обработка принадлежит приложению/frontend |
| `ActionCode` | Код действия | `ReferenceCode` | Да при `TargetType = Action` | `Catalog.AllActions` | Задаёт ссылку на `Action`, запускаемый потребителем | Только `Item + Action`; handler и параметры `Action` здесь не описываются |

Значение `TargetType` определяет не только target, но и обязательность и
видимость остальных свойств. При смене `TargetType` несовместимые свойства
должны быть удалены, сброшены или отклонены серверной проверкой; текущий
schema-контракт не разрешает хранить их как допустимую конфигурацию.
([schema rules][schemas])

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

| Группа значений | Источник | Канал получения | Обновление | Кто проверяет |
| --- | --- | --- | --- | --- |
| `ItemKind` | `ConfigurationNavigationItemKindCodes` | Статический backend catalog | При поставке новой версии кода | Configuration schema и validator |
| `TargetType` | `ConfigurationNavigationTargetTypeCodes` | Статический backend catalog | При поставке новой версии кода | Configuration schema и validator |
| `ResolutionMode` | `ConfigurationNavigationResolutionModeCodes` | Статический backend catalog | При поставке новой версии кода | Configuration schema |
| `TargetViewCode` | `Catalog.AllViews` | Server-side catalog source | Обновляется при изменении доступного каталога views | Configuration editor/schema; фактическое разрешение — runtime owner |
| `ActionCode` | `Catalog.AllActions` | Server-side catalog source | Обновляется при изменении доступного каталога actions | Configuration editor/schema; выполнение — runtime owner |
| `ParentCode` | `Menu.NavigationItems` | Server-side effective sibling collection | Обновляется вместе с effective menu tree | Configuration validator/editor |
| `Title`, `Description` | Localized text values | Configuration storage/effective projection | Через draft и publish | Configuration validation; language fallback — consumer |
| `IconCode` | Ручное или baseline значение | Server-side string value; каталог рендерера — client-side | При изменении baseline или draft | Schema проверяет тип; icon registry проверяет consumer |
| `Url`, `Route`, `ResolutionSource` | Ручное или baseline значение | Server-side string value | При изменении draft и publish | Schema проверяет тип; формат и исполнение принадлежат consumer |

`IconCode` остаётся строковым schema-значением: Configuration не проверяет
наличие кода в frontend. В текущем frontend код разрешается общим реестром
`ActionIcon`; полный фактический набор кодов находится в [реестре иконок][icon-registry],
а предметные `navigation.*` коды и их назначение — в [каталоге навигационных
иконок][navigation-icons]. Если код не зарегистрирован в consumer, иконка не
отрисовывается.

Текущая schema не задаёт отдельный client refresh protocol и не регистрирует
каталог `Route` или `ResolutionSource`. Наличие frontend registry
или route handler не превращает этот registry в свойство `Menu`.

## 6. Ограничения и зависимости

- `Menu.NavigationItems[]` — иерархическая ordered collection с допустимыми
  операциями create, delete subtree, move и reorder.
- `ParentCode` может ссылаться только на `NavigationItem` с `ItemKind = Group` в
  том же effective menu.
- Группа не имеет target-specific properties.
- Для пункта обязателен `TargetType`; для каждого target требуется своё
  условное свойство: `TargetViewCode`, `Url`, `Route` или `ActionCode`.
- `ResolutionMode` и `ResolutionSource` допустимы только для `TargetType = View`.
- `TargetViewCode` и `ActionCode` являются ссылками на каталоги потребляемых
  артефактов, а не вложенными `View` или `Action`.
- `NavigationItem` с `TargetType = Action` ссылается на `Action`, но не
  запускает workflow `CommandCode` и не содержит handler или parameters.
- `Visible` задаёт базовую видимость. Permission-based visibility и policy
  runtime не являются schema properties этого артефакта.
- `ModuleCode` и owner baseline участвуют в поставке и сборке, но не являются
  частью root schema.

## 7. Операции над структурой

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| Корневой `Menu` | Да, через root editor/baseline builder | `Title`, `Description` и effective overrides | По общим правилам root-артефактов | Не применяется | `Code` является identity и не является обычным редактируемым свойством |
| `NavigationItems[]` | Да, через `MenuNodeBuilder` или editor fixed choice | Свойства узла и guarded parent change | Да, удаление subtree | Да, через `Order` | Коллекция иерархическая; операции зарегистрированы schema |
| `NavigationItem` с `Group` | Да | Общие свойства группы и `ParentCode` | Да | Да | Target properties запрещены |
| `NavigationItem` с `Item` | Да | Общие и допустимые target properties | Да | Да | `TargetType` определяет условные свойства и обязательность |

Переименование сохранённого `ArtifactNode.Code` не является обычным
редактированием свойства: код участвует в identity и effective merge. Точные
правила editor session и tombstone/reset описываются общим Configuration
editor-контрактом; проверка перемещения под самого себя или своего потомка
выполняется editor command service. ([schema][schemas]; [builder][builder];
[editor][editor]; [editor-command][editor-command])

## 8. Наследование и рассчитанный результат

`Menu` не имеет отдельного свойства наследования. Он участвует в общем
effective resolution Configuration вместе с другими root-артефактами.

Для `NavigationItems[]` используется иерархическое слияние по identity
дочерних узлов. В результате runtime получает одно effective menu tree, а не
набор несвязанных фрагментов. Порядок, `ParentCode`, `ItemKind` и target
properties проверяются на эффективной конфигурации.

`Title`, `Description` и свойства `NavigationItem` могут быть значениями
базового слоя или override-слоя, если это разрешено общими правилами
Configuration. Это не означает, что consumer может изменить schema или
статические каталоги `ItemKind`, `TargetType` и `ResolutionMode`.
([merge policy][schemas])

## 9. Создание, проверка и публикация

1. Baseline или editor создаёт корневой `Menu` с техническим кодом
   `ArtifactNode.Code`.
2. В коллекцию `NavigationItems[]` добавляются группы и пункты с обязательным
   `ItemKind`.
3. Для `Item` задаётся `TargetType` и соответствующее target-specific property.
4. Schema и editor проверяют условную обязательность, запрещённые свойства,
   parent-ссылки и отсутствие циклов дерева. Schema задаёт условные правила;
   editor command service дополнительно проверяет parent group, self-parent и
   перемещение под потомка.
5. Draft проходит обычный Configuration validation и публикацию effective
   configuration.
6. Runtime получает опубликованный `Menu` по `MenuCode`/`ArtifactNode.Code` и
   проецирует его в `RuntimeMenuDefinitionResponse`.

Текущий runtime materializer возвращает `MenuCode`, runtime `ModuleCode`,
локализованный `Title` и плоский набор `RuntimeNavigationItemResponse` с
`ParentCode`. Он также использует `Visible = true`, если значение не задано,
и fallback `Code`, если заголовок отсутствует. Это runtime projection, а не
дополнительная schema `Menu`. Общий frontend navigation adapter затем строит
из `ParentCode` дерево и маршрут для `View` или `Route`; `Action` не становится
кликабельным пунктом в этом adapter, а обработка `ExternalUrl` в нём отдельно
не реализована. ([materializer][materializer]; [runtime contract][runtime-contract];
[runtime navigation adapter][runtime-navigation-adapter])

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `Menu` и `NavigationItem` зарегистрированы как root и child schema types;
- `Menu` содержит `Title`, `Description` и `NavigationItems[]`;
- `NavigationItem` использует discriminator `ItemKind` со значениями `Group`
  и `Item`;
- `TargetType` поддерживает `View`, `ExternalUrl`, `InternalRoute` и `Action`;
- conditional rules для target-specific properties зарегистрированы в schema;
- canonical/baseline builders, editor projector и runtime menu materializer
  используют эту модель;
- runtime contract передаёт target-поля и `TargetObjectTypeCode`, но общий
  frontend adapter сейчас обрабатывает как переходы только `View` и `Route`;
  `Action` и `ExternalUrl` требуют отдельного поведения потребителя;
- интеграционные тесты проверяют локализацию assembled menu, parent hierarchy,
  target view и отображение меню в Explorer/Artifact Editor.

### Дополнительные подтверждённые изменения

Они не меняют schema-контракт `Menu`, но уточняют его потребителей и поставку
baseline:

- сборка одного общего меню из baseline нескольких модулей зафиксирована как
  реализованная (`MenuCode`, owner-aware origin и assembled runtime tree);
- frontend строит из `ParentCode` рекурсивное дерево навигации, а не только
  плоские группы и пункты;
- для `IconCode` добавлен frontend-каталог смысловых кодов иконок;
- добавлен baseline навигации demo-модуля, а baseline General Master Data
  получил `IconCode`.

Поэтому это уже учтённые факты актуального исходного кода, а не будущая
доработка. Повторно переписывать schema-таблицы `Menu` и `NavigationItem` из-за
этого не требуется; уточняется только реализация runtime/frontend-потребителя.

### За пределами текущего schema-контракта `Menu`

- расширения shell/application selection и правила поставки меню для разных
  приложений;
- frontend icon registry, route registry и поведение, зависящее от рендерера; текущий
  общий adapter не задаёт общего поведения для `ExternalUrl` и запуска
  `Action`;
- permission policy, dynamic visibility и authorization runtime;
- полная семантика `ResolutionMode` и `ResolutionSource` для всех приложений;
- гарантии runtime routing, action execution и workflow integration;
- отдельные contribution descriptors и правила конфликтов между владельцами
  baseline.

Это границы schema-документа, а не утверждение, что перечисленные механизмы
никогда не понадобятся. Открытые решения и вопросы владельцев ведутся в
[трассировке Configuration][traceability]; этот файл не заменяет документы
фронтенд-платформа, Object Runtime, Tenant Security или приложений.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Меню | Menu | Корневой конфигурационный артефакт дерева навигации |
| Пункт навигации | NavigationItem | Вложенный узел коллекции `Menu.NavigationItems[]` |
| Группа меню | Group | `NavigationItem` с `ItemKind = Group`, структурный контейнер без target |
| Пункт перехода | Item | `NavigationItem` с `ItemKind = Item`, имеющий target |
| Тип цели | Target type / `TargetType` | Выбор между `View`, `ExternalUrl`, `InternalRoute` и `Action` |
| Родительская группа | Parent group / `ParentCode` | Ссылка на группу-родителя в том же меню |
| Код представления | Target view code / `TargetViewCode` | Ссылка на целевой `View` |
| Режим разрешения | Resolution mode / `ResolutionMode` | Режим разрешения объекта или контекста для `View` |
| Внутренний маршрут | Internal route / `Route` | Строка маршрута, обрабатываемая приложением |
| Внешний адрес | External URL / `Url` | Строка адреса внешнего перехода |
| Код действия | Action code / `ActionCode` | Ссылка на `Action`, которую потребитель может запустить |

Общие термины Configuration ведутся в [тематическом глоссарии][configuration-terms].
Локальная таблица выше нужна для чтения этого документа и не является
отдельным источником истины.

## 12. Источники в коде и тестах

- [schema `Menu` и `NavigationItem`][schemas];
- [коды типов артефактов][artifact-codes], [коды свойств][property-codes] и
  [коды коллекций][collection-codes];
- [источники значений][source-codes], [статические каталоги вариантов][options]
  и [статические коды][static-codes];
- [canonical builders][builder];
- [editor projector][editor] и [validator][validator];
- [runtime materializer][materializer] и [runtime contract][runtime-contract];
- [интеграционные тесты навигации][tests].

Исторические предложения о Navigation используются только для трассировки и
не заменяют подтверждение текущим кодом. ([трассировка][traceability])

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
