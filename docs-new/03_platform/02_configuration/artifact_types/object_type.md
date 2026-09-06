---
id: DOC-03-02-AT-OBJECT-TYPE
title: 'Тип конфигурационного артефакта — ObjectType'
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
created_at: 2026-08-25 17:20
created_by: '@axelprosoft'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Тип конфигурационного артефакта — ObjectType

[project]: ../../../../src/Platform/DMP.Platform.Configuration/
[contracts]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/
[schemas]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ObjectTypeArtifactSchemas.cs
[schema]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ObjectTypeArtifactSchemas.cs
[artifact-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationArtifactTypeCodes.cs
[property-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaPropertyCodes.cs
[collection-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaCollectionCodes.cs
[artifact-document]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/ArtifactDocument.cs
[artifact-node]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/ArtifactNode.cs
[artifact-value]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/ArtifactValue.cs
[static-options]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactStaticOptionCatalog.cs
[static-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[static-value-codes]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/ConfigurationStaticValueCodes.cs
[value-source-codes]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceCodes.cs
[value-source-definition]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ArtifactValueSourceDefinition.cs
[metadata-catalog]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/ConfigurationSchemaMetadataCatalog.cs
[editor-contracts]: ../../../../src/Frontend/packages/contracts/src/configuration.ts
[editor-field-response]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/Responses/ConfigurationArtifactEditorFieldResponse.cs
[editor-node-response]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/Responses/ConfigurationArtifactEditorNodeResponse.cs
[baseline-entry]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/ConfigurationBaselineArtifactEntry.cs
[baseline-property]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/ConfigurationBaselineArtifactProperty.cs
[node-builder]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/Authoring/ObjectTypeNodeBuilder.cs
[effective-resolver]: ../../../../src/Platform/DMP.Platform.Configuration/Domain/Services/ObjectTypeEffectiveModelResolver.cs
[baseline-builder]: ../../../../src/Platform/DMP.Platform.Contracts/Configuration/BaselinePackages/Authoring/ObjectTypeBaselineBuilder.cs
[schema-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ArtifactSchemaRegistryIntegrationTests.cs
[builder-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ObjectTypeNodeBuilderIntegrationTests.cs
[effective-tests]: ../../../../tests/DMP.Platform.IntegrationTests/Configuration/ObjectTypeEffectiveModelResolverIntegrationTests.cs
[runtime-service]: ../../../../src/Platform/DMP.Platform.Runtime/Application/Services/Core/ApplicationRuntimeService.cs
[configuration-terms]: ../../../11_glossary/configuration_terms.md
[configuration-architecture]: ../02_architecture.md
[configuration-contracts]: ../03_contracts.md

## 1. Назначение

Коротко: этот документ отвечает на вопрос, какие настройки типа объекта
Configuration может хранить, проверять и публиковать. Он не описывает сам
бизнес-объект, его таблицу в базе данных или экран приложения.

`ObjectType` — корневой конфигурационный артефакт, описывающий конфигурируемую
модель типа объекта. Он связывает стабильный код типа объекта с его
отображением, полями, локальным поведением полей, ссылками на представления и
привязками правил.

Документ описывает модель `ObjectType` в области `Configuration`. Он не заменяет
исполняемый контракт бизнес-объекта и не описывает внутреннюю реализацию
`Object Runtime`.

## 2. Идентичность и граница

### 2.1. Идентичность

`ObjectType` определяется кодом `ArtifactNode.Code`. Значение
`ArtifactNode.ArtifactTypeCode` для этого узла равно `ObjectType`.

Таким образом, `ObjectTypeCode` — это устоявшееся прикладное название кода типа
объекта, но не отдельное свойство корневого узла в схеме. При ссылке на тип из
другого модуля коду может быть добавлен код модуля: `<ModuleCode>:<Code>`.

### 2.2. Входит в документ

- корневой узел `ObjectType`;
- его свойства отображения, наследования, иерархии, управления и представлений
  по умолчанию;
- дочерняя коллекция `Members`;
- дочерняя коллекция `MemberBehaviors`;
- дочерняя коллекция `RuleBindings`;
- правила проверки модели перед публикацией;
- получение итогового набора полей с учётом `BaseObjectTypeCode`.

### 2.3. Не входит в документ

- исполняемый CLR-контракт и хранение бизнес-объекта;
- процесс изменения объекта (`mutation pipeline`), repository, транзакции и
  жизненный цикл объекта;
- самостоятельные артефакты `View`, `Action`, `Workflow`, `Rule`, `Report`,
  `Output`, `ValueSet`, `SystemEnum` и `NumberingRule`;
- общий shell приложений и экраны Studio/Admin;
- выполнение правил, workflow и numbering;
- отдельный `Dataset` или общий контракт чтения данных.

Эти сущности могут быть связаны с `ObjectType` по стабильным кодам, но не
становятся его дочерними узлами только из-за наличия ссылки.

## 3. Место в Configuration

`ObjectType` хранится во внутреннем дереве Configuration (`ArtifactDocument`)
как узел с дочерними узлами. Реестр схем регистрирует его с признаками `Node`,
`ObjectBound`, `HasConditions` и `RuntimeRenderable`. ([schema][schemas])

 Поставка через базовый пакет конфигурации формирует корневую запись `ObjectType`, а затем записи
его `ObjectMember`, `ObjectMemberBehavior` и `ObjectRuleBinding` с привязкой к
корневой записи. ([baseline builder][baseline-builder])

`ObjectTypeNodeBuilder` показывает программный способ собрать такую же
внутреннюю модель при создании и тестировании. ([node builder][node-builder])

### 3.1. Владение по слоям

| Слой | Что фиксируется для `ObjectType` | Владелец |
| --- | --- | --- |
| Структура и схема | `ObjectType`, свойства, `Members`, `MemberBehaviors`, `RuleBindings`, дискриминаторы и правила проверки | Configuration, этот документ |
| Создание и публикация | Baseline, редактор, сохранение, проверка, версии и итоговая конфигурация | Configuration, `03_contracts.md`, `04_runtime.md`, `06_user_experience.md` |
| Выполнение | CLR-контракт, описание типа (descriptor), хранение, изменения объектов, иерархия, общая таблица наследования (TPH) и жизненный цикл | Object Runtime, целевая область `03_object_runtime` |
| Отображение в приложении | Компоненты отображения (рендерер), общие runtime-пакеты, формы и списки приложений | фронтенд-платформа и конкретное приложение |
| Предметный смысл | Какие конкретные типы объектов и поля объявляет модуль | Документация прикладного модуля |

Внутреннее представление Configuration задаёт форму артефакта, но не выполняет
операции над бизнес-объектами и не рисует интерфейс. Если текущая реализация
использует временный реестр интерфейса или локальный поставщик данных runtime,
это фиксируется как ограничение реализации. Владелец конфигурационного
`ObjectType` при этом остаётся Configuration.

### 3.2. Контекст поставки и хранения (не свойства `ObjectType`)

Этот подраздел нужен для различения свойств артефакта и технических ключей. Его
можно пропустить при первом чтении таблицы свойств.

Узел `ArtifactNode` содержит тип артефакта и его код, но не содержит
`ModuleCode`, `VersionId` или технический идентификатор записи БД. Сведения о
модуле, версии и хранении добавляются Configuration при импорте поставки и
сохранении. Общая модель описана в
[архитектуре Configuration][configuration-architecture] и
[контрактах Configuration][configuration-contracts].

| Понятие | Что означает для `ObjectType` | Где находится |
| --- | --- | --- |
| Код артефакта | `ArtifactNode.ArtifactTypeCode = ObjectType` и `ArtifactNode.Code = ObjectTypeCode`. | Внутреннее представление и этот документ |
| `ModuleCode` | Код модуля, который поставляет артефакт. Это не свойство `ObjectType`. | `ConfigurationEntry.ModuleCode`, `ConfigurationBaselinePackageIdentity.ModuleCode` |
| `OriginKey` | Стабильный ключ записи для импорта, переопределения, переноса и объединения. | `ConfigurationEntry.OriginKey` |
| Ключ записи версии | Пара `(VersionId, OriginKey)`, по которой запись находится внутри версии. | `ConfigurationEntry` и индекс Configuration |
| Технический ключ | `ConfigurationEntry.Id`; нужен базе данных и не заменяет код артефакта. | Модель хранения Configuration |
| Ключ свойства | Пара `(EntryId, PropertyCode)`; локализация дополнительно имеет язык. | `ConfigurationProperty` и `PropertyLocalization` |

При сохранении `ObjectType` с учётом модуля `OriginKey` получает вид
`ObjectType|<ModuleCode>|<ObjectTypeCode>`.
Дочерние узлы получают ключ на основе ключа родителя, типа дочернего узла и его
кода. Формат ключа не нужно повторять в документации каждого дочернего типа.

Запись вида `<ModuleCode>:<Code>` — это ссылка на код другого модуля, а не ключ
хранения. Она используется только там, где это разрешено контрактом.

## 4. Структура и схема `ObjectType`

### 4.1. Состав схемы

Ниже приведена схема в компактном виде: она показывает состав узлов и
коллекций, но не заменяет таблицы свойств.

`ObjectType` состоит из корневого узла и трёх дочерних коллекций. Ниже
технические имена приведены ровно так, как они называются в коде и schema:

| Что это | Техническое имя в коде | Русский смысл | Где описано подробно |
| --- | --- | --- | --- |
| Идентичность корневого узла | `ArtifactNode.ArtifactTypeCode` + `ArtifactNode.Code` | Тип узла и его уникальный код, например `ObjectType` + `Order`. Это не свойства. | Раздел 2.1 |
| Свойства корневого узла | `ArtifactNode.Properties`, проверенные схемой `ObjectType` | Настройки типа: `Title`, наследование, иерархия, удаление, представления по умолчанию. | Раздел 4.2 |
| Коллекция полей | `Members` → `ObjectMember` | Поля типа объекта: обычные значения, ссылки и коллекции. | Раздел 4.3 |
| Коллекция поведений полей | `MemberBehaviors` → `ObjectMemberBehavior` | Локальные реакции на загрузку или изменение поля. | Раздел 4.4 |
| Коллекция привязок правил | `RuleBindings` → `ObjectRuleBinding` | Связи типа объекта или его поля с `Rule`. | Раздел 4.5 |

Схема в компактном виде:

```text
ObjectType (ArtifactNode)
├── identity: ArtifactTypeCode = ObjectType; Code = Order
├── properties: { Title, IsAbstract, DeleteCapability, ... }
└── child collections:
    ├── Members[0..n] → ObjectMember
    ├── MemberBehaviors[0..n] → ObjectMemberBehavior
    └── RuleBindings[0..n] → ObjectRuleBinding
```

Здесь `properties` обозначает словарь `ArtifactNode.Properties`, а `Members`,
`MemberBehaviors` и `RuleBindings` — реальные дочерние коллекции
`ArtifactNode.Children`. Русские названия в таблице объясняют их назначение, но
не заменяют технические имена.

Дальше в документе слово «свойство» означает только настройку, которую схема
разрешает сохранить у узла. Все остальные поля тоже должны иметь владельца и
описание, но в другом списке или документе.

Например, условный тип `Order` может иметь код `Order`, свойство `Title` со
значением «Заказ» и дочерние поля `Number` и `Customer`. Код `Order` говорит,
какой это тип; `Title` задаёт его название; `Number` и `Customer` описываются
как дочерние `ObjectMember`. Это разные части одной конфигурации.

Не следует считать свойствами всё, что встречается в техническом контракте:
`OriginKey` нужен для импорта и хранения, а `Options`, `IsReadOnly` и список
доступных операций нужны ответу редактора. Эти значения могут быть рассчитаны
сервером из сохранённых свойств, прав пользователя и состояния версии; в
таблицах свойств `ObjectType` их нет.

### 4.2. Свойства корневого узла

Ниже приведён полный каталог свойств, зарегистрированных в текущей схеме
`ObjectType`. `Required` означает обязательность свойства в схеме; это
не всегда означает обязательность пользовательского ввода в UI. Источники
значений и правила обновления описаны в разделе 5. Смысл значений и их
влияние указаны в этой же строке таблицы, а не в отдельном повторном каталоге.

Пометка «статический каталог вариантов» означает, что допустимые коды заданы в
[каталоге статических вариантов][static-options]. Это не список, который может
произвольно расширить клиентский редактор.

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
| `Title` | Название типа объекта | `LocalizedText` | Нет | Локализованные значения | Название для редактора и отображения; код типа не меняет | Default |
| `BaseObjectTypeCode` | Базовый тип объекта | `ReferenceCode` | Нет | Каталог `ObjectTypes` | Задаёт базовый тип; его свойства и правила участвуют в effective-наборе производного типа | `InheritedOnly` |
| `IsAbstract` | Абстрактный тип | `Bool` | Нет / `false` | `true`, `false` | `true` запрещает прямое создание экземпляров; `false` такого запрета не задаёт | `InheritedOnly` |
| `IsReferenceOnly` | Только для ссылок и lookup | `Bool` | Нет / `false` | `true`, `false` | `true` помечает тип как предназначенный для ссылок и lookup; `false` такого ограничения не задаёт | `InheritedOnly` |
| `DiscriminatorMemberCode` | Поле-дискриминатор | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле, по которому выбирается конкретный тип объекта | `InheritedOnly` |
| `DiscriminatorValue` | Значение дискриминатора | `String` | Нет | Строковый код конкретного типа; формат строки схема не задаёт | Значение сопоставляется с `DiscriminatorMemberCode`; схема не определяет дополнительную обработку | `InheritedOnly` |
| `HierarchyParentMemberCode` | Поле непосредственного родителя | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле связи с непосредственным родителем | `InheritedOnly` |
| `HierarchyRootMemberCode` | Поле корня дерева | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле, в котором хранится корень иерархии | `InheritedOnly` |
| `HierarchyLevelMemberCode` | Поле уровня | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле уровня узла в иерархии | `InheritedOnly` |
| `HierarchyPathMemberCode` | Поле пути дерева | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле пути узла в иерархии | `InheritedOnly` |
| `HierarchyHasChildrenMemberCode` | Поле наличия дочерних узлов | `ReferenceCode` | Нет | `ObjectType.Members` | Указывает поле признака наличия дочерних узлов | `InheritedOnly` |
| `HierarchyRootBehavior` | Политика корней | `Enum` | Нет / `MultipleRoots` | `MultipleRoots`, `SingleRoot`; статический каталог вариантов | `MultipleRoots` разрешает несколько корней; `SingleRoot` ограничивает иерархию одним корнем | `InheritedOnly` |
| `HierarchyPreventCycles` | Запрет циклов | `Bool` | Нет / `true` | `true`, `false` | `true` требует отклонять циклические связи; `false` не задаёт такого запрета схемой | `InheritedOnly` |
| `DefaultListViewCode` | Представление списка по умолчанию | `ReferenceCode` | Нет | Каталог `Views`, `ViewType = ObjectList` | Выбирает представление списка по умолчанию | Schema filter |
| `DefaultDetailViewCode` | Представление карточки по умолчанию | `ReferenceCode` | Нет | Каталог `Views`, `ViewType = ObjectForm` | Выбирает представление карточки по умолчанию | Schema filter |
| `DefaultLookupViewCode` | Представление выбора по умолчанию | `ReferenceCode` | Нет | Каталог `Views`, `ViewType = LookupListView` | Выбирает представление lookup по умолчанию | Schema filter |
| `AdministrationSectionDefault` | Секция администрирования по умолчанию | `Enum` | Нет | `Auto`, `Hidden`; статический каталог вариантов | `Auto` оставляет выбор секции потребителю; `Hidden` указывает скрыть секцию | Default |
| `ManagementKind` | Вид управляемого объекта | `Enum` | Нет | `None`, `ManagedObject`, `ManagedCatalogObject`; статический каталог вариантов | `None` — специальный вид не задан; остальные значения помечают вид управляемого объекта; полный сценарий схемой не задан | Default |
| `DeleteCapability` | Возможность удаления корня | `Enum` | Нет | `Disabled`, `SoftDelete`, `HardDelete`; статический каталог вариантов | `Disabled` запрещает удаление; `SoftDelete` задаёт мягкое удаление; `HardDelete` — физическое удаление | Default |
| `SupportsDirectArchive` | Прямое архивирование | `Bool` | Нет | `true`, `false` | `true` разрешает прямое архивирование; `false` не разрешает его этим признаком | Default |
| `SupportsDirectRestore` | Прямое восстановление | `Bool` | Нет | `true`, `false` | `true` разрешает прямое восстановление; `false` не разрешает его этим признаком | Default |
| `CreateFromExistingUsage` | Участие в создании из существующего | `Enum` | Нет / `None` | `None`, `Root`, `OwnedPart`, `Both`; статический каталог вариантов | `None` — не участвует; `Root` — корневой объект; `OwnedPart` — принадлежащая часть; `Both` — оба уровня | `InheritedOnly` |

`Title` влияет на отображение, но не является кодом типа. Исполняемая семантика
поля, обработчики, хранение и жизненный цикл объекта не задаются этими
свойствами.
Значения перечислений берутся из [каталога статических опций][static-options] и
[технических code constants][static-codes], а не из произвольного списка,
написанного в UI.

### 4.3. Вложенный тип `ObjectMember`

`ObjectMember` идентифицируется своим `ArtifactNode.Code` внутри `Members`.
Обязательное дискриминирующее свойство `MemberType` принимает значения:
`Scalar`, `Reference` или `Collection`.

Полный каталог свойств:

В таблице сохраняются технические коды из схемы. `Scalar`, `Reference` и
`Collection` — три вида поля; пояснения общих условий их применения приведены в
разделе 6. Значения `InheritedOnly` и `OverrideAllowed` читаются так же, как
в таблице корневых свойств выше.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `MemberType` | Вид поля | `Enum` | Да / `Scalar` | `Scalar`, `Reference`, `Collection`; статический каталог вариантов | `Scalar` — простое значение; `Reference` — ссылка; `Collection` — коллекция дочерних объектов | Дискриминатор, `InheritedOnly` |
| `Title` | Название поля | `LocalizedText` | Нет | Локализованные значения | Подпись поля в редакторе и отображении; тип и правила поля не меняет | Все виды |
| `Description` | Описание поля | `LocalizedText` | Нет | Локализованные значения | Пояснение для пользователя; поведение поля не меняет | Все виды |
| `DataType` | Тип значения | `Enum` | Нет / `String` | `String`, `Number`, `Boolean`, `Enum`, `Date`, `DateTime`, `Guid`; статический каталог типов данных | `String` — текст; `Number` — число; `Boolean` — логическое значение; `Enum` — значение перечисления; `Date` — дата; `DateTime` — дата и время; `Guid` — идентификатор GUID | `Scalar`, `Reference`; `InheritedOnly` |
| `Required` | Обязательность | `Bool` | Нет / `false` | `true`, `false` | `true` требует значение; `false` не устанавливает обязательность | `Scalar`; `InheritedOnly` |
| `DefaultEditorType` | Редактор по умолчанию | `Enum` | Нет | `TextBox`, `TextArea`, `Select`, `MultiSelect`, `Checkbox`, `Switch`, `Number`, `Date`, `DateTime`, `Lookup`, `Collection`, `Badge`; статический каталог вариантов | `TextBox` — однострочный текст; `TextArea` — многострочный текст; `Select` — один вариант; `MultiSelect` — несколько вариантов; `Checkbox`/`Switch` — логическое значение; `Number` — число; `Date`/`DateTime` — дата или дата со временем; `Lookup` — ссылка; `Collection` — коллекция; `Badge` — компактное отображение значения | Только подсказка редактору; значение проверяет сервер |
| `ValueSetCode` | Набор значений | `ReferenceCode` | Нет | Каталог `ValueSets` | Ограничивает допустимые значения ссылочным набором | Только `Scalar`; взаимоисключим с `SystemEnumCode`; `InheritedOnly` |
| `SystemEnumCode` | Системное перечисление | `ReferenceCode` | Нет | Каталог `SystemEnums` | Ограничивает допустимые значения системным перечислением | Только `Scalar`; взаимоисключимо с `ValueSetCode`; `InheritedOnly` |
| `ReferenceSourceCode` | Источник ссылочных значений | `ReferenceCode` | Для `Reference` — да | Код источника ссылок; каталог не задан schema | Указывает владельца и поставщика ссылочных значений; дальнейший формат задаёт владелец источника | Запрещено для `Scalar`; `InheritedOnly` |
| `ReferenceKeyPath` | Путь ключа ссылки | `String` | Нет | Строковый путь; синтаксис пути схема не задаёт | Указывает, где в ссылочном значении находится ключ; схема не задаёт разбор пути | Только `Reference`; `InheritedOnly` |
| `SemanticRole` | Платформенная семантика поля | `Enum` | Нет | `TenantScope`, `SiteScope`, `TimeAmount`; статический каталог вариантов | `TenantScope` — область tenant; `SiteScope` — область site; `TimeAmount` — время или длительность; правила применения задаёт потребитель | Зависит от `MemberType`/`DataType`; `InheritedOnly` |
| `TimeAmountFormatOverride` | Формат времени | `Enum` | Нет | `Standard`, `Industrial`; статический каталог вариантов | `Standard` — стандартное представление; `Industrial` — промышленное представление | Только `Scalar`; `OverrideAllowed` |
| `TimeAmountValueType` | Тип числового времени | `Enum` | Нет | `Decimal`; статический каталог вариантов | `Decimal` задаёт числовую основу значения времени | Только `Scalar`; `InheritedOnly` |
| `TargetObjectTypeCode` | Целевой тип объекта | `ReferenceCode` | Для `Collection` — да | Каталог `ObjectTypes` | Задаёт тип элементов коллекции | Запрещено для `Scalar` и `Reference`; `InheritedOnly` |
| `ParentLinkMemberCode` | Поле связи с владельцем | `ReferenceCode` | Нет | `ObjectType.Members` целевого типа | Указывает поле, связывающее элемент коллекции с владельцем | Только `Collection`; зависит от `TargetObjectTypeCode`; `InheritedOnly` |
| `Aggregation` | Семантика коллекции | `Enum` | Для `Collection` — да | `Aggregate`, `Association`; статический каталог вариантов | `Aggregate` — элемент является частью владельца; `Association` — только связан с владельцем | Только `Collection`; `InheritedOnly` |
| `SaveMode` | Способ сохранения коллекции | `Enum` | Для `Collection` — да | `WithOwner`, `Separate`; статический каталог вариантов | `WithOwner` — сохраняется вместе с владельцем; `Separate` — отдельно от владельца | Только `Collection`; `InheritedOnly` |
| `DeleteBehavior` | Удаление элементов коллекции | `Enum` | Нет | `Cascade`, `Restrict`; статический каталог вариантов | `Cascade` распространяет удаление владельца; `Restrict` запрещает удаление при наличии элементов | Только `Collection`; `InheritedOnly` |
| `MissingItemBehavior` | Поведение отсутствующего элемента | `Enum` | Нет | `Delete`, `Ignore`; статический каталог вариантов | `Delete` удаляет отсутствующий элемент; `Ignore` не удаляет его автоматически | Только `Collection`; `InheritedOnly` |
| `CreateFromExisting` | Перенос значения при копировании | `Enum` | Нет / `Default` | `Default`, `Exclude`, `Include`; статический каталог вариантов | `Default` — обычная политика; `Exclude` — не переносить; `Include` — переносить | Для `Scalar`, `Reference`, `Collection`; `InheritedOnly` |
| `NumberingEnabled` | Использование нумерации | `Bool` | Нет / `false` | `true`, `false` | `true` разрешает использование нумерации; `false` её не включает | Только `Scalar`; `InheritedOnly` |
| `NumberingSupportedMoments` | Моменты нумерации | `Json` | Нет | JSON-список; допустимые коды моментов и формат схема не задаёт | Перечисляет моменты, в которых потребитель может применять нумерацию | Только `Scalar`; `InheritedOnly` |
| `NumberingAllowedSourcesJson` | Разрешённые источники нумерации | `Json` | Нет | JSON-структура; формат и допустимые источники схема не задаёт | Ограничивает источники, которые может использовать numbering; формат задаёт владелец numbering | Только `Scalar`; `InheritedOnly` |
| `NumberingManualInputPolicy` | Ручной ввод номера | `Enum` | Нет / `Forbidden` | `Forbidden`, `Allowed`, `AdminOnly`; статический каталог вариантов | `Forbidden` — запрещён; `Allowed` — разрешён; `AdminOnly` — только администратору | Только `Scalar`; `InheritedOnly` |
| `NumberingMissingRulePolicy` | Нет правила нумерации | `Enum` | Нет / `Block` | `Block`, `AllowManual`; статический каталог вариантов | `Block` блокирует операцию; `AllowManual` разрешает ручной ввод при отсутствии правила | Только `Scalar`; `InheritedOnly` |
| `FlagsNullBehavior` | Обработка `null` для flags enum | `Enum` | Нет / `Default` | `Default`, `All`; статический каталог вариантов | `Default` — обычная обработка; `All` трактует `null` как все flags, если это подтверждено runtime | Только `Scalar`; `InheritedOnly` |

`ObjectMember` описывает настройку поля. Он не создаёт автоматически поле в
классе и не меняет тип поля, способ хранения или обработчик выполнения. Для
`Reference` и `Collection` ссылки должны указывать на допустимые источники и
типы объектов; правила применимости зависят от `MemberType`.

### 4.4. Вложенный тип `ObjectMemberBehavior`

`ObjectMemberBehavior` задаёт локальную реакцию, связанную с полями данного
`ObjectType`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название поведения | `LocalizedText` | Нет | Локализованные значения | Подпись поведения для редактора; выполнение не меняет | Все экземпляры |
| `Trigger` | Момент срабатывания | `Enum` | Да | `OnLoad`, `OnFieldChanged`, `OnValueChanged`; статический каталог вариантов | `OnLoad` — при загрузке; `OnFieldChanged` — при изменении поля; `OnValueChanged` — при изменении значения | Все экземпляры |
| `SourceMemberCodes` | Исходные поля | `Json` | Нет | JSON-массив кодов полей | Задаёт поля, от которых зависит поведение; формат массива schema не задаёт | Все экземпляры |
| `TargetMemberCode` | Целевое поле | `ReferenceCode` | Да | `ObjectType.Members` | Указывает поле, к которому применяется эффект | Все экземпляры |
| `ConditionExpression` | Условие | `String` | Нет | Выражение условия; язык и источники контекста не заданы schema | Ограничивает применение поведения условием; язык выражения schema не задаёт | Если условие задано |
| `EffectKind` | Вид эффекта | `Enum` | Да | `Warn`, `SetValue`, `SetRequired`, `SetVisible`, `SetEnabled`, `SetReadOnly`, `SetReadonly`, `SetEditable`, `ReloadOptions`, `SelectValue`, `CalculateValue`, `AllowedValues`, `Lock`; статический каталог вариантов | `Warn` — предупреждение; `SetValue` — установить значение; `SetRequired` — изменить обязательность; `SetVisible` — изменить видимость; `SetEnabled` — изменить доступность; `SetReadOnly`/`SetReadonly` — только чтение; `SetEditable` — разрешить редактирование; `ReloadOptions` — обновить варианты; `SelectValue` — выбрать значение; `CalculateValue` — рассчитать значение; `AllowedValues` — ограничить варианты; `Lock` — заблокировать изменение | Все экземпляры; точный контракт эффекта принадлежит редактору |
| `EffectValue` | Значение эффекта | `Json` | Нет | JSON-значение; формат зависит от `EffectKind` и не задан schema | Параметр эффекта; его форма определяется выбранным `EffectKind` | Если выбранный `EffectKind` требует параметр |
| `Order` | Порядок поведения | `Number` | Нет | Число; используется для изменения порядка | Определяет порядок применения поведений | Все экземпляры |

Этот узел описывает локальную реакцию редактора или другого потребителя
конфигурации. Он не является общей оболочкой приложений и не заменяет
серверную бизнес-логику.

### 4.5. Вложенный тип `ObjectRuleBinding`

`ObjectRuleBinding` связывает `Rule` с `ObjectType` или с конкретным полем.
Обязательны `RuleCode` и `TargetLevel`.

| Технический код | Русский смысл | Тип значения | Обязательность / значение по умолчанию | Допустимые значения или источник | Смысл значения и влияние | Когда используется / переопределение |
| --- | --- | --- | --- | --- | --- | --- |
| `Title` | Название привязки | `LocalizedText` | Нет | Локализованные значения | Подпись привязки; применение правила не меняет | Все экземпляры |
| `RuleCode` | Код правила | `ReferenceCode` | Да | Корневой артефакт `Rule` | Указывает правило, которое должно применяться | Все экземпляры |
| `TargetLevel` | Уровень применения | `Enum` | Да | `ObjectType`, `Member`; статический каталог вариантов | `ObjectType` — правило относится ко всему типу; `Member` — к одному полю | Все экземпляры |
| `TargetCode` | Код целевого поля | `ReferenceCode` | Условно | Обязателен для `TargetLevel = Member`, запрещён для `ObjectType`; источник `ObjectType.Members` | Указывает поле, к которому относится правило, если уровень — `Member` | Только при `TargetLevel = Member` |

Само правило и механизм его проверки описываются в документе
`artifact_types/rule.md`; здесь фиксируется только связь правила с типом объекта
или его полем.

### 4.6. Связи и структурные правила

Поля не складываются в одну общую таблицу. Они распределяются по уровням
схемы:

| Что фиксируем | Где это описано | Пример |
| --- | --- | --- |
| Свойства корневого `ObjectType` | Раздел 4.2 этого документа | `Title`, `IsAbstract`, `DeleteCapability` |
| Свойства `ObjectMember` | Раздел 4.3 этого документа | `MemberType`, `DataType`, `Required` |
| Свойства `ObjectMemberBehavior` | Раздел 4.4 этого документа | `Trigger`, `EffectKind`, `Order` |
| Свойства `ObjectRuleBinding` | Раздел 4.5 этого документа | `RuleCode`, `TargetLevel`, `TargetCode` |
| Код и ключи хранения | Разделы 2.1 и 3.2 этого документа и архитектура Configuration | `Code`, `OriginKey`, `ModuleCode`, `EntryId` |
| Поля ответа редактора | `03_contracts.md` и `06_user_experience.md` | `Options`, `IsReadOnly`, доступные операции |
| Рассчитанные значения выполнения | `04_runtime.md` и Object Runtime | итоговый список полей, рассчитанная иерархия |

Отсутствие поля в таблице свойств не означает, что оно забыто. Это означает,
что поле принадлежит другому уровню и описывается у его владельца. Например,
`Options` нужно документировать в контракте редактора, но нельзя объявлять
сохраняемым свойством `ObjectType`.

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

## 6. Ограничения и зависимости

Сервер проверяет следующие ограничения и зависимости:

- `ValueSetCode` и `SystemEnumCode` нельзя задавать одновременно;
- `ParentLinkMemberCode` можно задавать только вместе с `TargetObjectTypeCode`;
- `TargetCode` обязателен при `TargetLevel = Member` и запрещён при
  `TargetLevel = ObjectType`.

Локальные условия применения указаны в строке соответствующего свойства.

## 7. Операции над структурой

| Уровень | Создание | Изменение | Удаление | Изменение порядка | Ограничение и подтверждение |
| --- | --- | --- | --- | --- | --- |
| Корневой `ObjectType` | Общий контракт редактирования Configuration | Обновление свойств | Общая команда записи, если разрешена политикой | Нет | API и права описаны в `03_contracts.md`; отдельной CRUD-матрицы для корня в схеме нет |
| `Members` | Да | Через общий контракт записи | Не объявлено схемой | Нет | Разрешено создание дочернего узла |
| `MemberBehaviors` | Да | Через общий контракт записи | Не объявлено схемой | Да | Разрешены создание и изменение порядка; порядок хранится в `Order` |
| `RuleBindings` | Не объявлено схемой | Да | Не объявлено схемой | Нет | Разрешено изменение дочернего узла |
| Отдельное свойство | Общая команда записи | Да с учётом правил изменения и наследования | Явный сброс или `null` по контракту записи | Нет | Точный HTTP/C# контракт описан в `03_contracts.md` |

Отсутствие операции в схеме не означает, что её невозможно реализовать любым
API. Это означает только, что текущая схема не объявляет её структурной
операцией. Фактические команда, право и действие редактора проверяются по
контрактам Configuration и состоянию редактора.

## 8. Наследование и рассчитанный результат

`BaseObjectTypeCode` задаёт цепочку наследования типов объектов.
`ObjectTypeEffectiveModelResolver` проходит цепочку от базового типа к
производному и формирует итоговый список `EffectiveMembers`:

1. сначала поля базового типа;
2. затем поля производного типа;
3. повторное объявление одного `MemberCode` в цепочке считается ошибкой;
4. цикл цепочки наследования считается ошибкой;
5. отсутствующий базовый тип считается ошибкой.

Итоговый список не заменяет опубликованную версию и не является новым видом
конфигурационного артефакта. Это рассчитанный результат для чтения и выполнения.
([resolver][effective-resolver]; [tests][effective-tests])

## 9. Создание, проверка и публикация

 Для создания используются служебные сборщики внутренней модели и базовый пакет конфигурации.
Перед публикацией реестр схем и проверяющий код проверяют структуру узлов,
обязательные свойства, условия применимости, ссылки и цепочку наследования.

Подтверждённые текущим кодом проверки:

- в реестре зарегистрированы схемы всех четырёх узлов семейства `ObjectType`;
- `MemberType` является признаком, который определяет вид `ObjectMember`;
- `ObjectRuleBinding.TargetCode` обязателен для `Member` и запрещён для
  `ObjectType`;
- `MemberBehaviors` имеют поле порядка;
- расчёт итогового набора возвращает ошибки при цикле наследования, отсутствии
  базового типа и повторного кода поля.

Публикация, цепочка версий, эффективная конфигурация и кэш описываются в
`04_runtime.md` области `Configuration`; данный документ фиксирует только
модель артефакта и проверки этой модели.

## 10. Статус и границы документа

### Подтверждено в текущем MVP

- `ObjectType`, `ObjectMember`, `ObjectMemberBehavior` и `ObjectRuleBinding`
  зарегистрированы в schema registry;
- канонические builders и baseline builders поддерживают семейство;
- inheritance chain и effective members реализованы и покрыты интеграционными
  тестами;
- `View` подключается к `ObjectType` ссылками default views, но остаётся
 отдельным root-артефактом.
- Object Runtime поддерживает lookup по abstract
  `ObjectType` с поиском concrete descriptors и возвращает `ModuleCode`/
  `ObjectTypeCode` для concrete reference identity. Это подтверждает runtime
  потребление object metadata, но не переносит lookup/query contract в schema
  `ObjectType` и не закрывает вопрос владельца `Dataset / Read Query Capability`.
  ([runtime-service][runtime-service])

### За пределами текущего schema-контракта `ObjectType`

- полный договор между `ObjectType` и Object Runtime для всех типов полей;
- окончательная матрица иерархии, TPH и `reference-only` для каждого прикладного
  модуля;
- общий `Dataset / Read Query Capability`;
- полный production-контракт numbering для всех сценариев.

Перечисленное не объявляется отсутствующим. Это только граница данного
документа: подробности должны появиться в документах владельцев после сверки с
кодом и отдельного архитектурного решения.

Сопоставление со старым документом не является частью контракта `ObjectType`.
Оно ведётся в [трассировке Configuration](../90_traceability.md), где отдельно
указаны перенесённые сведения, расхождения и решения. В этот документ попадают
только свойства и правила текущей схемы.

## 11. Термины

| Русский термин | English / code | Значение |
| --- | --- | --- |
| Свойство артефакта | Artifact property | Настройка, сохранённая у узла и разрешённая схемой артефакта. |
| Схема артефакта | Artifact schema | Список разрешённых свойств, дочерних узлов и правил их проверки. |
| Код артефакта | `ArtifactNode.Code` | Стабильное имя узла; для `ObjectType` это код типа объекта. |
| Тип объекта | ObjectType | Корневой конфигурационный артефакт типа объекта. |
| Поле объекта | ObjectMember | Дочерний узел `ObjectType`, описывающий поле. |
| Поведение поля | ObjectMemberBehavior | Дочерний узел локальной реакции на событие или изменение поля. |
| Привязка правила | ObjectRuleBinding | Дочерний узел, связывающий `Rule` с объектом или полем. |
| Итоговый набор полей | EffectiveMembers | Рассчитанный набор полей после применения цепочки базовых типов. |

Общие термины Configuration ведутся в [тематическом глоссарии][configuration-terms].
Локальная таблица выше нужна для чтения документа и не является отдельным
источником истины.

## 12. Источники в коде и тестах

- [Canonical artifact document][artifact-document]
- [Canonical artifact node][artifact-node]
- [Canonical artifact value][artifact-value]
- [Baseline artifact entry][baseline-entry]
- [Baseline artifact property][baseline-property]
- [Editor field response][editor-field-response]
- [Editor node response][editor-node-response]
- [ObjectType schema][schemas]
- [ObjectType node builder][node-builder]
- [ObjectType baseline builder][baseline-builder]
- [Effective model resolver][effective-resolver]
- [Schema registry tests][schema-tests]
- [Node builder tests][builder-tests]
- [Effective model tests][effective-tests]

## 13. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
