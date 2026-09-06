# Журнал решений - 00 Common

Документ фиксирует принятые проектные решения по базовому прикладному модулю `00_common`.

Это рабочий журнал решений, а не финальный дизайн-документ. Итоговые нормативные документы должны быть собраны из этих решений в более компактной структуре:

```text
../00_module_overview.md
../02_domain_model.md
../05_rules.md
../03_object_runtime_model.md
../90_traceability_pr00.md
```

## Решение 1. Базовая модель Common v1

Статус: принято для Common v1.

### Контекст

В исходных требованиях `DMP ПР00 Общие требования` базовая объектная модель описана через цепочку:

```text
DmpBase
DmpStatusBase : DmpBase
DmpCatalogBase : DmpStatusBase
```

Во втором документе, `DMP ПР01 Общая НСИ`, эти базовые классы используются по-разному:

- часть объектов является справочниками/каталогами и требует `Code` + `Name`;
- часть объектов является строками, присвоениями, контактами или значениями и не требует `Code` + `Name`;
- часть объектов в старых требованиях наследуется от `DmpStatusBase`, но решение о статусности не должно автоматически превращать `Status` в обязательное поле всех объектов Common.

Текущая реализация `DMP.Modules.Common` содержит `BusinessEntity` с полями `Id`, `Code`, `Name`. Этот класс покрывает только каталоговый случай и не должен считаться универсальной базой для всех object types.

### Решение

Для Common v1 принимается разделение базовой технической идентичности и каталоговой бизнес-идентификации.

```text
CommonObject
  Id
  ExternalId?
  CreatedAtUtc
  CreatedBy
  ModifiedAtUtc?
  ModifiedBy?
  IsArchived
  ArchivedAtUtc?
  ArchivedBy?
  ArchiveReason?
  IsDeleted
  DeletedAtUtc?
  DeletedBy?
  DeleteReason?

CommonCatalogObject : CommonObject
  Code
  Name
```

`CommonObject` является базой для прикладных объектов, которым не нужны `Code` и `Name`: строк, значений, присвоений, контактов, вспомогательных прикладных сущностей и других не-каталоговых объектов.

`CommonCatalogObject` является базой для справочников и каталоговых бизнес-объектов, где `Code` и `Name` являются частью бизнес-идентификации.

### Внешний идентификатор

`ExternalId` включается в `CommonObject` как nullable-поле.

Причина: исходные требования и интеграционные сценарии используют внешний идентификатор как стандартный способ сопоставления объекта с внешними системами. При этом поле не должно быть обязательным, потому что не каждый объект обязательно приходит из внешнего источника.

Базовые правила:

- `ExternalId` допускает `null`;
- тип `ExternalId`: `string?`;
- максимальная длина: 100 символов;
- значение нормализуется через `Trim`;
- пустая строка после нормализации трактуется как `null`;
- уникальность `ExternalId` не считается глобальной;
- политика уникальности должна задаваться на уровне object type / tenant / интеграционного источника;
- поиск по `ExternalId` должен рассматриваться как типовой runtime/integration-сценарий;
- если в будущем потребуется несколько внешних источников на один объект, нужно отдельное решение по модели external identities.

Сравнение `ExternalId` не должно навязываться Common v1 глобально как case-sensitive или case-insensitive. Это зависит от конкретной внешней системы, object type и интеграционного сценария.

### Поля аудита объекта

Поля:

```text
CreatedAtUtc
CreatedBy
ModifiedAtUtc?
ModifiedBy?
```

включаются в `CommonObject` как системные поля объекта.

Причина: они нужны не только для compliance-аудита, но и для пользовательских сценариев: списков, карточек, сортировки, фильтрации и вкладки/секции администрирования.

При этом эти поля не заменяют Audit History Platform.

В DMP уже есть отдельная модель audit history:

```text
audit_history.audit_records
AuditRecord
PersistentAuditHistoryWriter
ObjectRuntimeAuditSink
```

Audit History остается неизменяемым журналом значимых операций и изменений. Поля аудита объекта хранят текущее техническое состояние записи: кто и когда создал/изменил объект.

### Архивные поля

Поля:

```text
IsArchived
ArchivedAtUtc?
ArchivedBy?
ArchiveReason?
```

включаются в `CommonObject`.

Причина: архивирование является типовым способом вывести объект из активного использования без потери истории, ссылок и аудита. Наличие этих полей в базовом объекте упрощает списки, фильтры, поиск, API и отчеты.

При этом наличие полей не означает, что каждый тип объекта обязан поддерживать пользовательскую операцию архивирования. Доступность архивирования задается контрактом и правилами конкретного object type.

### Поля мягкого удаления

Поля:

```text
IsDeleted
DeletedAtUtc?
DeletedBy?
DeleteReason?
```

включаются в `CommonObject`.

Причина: удаленные объекты должны системно исключаться из обычных запросов Object Runtime. Этот фильтр не должен вручную повторяться в каждом списке, lookup, отчете или прикладном модуле.

При этом наличие полей не означает, что каждый тип объекта обязан поддерживать пользовательскую операцию удаления. Доступность удаления и способ выполнения задаются контрактом и правилами конкретного object type.

### Представление объекта

`Presentation` не включается в `CommonObject` как обязательное физическое поле.

Представление объекта должно задаваться как вычисляемое или конфигурируемое правило отображения:

```text
DisplayName / правило представления
```

Примеры:

```text
{Code} - {Name}
{Name}
{ExternalId} - {Name}
```

Такое правило должно относиться к Object Runtime / baseline configuration / UI presentation, а не обязательно храниться в каждой таблице.

### Статус

`Status` не включается в базовые классы Common v1.

Старая модель `DmpStatusBase` не переносится в код и документацию как обязательный родитель для каталогов.

Статусность объектов рассматривается как отдельная возможность. Для объектов, которым нужен жизненный цикл `Черновик / Опубликован / Архив`, вероятное целевое решение - типовой workflow-шаблон или workflow-возможность, а не поле `Status` в каждом базовом классе.

Для Common v1 типовой workflow-шаблон детально не проектируется.

Но фиксируются рамки будущего решения:

```text
Status / DmpStatusBase:
  не переносятся в CommonObject / CommonCatalogObject

Lifecycle/status:
  реализуется как отдельная возможность object type

Workflow template:
  должен проектироваться отдельным решением Workflow / Object Runtime
```

Требования к будущему workflow-шаблону:

- состояния workflow не должны дублировать `IsArchived` как независимое ручное поле;
- архивное состояние должно помечаться признаком `IsArchiveState`;
- переходы архивирования и восстановления должны использовать стандартные коды Common `Archive` / `Restore`;
- для одного object type нельзя смешивать прямое архивирование runtime action-ом и архивирование через workflow;
- если object type управляется workflow, источник архивного состояния - workflow state, а `IsArchived` является материализованным техническим признаком.

Следствие:

```text
DmpCatalogBase из требований не переносится как CatalogObject : StatusObject.
CommonCatalogObject не содержит Status.
```

### Отношение к текущему BusinessEntity

Текущий `BusinessEntity` в `DMP.Modules.Common` соответствует только части новой модели:

```text
BusinessEntity ~= CommonCatalogObject без ExternalId, полей аудита, архивных полей и полей удаления
```

Для дальнейшей реализации нужно отдельное техническое решение:

1. переименовать/мигрировать `BusinessEntity` в `CommonCatalogObject`;
2. добавить отдельный `CommonObject`;
3. обновить base object contracts и baseline configuration;
4. обеспечить совместимость с уже существующими object runtime примерами.

До этого `BusinessEntity` нельзя описывать как финальную универсальную базу Common.

### Что фиксируется для дизайн-документов

Документы `00_common` должны исходить из следующей целевой модели:

```text
CommonObject
  базовая техническая идентичность, внешний идентификатор, поля аудита, архивные поля и поля удаления

CommonCatalogObject
  каталоговая бизнес-идентификация Code + Name поверх CommonObject

Status
  не часть базовых классов Common v1
  выносится в будущее решение по workflow

Presentation
  вычисляемое/конфигурируемое правило отображения
  не обязательное физически хранимое поле
```

## Решение 2. Модель внешней идентичности v1

Статус: принято для Common v1.

### Контекст

В требованиях `ExternalId` описан как внешний идентификатор элемента справочника или бизнес-объекта во внешней системе. В интеграционных сценариях `DMP ПР01 Общая НСИ` он используется как один из стандартных ключей поиска и сопоставления объекта наряду с `Id`, `Code` и `Name`.

При этом для промышленной DMP-модели возможна ситуация, когда один объект сопоставляется сразу с несколькими внешними системами: ERP, PLM, MES, WMS и другими источниками. В такой модели пары `ExternalId` недостаточно без указания источника.

### Рассмотренные варианты

```text
Вариант A
CommonObject
  ExternalId?

Вариант B
CommonObject
  ExternalId?
  ExternalSourceSystemCode?

Вариант C
CommonExternalIdentity
  ObjectTypeCode
  ObjectId
  SourceSystemCode
  ExternalId
```

### Решение

Для Common v1 принимается вариант A:

```text
CommonObject
  ExternalId?
```

`SourceSystemCode` не включается в `CommonObject` v1.

Причина: добавление `SourceSystemCode` рядом с `ExternalId` в базовый объект фиксирует модель "один объект - один основной внешний источник". Это может оказаться слишком узким решением, если позже потребуется несколько внешних идентификаторов для одного объекта.

`ExternalId` в `CommonObject` трактуется как основной внешний идентификатор объекта в текущем интеграционном контексте или в основном внешнем источнике, если такой источник определен модулем/интеграционным сценарием.

После анализа Integration Capability принято дополнительное уточнение: `CommonObject.ExternalId` не является полной моделью интеграционного сопоставления.

`ExternalId` в Common - это необязательный внешний или унаследованный идентификатор, который может быть видимым на карточке объекта, использоваться в прикладном API или присутствовать в старых требованиях к объекту.

Модель сопоставления с конкретными внешними системами принадлежит Integration Foundation / Integration Service:

```text
ExternalObjectMapping
  TenantId
  ExternalSystemCode
  ExternalObjectType
  ExternalObjectId
  InternalModuleCode
  InternalObjectTypeCode
  InternalObjectId
  MappingStatus
  CreatedAtUtc
  LastSeenAtUtc
  LastSyncAtUtc
```

Следствие: `SourceSystemCode` не переносится в `CommonObject`, потому что источник является частью `ExternalObjectMapping`, а один внутренний объект может иметь сопоставления с несколькими внешними системами.

Если конкретный object type не должен показывать внешний идентификатор пользователям или в runtime API, `ExternalId` может быть скрыт на уровне baseline configuration / view policy, но поле остается nullable частью базового Common contract.

### Правила v1

- `ExternalId` является nullable-полем.
- `ExternalId` не обязан быть задан для объектов, созданных только внутри DMP.
- `ExternalId` не является глобально уникальным.
- Отдельная специальная `ExternalIdUniquenessPolicy` в Common v1 не вводится.
- Если конкретному object type нужна уникальность `ExternalId`, она должна выражаться через общий механизм уникальности object type: `UniqueConstraint` / `UniqueIndex` / alternate key.
- Если object type требует строгого сопоставления с внешней системой, это должно быть описано в документации этого object type и в его validation/integration rules.
- `SourceSystemCode` не используется как базовое поле Common v1.

Примеры:

```text
корпоративный справочник:
  UniqueConstraint(ExternalId)

tenant-owned объект:
  UniqueConstraint(TenantId, ExternalId)

объект без строгой внешней идентичности:
  unique constraint для ExternalId не задается
```

До появления общего механизма unique constraints в Object Runtime / persistence uniqueness должна проверяться object-specific validator-ом или DB index-ом конкретного модуля.

### Будущее решение

Если потребуется поддержать несколько внешних идентификаторов на один объект, нужно отдельное решение по модели:

```text
ExternalObjectMapping / CommonExternalIdentity
  TenantId
  ExternalSystemCode
  ExternalObjectType
  ExternalObjectId
  InternalModuleCode
  InternalObjectTypeCode
  InternalObjectId
  InternalStableCode?
  MappingStatus
  CreatedAtUtc
  LastSeenAtUtc
  LastSyncAtUtc
```

Эта модель должна решать:

- область уникальности пары `SourceSystemCode + ExternalId`;
- хранение нескольких источников на один объект;
- API поиска объекта по внешнему идентификатору;
- права и аудит изменения внешних идентификаторов;
- миграцию данных из простого `CommonObject.ExternalId`.

## Решение 3. Поля аудита объекта v1

Статус: принято для Common v1.

### Контекст

В исходных требованиях `DmpBase` содержит поля создания и изменения объекта. В DMP также уже есть отдельная Audit History Platform, которая пишет неизменяемый журнал значимых действий и изменений.

Эти две модели не заменяют друг друга:

- поля аудита объекта описывают текущее системное состояние записи;
- Audit History хранит журнал операций, пригодный для контроля, расследований и compliance.

### Решение

`CommonObject` содержит следующие поля аудита:

```text
CreatedAtUtc
CreatedBy
ModifiedAtUtc?
ModifiedBy?
```

Семантика:

```text
CreatedAtUtc
  дата и время создания объекта в DMP, UTC
  required
  immutable after create

CreatedBy
  actor/security principal id, создавший объект
  required
  immutable after create

ModifiedAtUtc
  дата и время последнего изменения объекта в DMP, UTC
  nullable
  null, если объект после создания не изменялся

ModifiedBy
  actor/security principal id последнего изменения
  nullable
  null, если ModifiedAtUtc = null
```

### Заполнение полей аудита

Поля аудита `CommonObject` принадлежат runtime-логике и не редактируются пользователем напрямую.

Это означает:

```text
CreatedAtUtc / CreatedBy
  выставляются Object Runtime при create

ModifiedAtUtc / ModifiedBy
  выставляются Object Runtime при update
```

Термины текущего кода DMP:

```text
ObjectMutationPipeline
ObjectMutationContext
IObjectMutationLifecycleHook
IObjectLifecycleHandler.BeforeAsync
ObjectMutationContext.SetValue(...)
ITenantContext.UserId
IClock.UtcNow
```

Целевая точка реализации - Object Runtime mutation pipeline до записи объекта.

В текущем коде `ObjectMutationPipeline` выполняет validation, затем внутри transaction вызывает lifecycle hooks, затем object-specific writer, audit sinks и outbox sinks. `ObjectMutationContext.SetValue(...)` уже предназначен для добавления pending values в before lifecycle для технических и производных полей, которые должны быть сохранены вместе с исходной операцией.

Поэтому поля аудита Common v1 должны выставляться не UI, не обработчиком действия и не прикладным модулем вручную, а общей runtime-логикой для object type, основанных на `CommonObject`.

Рекомендуемая реализация:

```text
CommonObjectLifecycleHandler или системный CommonObjectMutationLifecycleHook
  on create:
    CreatedAtUtc = IClock.UtcNow
    CreatedBy = ITenantContext.UserId
    ModifiedAtUtc = null
    ModifiedBy = null

  on update:
    ModifiedAtUtc = IClock.UtcNow
    ModifiedBy = ITenantContext.UserId
```

Если `ITenantContext.UserId` отсутствует или равен `Guid.Empty`, runtime должен использовать явно определенного системного actor-а для системных, интеграционных, миграционных и фоновых сценариев. Молчаливо писать `Guid.Empty` в поля аудита нельзя.

Обычный payload создания или изменения не должен принимать поля аудита как редактируемые пользовательские значения. Исключения допустимы только для специальных сценариев миграции, импорта и администрирования. Такие исключения должны быть явно оформлены отдельным execution profile, командой или bypass policy.

Текущее состояние кода:

- `BusinessEntity` пока содержит только `Id`, `Code`, `Name`;
- ProductDefinition-сущности сейчас сами управляют `ConcurrencyToken`;
- generated writer уже применяет `context.EffectiveValues`, поэтому технические значения, добавленные через before lifecycle, могут быть сохранены вместе с create/update;
- текущий delete writer выполняет hard delete через удаление aggregate, поэтому SoftDelete потребует отдельного изменения в рамках `DeleteCapability`.

### Идентификация инициатора

`CreatedBy` и `ModifiedBy` трактуются не как имя пользователя, а как стабильный идентификатор actor/security principal.

Инициатор операции, или actor, может быть:

- пользователь;
- системный actor;
- integration actor;
- service account;
- migration/background job actor.

Для Common v1 тип полей:

```text
CreatedBy: Guid
ModifiedBy: Guid?
```

Имена пользователя, login, display name или email не хранятся в `CommonObject` как отдельные физические поля.

### Отображение инициатора

Человеко-читаемое отображение actor-полей не является частью структуры данных `CommonObject`.

К полям инициатора относятся:

```text
CreatedBy
ModifiedBy
ArchivedBy
DeletedBy
```

В `CommonObject` хранятся только стабильные идентификаторы actor-а:

```text
Guid
```

Display name, login, email или системное имя actor-а не хранятся в доменных таблицах Common.

Object Runtime должен трактовать actor-поля как системные ссылочные поля и получать человеко-читаемое отображение через общий resolver:

```text
ActorDisplayResolver / IAM resolver
```

Целевая response-семантика:

```text
FieldCode = CreatedBy
Value = actor Guid
DisplayValue = resolved actor display name
```

Примеры display value:

```text
Иван Петров
integration:ERP
system:migration
service:background-job
```

UI не должен самостоятельно знать, как по `Guid` найти пользователя или системного actor-а. Это ответственность Object Runtime / platform resolver-а.

Тип actor-а также не хранится в `CommonObject`.

Поля вида:

```text
CreatedByActorType
ModifiedByActorType
ArchivedByActorType
DeletedByActorType
```

не добавляются.

Причина: такие поля продублируют IAM/Actor Registry во всех доменных таблицах и создадут риск рассинхронизации.

Целевое решение:

```text
CommonObject хранит только actor Guid
IAM / Actor Registry хранит или определяет actor kind/type
ActorDisplayResolver возвращает display value и, при необходимости, actor type для UI/API
```

Это отдельный runtime/design вопрос и не должен приводить к добавлению полей:

```text
CreatedByDisplayName
ModifiedByDisplayName
ArchivedByDisplayName
DeletedByDisplayName
```

в базовый `CommonObject`.

### Связь с Audit History

Audit History продолжает хранить неизменяемые записи операций:

```text
ActionCode
TenantId
Actor/UserId
CorrelationId
OccurredAtUtc
Details
```

Поля аудита объекта не заменяют Audit History и не должны использоваться как полный журнал изменений.

Отдельный `ActorType` в Audit History для Common v1 не вводится. Текущая модель Audit History использует `Actor/UserId`; расширение Audit History actor-моделью может быть отдельным платформенным решением, но не является требованием Common v1.

### Секция "Администрирование" в карточке

Audit/archive/delete поля Common не должны вручную повторяться в каждом baseline `View`.

Для объектов на базе `CommonObject` вводится системная секция карточки:

```text
Администрирование
```

Управление выполняется на двух уровнях.

На уровне object type / base contract:

```text
AdministrationSectionDefault
  Auto
  Hidden
```

Default для object type на базе `CommonObject`:

```text
AdministrationSectionDefault = Auto
```

На уровне `View` artifact:

```text
AdministrationSectionMode
  Inherit
  Auto
  Hidden
  Explicit
```

Семантика:

```text
Inherit
  использовать настройку object type

Auto
  Object Runtime сам добавляет стандартную секцию/таб в конец карточки

Hidden
  не показывать системную секцию в этой карточке

Explicit
  Object Runtime не добавляет секцию автоматически;
  секция описана вручную в View baseline
```

Автоматическое добавление выполняется только для карточки просмотра:

```text
ViewType = ObjectForm / detail card
OperationMode = View
AdministrationSectionMode resolved to Auto
у пользователя есть право на просмотр технических данных
```

Для `Create` и `Edit` карточек секция не добавляется автоматически.

Причина:

```text
Create:
  объект еще не создан, технические поля пустые или бессмысленные

Edit:
  audit/archive/delete поля не редактируются;
  автоматическая readonly-секция в edit form создает путаницу
```

Если специальной admin/edit карточке нужно показать эти поля, `View` должен использовать:

```text
AdministrationSectionMode = Explicit
```

и описать секцию вручную.

По умолчанию в стандартной секции показываются:

```text
CreatedAtUtc
CreatedBy
ModifiedAtUtc
ModifiedBy
IsArchived
ArchivedAtUtc
ArchivedBy
ArchiveReason
IsDeleted
DeletedAtUtc
DeletedBy
DeleteReason
```

Runtime может скрывать условные группы:

```text
archive fields:
  показывать, если IsArchived = true или пользовательский режим требует технические детали

delete fields:
  показывать только в IncludeDeleted/admin/recovery сценариях
```

## Решение 4. Имена базовых классов и трассировка требований

Статус: принято для Common v1.

### Контекст

В исходных требованиях используются имена:

```text
DmpBase
DmpStatusBase
DmpCatalogBase
```

Эти имена отражают старую объектную модель и старую иерархию наследования. В целевой архитектуре Common v1 базовая модель пересобрана:

- статус не входит в базовые классы Common v1;
- каталоговая бизнес-идентификация отделена от базовой технической идентичности;
- `ExternalId` и поля аудита входят в базовый объект;
- `Presentation` является правилом отображения, а не обязательным физически хранимым полем.

### Решение

В целевых дизайн-документах и будущем коде используются имена:

```text
CommonObject
CommonCatalogObject
```

Имена `DmpBase`, `DmpStatusBase`, `DmpCatalogBase` не используются как целевые имена классов в коде Common v1.

### Сопоставление с исходными именами

Для связи с исходными требованиями фиксируется сопоставление:

```text
DmpBase
  -> CommonObject

DmpCatalogBase
  -> CommonCatalogObject

DmpStatusBase
  -> не переносится в Common v1
  -> отдельное будущее решение по workflow
```

### Причины

- `CommonObject` и `CommonCatalogObject` лучше соответствуют текущей модульной архитектуре DMP.
- Новые имена не закрепляют старую цепочку `DmpCatalogBase : DmpStatusBase`.
- Отсутствие `DmpStatusBase` в целевых именах снижает риск неявно вернуть `Status` в базовые классы.
- Сопоставление сохраняет трассировку требований без переноса старой реализации один к одному.

### Таблица трассировки требований ПР00

В составе дизайн-документов Common должна быть отдельная таблица трассировки:

```text
Требование ПР00 / старый класс / старое поле
Решение Common v1
Статус
Пояснение
Где описано в дизайн-документах Common
```

Эта таблица нужна для явного согласования исходных требований с целевой моделью Common v1. Она не должна механически повторять ПР00. Ее задача - показать, что именно переносится, что переосмысливается, что не переносится и почему.

Минимальный состав строк:

```text
DmpBase
DmpCatalogBase
DmpStatusBase
Id
Presentation
ExternalId
Created
CreatedBy
Updated
UpdatedBy
Status
Draft / Release / Archive
```

Примеры целевых решений:

```text
DmpBase -> CommonObject
DmpCatalogBase -> CommonCatalogObject
DmpStatusBase -> не переносится; будущая тема workflow
Presentation -> вычисляемое/настраиваемое представление, не физически хранимое поле CommonObject
Status -> не поле CommonObject; при необходимости реализуется через workflow
Archive -> не значение Status; отдельные поля архива и правила архивирования
Created/Updated -> CreatedAtUtc/ModifiedAtUtc, заполняются Object Runtime
CreatedBy/UpdatedBy -> CreatedBy/ModifiedBy как actor Guid
```

Рекомендуемое размещение:

```text
../00_module_overview.md
```

или отдельное приложение:

```text
../90_traceability_pr00.md
```

Предпочтительный вариант для Common v1 - отдельное приложение `../90_traceability_pr00.md`, чтобы основной дизайн-документ не превращался в сравнение со старой моделью, но трассировка требований оставалась явно доступной.

## Решение 5. Миграция текущего BusinessEntity

Статус: принято как целевое направление для реализации Common v1.

### Контекст

Текущий модуль `DMP.Modules.Common` содержит базовый класс/contract:

```text
BusinessEntity
  Id
  Code
  Name
```

После решений 1-4 эта модель больше не является универсальной базой Common:

- не все объекты имеют `Code` и `Name`;
- в базовом объекте Common v1 должны быть `ExternalId`, поля аудита, архивные поля и поля удаления;
- целевые имена Common v1 - `CommonObject` и `CommonCatalogObject`.

### Решение

`BusinessEntity` не является целевой концепцией Common v1.

Он должен быть мигрирован в:

```text
CommonCatalogObject
```

Дополнительно должен быть добавлен новый базовый объект:

```text
CommonObject
```

Полный состав полей `CommonObject` и `CommonCatalogObject` определяется решением 1.

### Целевая структура кода

```text
src/Modules/DMP.Modules.Common/
  CommonCodes.cs
  CommonModuleRegistration.cs
  CommonBaselineConfiguration.cs

  BaseObjects/
    CommonObject/
      Domain/
        CommonObject.cs
      Runtime/
        CommonObjectObjectContract.cs
      Configuration/
        CommonObjectBaselineConfiguration.cs

    CommonCatalogObject/
      Domain/
        CommonCatalogObject.cs
      Runtime/
        CommonCatalogObjectObjectContract.cs
      Configuration/
        CommonCatalogObjectBaselineConfiguration.cs
```

### Миграционные шаги

1. Добавить `CommonObject`.
2. Добавить `CommonObjectObjectContract`.
3. Добавить `CommonObjectBaselineConfiguration`.
4. Переименовать/заменить `BusinessEntity` на `CommonCatalogObject`.
5. Добавить `CommonCatalogObjectObjectContract`.
6. Добавить `CommonCatalogObjectBaselineConfiguration`.
7. Обновить `CommonCodes`:

```text
CommonCodes.CommonObject
CommonCodes.CommonCatalogObject
```

8. Обновить `CommonBaselineConfiguration`, чтобы он включал оба baseline package.
9. Обновить дочерние object contracts:

```csharp
.BaseObject<BusinessEntityObjectContract>()
```

на:

```csharp
.BaseObject<CommonCatalogObjectObjectContract>()
```

10. Обновить architecture/conformance tests.
11. Удалить `BusinessEntity`. Краткоживущий `[Obsolete]` compatibility alias допускается только если при реализации одномоментная миграция ломает слишком много зависимостей.

### Правило совместимости

Предпочтительный вариант - сразу мигрировать код на `CommonCatalogObject` и не оставлять `BusinessEntity`.

Если при реализации требуется снизить риск одномоментной правки, допускается временный слой совместимости:

```csharp
[Obsolete("Use CommonCatalogObject.")]
public abstract class BusinessEntity : CommonCatalogObject
{
}
```

Такой alias:

- не является целевой архитектурой;
- не должен попадать в дизайн-документы как рекомендованный base object;
- должен быть удален в рамках той же migration/refactoring задачи после обновления дочерних object contracts и тестов.

## Решение 6. Владение tenant и привязка к site

Статус: принято для Common v1.

### Контекст

В DMP уже есть отдельные платформенные механизмы контекста tenant, site, пользователя и прав:

```text
ITenantContext
  TenantId
  SiteId?
  UserId
  RoleCodes

ITenantOwnedEntity
  TenantId
```

В Object Runtime принадлежность объекта tenant-у объявляется явно в контракте объекта:

```csharp
.Tenant(SomeFields.TenantId, x => x.TenantId)
```

Документация Object Runtime также фиксирует, что применение tenant, site и прав доступа является общей обязанностью runtime, а не условием, которое каждый разработчик вручную добавляет во все правила списков, ссылок и фильтров.

### Решение

`TenantId` и `SiteId` не включаются в универсальный `CommonObject`.

Причина: `CommonObject` должен описывать общую базовую модель прикладного объекта, а tenant и site являются областью владения, изоляции, настройки или предметной видимости. Не каждый тип объекта принадлежит tenant-у, и еще меньше типов объектов имеют привязку к site.

Фрагмент модели, важный для tenant / site:

```text
CommonObject
  не содержит TenantId
  не содержит SiteId

Объект, принадлежащий tenant-у
  TenantId
  явно объявляется в runtime contract

Объект, привязанный к site
  SiteId?
  явно объявляется только если site входит в смысл объекта
```

### Tenant

`Tenant` - это основная граница владения и изоляции данных.

Если объект принадлежит tenant-у, он должен:

- иметь `TenantId` в своей доменной модели;
- реализовывать существующий контракт tenant-owned или совместимый паттерн;
- объявлять tenant-поле в Object Runtime contract;
- автоматически попадать под tenant/security scope в стандартных сценариях списка, карточки, поиска и выбора ссылочного объекта.

`TenantId` не должен вручную добавляться в каждое `SelectionCriteria`, если это базовая изоляция данных. Явное tenant-условие допустимо только как предметное правило: например, выбор из корпоративного каталога плюс текущий tenant или специальный сценарий доступа между tenant-ами.

### Site

`Site` - это более узкий контекст площадки/завода и уровень переопределений конфигурации:

```text
SystemBaseline -> Corporate -> Tenant -> Site
```

`SiteId` должен появляться только у объектов, где площадка является частью предметного смысла: производственная структура, ресурсы, рабочие центры, расписания, настройки конкретной площадки или наборы данных, видимые только на конкретной площадке.

Для общих объектов Common и большинства корпоративных/tenant-level справочников `SiteId` не должен быть базовым наследуемым полем.

Для Common v1 отдельный runtime-helper:

```csharp
.Site(...)
```

не вводится.

Если конкретному object type нужен `SiteId`, модуль описывает его как обычное предметное поле или ссылку и сам задает правила фильтрации, выбора и доступности в своих object contracts / views / validators.

Причина: tenant уже является очевидной базовой границей владения и изоляции данных, а site может иметь разные смыслы:

- область переопределения конфигурации;
- предметная ссылка на площадку;
- фильтр видимости;
- производственный контекст операции;
- часть маршрутизации или расписания.

Эти варианты нельзя безопасно свести к одному helper-у без реальных site-scoped сценариев.

Будущая возможность:

```text
SiteScoped
.Site(...)
```

может быть спроектирована отдельным решением Object Runtime после появления конкретных модулей и требований, где site-scope должен применяться системно.

### Что означает "возможность объекта"

В документах Common термин "возможность объекта" означает не обязательно один конкретный C# interface.

"Возможность объекта" - это проектный признак типа объекта, который может реализовываться комбинацией механизмов:

- доменный интерфейс или базовый класс, например `ITenantOwnedEntity`;
- runtime contract, например `.Tenant(...)` или будущий `.StateBinding(...)`;
- baseline configuration: поля, действия, workflow, правила отображения;
- прикладные и runtime-обработчики;
- проверки и права;
- аудит и события.

То есть принадлежность tenant-у, привязка к site, архивирование, управление через workflow и мягкое удаление - это возможности уровня типа объекта. В коде такая возможность может быть выражена интерфейсом, но не сводится только к интерфейсу.

### Сводная таблица общих возможностей объекта

В составе дизайн-документа Common должна быть отдельная таблица "Сводная таблица общих возможностей объекта".

Эта таблица не является новым runtime-механизмом, новым контрактом или новой конфигурацией. Она нужна как документационный чеклист, который показывает, как уже принятые решения согласуются между собой и с будущей реализацией.

Таблица должна отвечать на вопросы:

- входит ли признак физически в `CommonObject`;
- где включается поведение: в базовом объекте, object contract, workflow, view artifact или прикладном модуле;
- кто реализует поведение: Object Runtime, Workflow Runtime, IAM/Actor resolver, прикладной валидатор;
- какое поведение действует по умолчанию;
- что должен сделать прикладной модуль, если ему нужна эта возможность.

Рекомендуемые колонки таблицы:

```text
Возможность
Поля в CommonObject
Как включается поведение
Где реализуется
Поведение по умолчанию
Что делает прикладной модуль
```

Первичный состав строк:

```text
История создания/изменения
Внешний идентификатор
Архивирование
Мягкое удаление
Физическое удаление
Принадлежность tenant-у
Привязка к site
Управление через workflow
Вкладка "Администрирование"
```

Пример строки:

```text
Архивирование
  Поля в CommonObject: IsArchived, ArchivedAtUtc, ArchivedBy, ArchiveReason
  Как включается поведение: прямое действие Archive/Restore в object contract или архивное состояние в workflow
  Где реализуется: Object Runtime + Workflow Runtime при наличии workflow
  Поведение по умолчанию: поля есть, действие архивирования не включается автоматически
  Что делает прикладной модуль: выбирает прямое архивирование или архивирование через workflow
```

Таблицу целесообразно разместить в будущем документе модели Common, например:

```text
../02_domain_model.md
```

В текущем рабочем журнале фиксируется только само решение о необходимости такой таблицы и ее назначение.

## Решение 7. Архивирование объектов

Статус: принято для Common v1.

### Контекст

В исходных требованиях старый статус содержал значение `Archive`, но ранее принято решение не переносить `Status` в базовые классы Common v1.

Архивирование является типовым способом вывести объект из активного использования без физического удаления. В отличие от удаления, архивирование сохраняет объект как бизнес-факт: остаются ссылки, история, аудит, интеграционные сопоставления и возможность открыть карточку по прямому идентификатору.

`Status` не добавляется в `CommonObject`, но технические признаки удаления добавляются. Они нужны не как бизнес-статус, а как единый системный способ исключать удаленные объекты из обычных запросов Object Runtime.

### Решение

Архивные поля включаются в универсальный `CommonObject`:

```text
CommonObject
  IsArchived
  ArchivedAtUtc?
  ArchivedBy?
  ArchiveReason?
```

Наличие этих полей не означает, что каждый тип объекта обязан поддерживать пользовательскую операцию архивирования.

Отдельный обязательный `ArchivePolicy` в Common v1 не вводится.

Причина: в DMP уже есть места, которые управляют доступностью поведения объекта:

- object contract описывает runtime-поверхность объекта: поля, наборы данных, действия, состояние;
- workflow/baseline configuration описывает состояния, команды и переходы workflow;
- права и state policies управляют доступностью действий для пользователя.

Поэтому режим архивирования должен определяться из уже объявленных частей контракта и workflow-конфигурации, а не из отдельного дублирующего enum-а.

### Что описывает Common

Common задает только общую структуру архивных полей и общий смысл стандартных кодов:

```text
IsArchived
ArchivedAtUtc
ArchivedBy
ArchiveReason

Archive
Restore
Archived
```

Важно: Common не добавляет действия `Archive` / `Restore` автоматически в базовый контракт `CommonObject`.

Базовый контракт `CommonObject` должен описывать поля объекта. Он не должен навязывать всем наследникам пользовательские операции архивирования.

Стандартные коды архивирования фиксируются в Common v1:

```text
Archive
Restore
```

Эти коды используются:

- как action codes в object contract для объектов без workflow;
- как workflow command codes для объектов с workflow.

Конкретный прикладной модуль сам объявляет эти действия или команды, если тип объекта поддерживает архивирование. Common задает общий словарь и смысл, но не добавляет поведение автоматически.

### Что описывает object contract

Object contract конкретного типа объекта описывает прямые runtime-действия только тогда, когда этот тип объекта действительно поддерживает прямое архивирование без workflow.

Пример смысла:

```text
Object contract содержит action Archive
  -> объект можно архивировать прямым runtime-действием

Object contract содержит action Restore
  -> объект можно восстановить прямым runtime-действием
```

В этом случае обработчики действий `Archive` / `Restore` меняют архивные поля самого объекта:

```text
Archive
  IsArchived = true
  ArchivedAtUtc = текущее время UTC
  ArchivedBy = текущий пользователь/инициатор
  ArchiveReason = причина, если она передана

Restore
  IsArchived = false
  ArchivedAtUtc = null
  ArchivedBy = null
  ArchiveReason = null
```

Если прямое архивирование для типа объекта не нужно, object contract не должен объявлять actions `Archive` / `Restore`.

### Что описывает workflow

Если тип объекта управляется workflow, архивирование описывается не прямыми runtime actions, а workflow-конфигурацией.

Object contract в этом случае должен только указать, что объект имеет состояние:

```csharp
.Stateful(state => state.StateMember(...))
```

Это приводит к тому, что тип объекта поддерживает workflow через `StateBinding` / `SupportsWorkflow`.

Сами команды архивирования описываются в workflow/baseline configuration:

```text
Workflow
  State Archived
    IsArchiveState = true
  Command Archive
  Command Restore
  Transition SomeState -> Archived by Archive
  Transition Archived -> SomeState by Restore, если восстановление разрешено
```

Для состояния архива должен быть отдельный признак:

```text
Workflow State
  IsArchiveState: bool
```

Название `ToArchive` для признака состояния не принимается, потому что оно описывает действие или намерение перехода, а не устойчивый смысл состояния. Для состояния workflow используется `IsArchiveState`.

### Как расширяется артефакт состояния workflow

`IsArchiveState` добавляется как свойство артефакта `WorkflowState`.

Текущее состояние платформы:

```text
WorkflowState
  Title
  IsInitial
  IsFinal
  Order
```

Целевое состояние:

```text
WorkflowState
  Title
  IsInitial
  IsFinal
  IsArchiveState
  Order
```

Это не `WorkflowStatePolicy`, не свойство перехода и не свойство команды.

Причина: архивность является устойчивым смыслом состояния. Переходы и команды только переводят объект в это состояние или выводят из него.

Требуемые технические изменения:

```text
ConfigurationSchemaPropertyCodes.WorkflowState
  добавить IsArchiveState

WorkflowArtifactSchemas.CreateWorkflowStateSchema
  добавить ArtifactPropertySchema(IsArchiveState, Bool, required: false)

WorkflowStateNodeBuilder
  добавить метод ArchiveState(bool value = true)

WorkflowConfigurationDefinitionResolver
  читать IsArchiveState из published configuration

WorkflowDefinitionState
  добавить IsArchiveState: bool
```

После выполнения workflow-перехода runtime должен сравнить старое и новое состояние:

```text
новое состояние IsArchiveState = true
  установить архивные поля CommonObject

старое состояние IsArchiveState = true,
новое состояние IsArchiveState = false
  очистить архивные поля CommonObject

старое и новое состояние не являются архивными
  архивные поля не менять
```

Для workflow с архивированием должно быть не более одного активного состояния с `IsArchiveState = true`, если отдельный модуль явно не обосновал несколько архивных состояний. Для Common v1 рекомендуется одно архивное состояние.

### Как определяется доступность архивирования

Runtime/UI должны определять доступность архивирования так:

```text
Если object contract содержит прямое action Archive
  -> показывать/выполнять прямое архивирование по правилам action, прав и проверок

Если object contract содержит StateBinding
и опубликованный workflow содержит command Archive и переход в состояние с IsArchiveState = true
  -> показывать/выполнять архивирование как workflow-команду

Если нет ни прямого action Archive, ни workflow-команды Archive
  -> архивирование недоступно, хотя архивные поля физически есть в CommonObject
```

То же правило применяется для восстановления через `Restore`.

Для одного типа объекта нельзя смешивать прямое архивирование и архивирование через workflow. Если объект имеет workflow, источником архивного состояния является только workflow.

### Для объектов без workflow

Если тип объекта не управляется workflow, архивирование выполняется отдельным действием или прикладным сценарием:

```text
Действие Archive
  проверить права
  проверить правила объекта и ссылки
  установить IsArchived = true
  заполнить ArchivedAtUtc / ArchivedBy
  записать Audit History
  опубликовать доменные или интеграционные события, если они нужны
```

Восстановление, если оно разрешено для типа объекта, выполняется отдельным действием `Restore` и снимает архивный признак.

### Для объектов с workflow

Если тип объекта управляется workflow, состояние workflow должно быть главным источником информации о жизненном цикле объекта.

Чтобы избежать рассинхронизации, `IsArchived` для таких объектов должен рассматриваться как материализованный признак, производный от смысла состояния workflow, а не как независимое поле ручного редактирования.

Отдельное прямое изменение `IsArchived` для объекта с workflow запрещено.

При входе в состояние workflow с `IsArchiveState = true` runtime или прикладной сценарий выставляет:

```text
IsArchived = true
ArchivedAtUtc = текущее время UTC
ArchivedBy = идентификатор текущего пользователя/инициатора
```

При выходе из состояния workflow с `IsArchiveState = true` архивные поля очищаются только если такой переход явно разрешен workflow.

### Смысл архивных полей

Архивные поля описывают текущее архивное состояние объекта, а не всю историю архивирования.

Если объект не архивирован:

```text
IsArchived = false
ArchivedAtUtc = null
ArchivedBy = null
ArchiveReason = null
```

Если объект архивирован:

```text
IsArchived = true
ArchivedAtUtc обязателен
ArchivedBy обязателен
ArchiveReason может быть пустым
```

При восстановлении объекта архивные поля очищаются. Сам факт архивирования и восстановления сохраняется в Audit History, а для объектов с workflow - также в истории workflow.

### Видимость

Архивная видимость является общей обязанностью Object Runtime.

Стандартные сценарии списков, поиска и выбора новых ссылочных объектов должны по умолчанию скрывать архивные объекты:

```text
IsArchived = false
```

При этом уже сохраненные ссылки на архивные объекты должны продолжать отображаться в карточках, списках и деталях других объектов.

Пример:

```text
Документ ссылается на справочник, который позже был архивирован.

Карточка документа:
  ссылка должна показывать название архивного справочника

Выбор нового значения в этом же поле:
  архивный справочник по умолчанию не предлагается
```

То есть нужно разделять два сценария:

```text
Отображение уже сохраненной ссылки
  архивный объект видим по правам

Выбор нового ссылочного значения
  архивный объект скрыт по умолчанию
```

В списках должен быть универсальный фильтр показа архива. По умолчанию он выключен.

Фильтр "Показать архивные" оформляется как системный фильтр списка, а не как system action.

Причина: system action вроде `Refresh` выполняет команду интерфейса, а "Показать архивные" меняет условие выборки данных. В текущей runtime-модели для этого есть query parameters, filters и `RuntimeObjectListRequest.Filters`.

Object Runtime автоматически добавляет системный query parameter и filter definition для списков объектов на базе `CommonObject`:

```text
QueryParameterCode = __runtime_include_archived
FilterCode = __runtime_include_archived
Title = Показать архивные
Placement = FilterPanel
ValueType = bool
EditorType = checkbox / boolean
DefaultValue = false
```

Целевая логика фильтра:

```text
__runtime_include_archived = false или не передан
  показывать только IsArchived = false

__runtime_include_archived = true
  показывать IsArchived = false и IsArchived = true
```

Пользовательский интерфейс показывает этот параметр как переключатель "Показать архивные".

При выполнении запроса списка Object Runtime сам преобразует значение `__runtime_include_archived` в системное условие по `IsArchived`. Разработчики прикладных модулей не должны вручную добавлять `IsArchived = false` в каждый список.

Для lookup/reference selection применяется то же правило по умолчанию: архивные объекты не предлагаются для нового выбора, если специальный сценарий явно не разрешает включить архив.

Для расширенных сценариев позже можно добавить режим:

```text
ArchiveVisibility
  ActiveOnly
  ArchivedOnly
  All
```

но для Common v1 достаточно универсального фильтра "Показать архивные".

Карточка объекта по прямому `Id` должна быть доступна по правам, даже если объект архивный, чтобы сохранялись история, расследования и просмотр ссылок.

Архивный объект доступен только для просмотра. Редактирование архивного объекта запрещено, если только конкретный workflow-переход или специальная административная операция явно не разрешает восстановление или изменение технических данных.

## Решение 8. Удаление объектов

Статус: принято для Common v1.

### Контекст

Удаление и архивирование не являются одним и тем же.

Архивирование сохраняет объект как бизнес-факт, но выводит его из активного использования. Удаление убирает объект из текущей модели данных полностью или логически скрывает его как удаленный.

В платформе уже есть `IsDeleted` для узлов конфигурации при merge/override. Это не было основанием автоматически переносить такой же механизм на доменные бизнес-объекты, но для Common v1 принято отдельное решение: доменным объектам нужен общий технический признак удаления, чтобы Object Runtime мог системно исключать удаленные записи из обычных запросов.

### Решение

`IsDeleted` включается в `CommonObject` как технический признак мягкого удаления.

Common v1 вводит две разные системные семантики:

```text
архивирование:
  объект больше не используется, но остается видимым бизнес-фактом

мягкое удаление:
  объект исключен из обычной рабочей модели и должен скрываться из обычных запросов
```

Важно: наличие `IsDeleted` в `CommonObject` не означает, что любой объект можно удалить пользователем или через API. Доступность удаления остается поведением конкретного object type.

```text
CommonObject отвечает за:
  объект архивирован или нет
  объект удален или нет

ObjectType contract отвечает за:
  можно ли удалять сам объект как самостоятельную сущность
  каким способом удалять объект, если удаление разрешено

ObjectCollection contract отвечает за:
  что делать со строками коллекции в контексте владельца
```

Для `CommonObject` сохраняется решение:

```text
CommonObject
  Id
  ExternalId?
  CreatedAtUtc
  CreatedBy
  ModifiedAtUtc?
  ModifiedBy?
  IsArchived
  ArchivedAtUtc?
  ArchivedBy?
  ArchiveReason?
  IsDeleted
  DeletedAtUtc?
  DeletedBy?
  DeleteReason?
```

`IsDeleted` не является бизнес-статусом и не заменяет архивирование.

Причина включения в `CommonObject`: если объект помечен удаленным, Object Runtime должен исключать его из обычных запросов системно, а не требовать от каждого списка, lookup, отчета или модуля вручную добавлять условие `IsDeleted = false`.

### Уровень 1. Архивирование в Common

Архивирование является общей семантикой Common:

```text
IsArchived = true
```

Архивный объект:

```text
виден по уже сохраненным ссылкам
виден в списках при включенном фильтре "Показать архивные"
открывается на просмотр по правам
не редактируется
может быть восстановлен через разрешенную операцию или workflow-переход
```

### Уровень 2. Мягкое удаление в Common

Мягко удаленный объект:

```text
IsDeleted = true
```

Object Runtime должен применять системное правило:

```text
обычные списки, search, lookup и reference selection:
  IsDeleted = false

обычное открытие карточки по Id:
  удаленный объект не открывается как рабочий объект

административные, audit, recovery и reference integrity сценарии:
  могут иметь отдельный технический режим IncludeDeleted
```

Для Common v1 это означает:

```text
разработчики прикладных модулей не добавляют фильтр IsDeleted = false вручную
Object Runtime добавляет его сам для всех object type, основанных на CommonObject
```

Удаленный объект отличается от архивного:

```text
архивный объект остается видимым бизнес-фактом
удаленный объект скрывается из обычной рабочей модели
```

Поэтому для справочников и значимых бизнес-объектов предпочтительная пользовательская операция - архивирование. Удаление применяется там, где объект действительно должен исчезнуть из рабочей модели.

### Уровень 3. Удаление самостоятельного object type

Удаление самого объекта должно быть поведением конкретного object type, а не полем `CommonObject`.

Для Common v1 фиксируется принцип:

```text
по умолчанию наличие IsDeleted не открывает операцию удаления
значимые бизнес-объекты обычно архивируются, а не удаляются
```

Если конкретному object type нужно разрешить удаление, это должно быть явно описано в его object runtime contract и валидаторах. Например, удаление может быть допустимо для временных черновиков, технических импортных буферов или строк, которые не имеют самостоятельной бизнес-идентичности.

В object contract вводится явная возможность удаления:

```text
DeleteCapability
  Disabled
  SoftDelete
  HardDelete
```

Default:

```text
DeleteCapability = Disabled
```

Примерный fluent API:

```csharp
.Delete(delete => delete.Disabled())
.Delete(delete => delete.Soft())
.Delete(delete => delete.Hard())
```

`DeleteCapability` управляет только корневым удалением объекта как самостоятельной сущности. Он не заменяет настройки коллекций.

Семантика:

```text
Disabled:
  root delete запрещен
  стандартное действие Delete не появляется
  DELETE endpoint возвращает ошибку operation not supported

SoftDelete:
  IsDeleted = true
  DeletedAtUtc = current UTC
  DeletedBy = current actor
  DeleteReason = reason if passed
  объект исчезает из обычных списков, search, lookup и reference selection

HardDelete:
  физическое удаление записи
  допустимо только для объектов, где это явно разрешено и безопасно
```

Права:

```text
{ModuleCode}.{ObjectTypeCode}.Delete
```

Удаление должно проверять отдельное право `*.Delete`, а не `*.Edit`.

Стандартное runtime-действие:

```text
ActionCode = __runtime_delete
StandardActionKind = Delete
ExecutionBoundary = ObjectMutation
Placement = Row/Menu
```

Object Runtime может материализовать стандартное действие `Delete`, только если одновременно:

```text
DeleteCapability != Disabled
у пользователя есть право *.Delete
объект не заблокирован state/read-only политикой, если такая политика есть
view/list разрешает показывать действие
```

Workflow не является обязательной частью удаления.

Он учитывается только для тех object type, которые сами подключили workflow/state policies. Если у object type нет workflow, удаление определяется только `DeleteCapability`, правами, валидаторами и настройками коллекций.

Если object type управляется workflow, delete не должен обходить жизненный цикл объекта. Поэтому root delete по умолчанию запрещен в read-only/state-restricted состоянии и может быть разрешен только явной state policy или отдельной workflow-командой, если удаление является частью бизнес-процесса.

Текущий Object Runtime уже имеет технический delete endpoint и `ObjectMutationKind.Delete`, но реализацию нужно привести к этому контрактному решению:

```text
DELETE /objects/{objectTypeCode}/{id}
ObjectMutationKind.Delete
```

### Уровень 4. Удаление строк коллекций

Настройки удаления у коллекций не являются политикой удаления `CommonObject`.

Они отвечают только на вопрос: что делать со строкой коллекции в контексте объекта-владельца.

В Object Runtime уже есть:

```text
Aggregation
SaveMode
ObjectCollectionDeleteBehavior
ObjectCollectionMissingItemBehavior
RuntimeSystemActionCodes.CollectionDelete
RuntimeSystemActionCodes.CollectionUnlink
```

Для aggregate-коллекции обычно:

```text
Aggregation = Aggregate
SaveMode = WithOwner
MissingItemBehavior = Delete
DeleteBehavior = Cascade
```

Смысл: строка является частью владельца. Если строку убрали из карточки владельца, она удаляется при сохранении владельца. Если удаляется владелец, строки удаляются вместе с ним.

Для association/separate-коллекции обычно:

```text
Aggregation = Association
SaveMode = Separate
MissingItemBehavior = Ignore
DeleteBehavior = Restrict
```

Смысл: строка коллекции или связанный объект не исчезает автоматически только потому, что ее нет в payload объекта-владельца.

Политика коллекции и политика самого object type работают на разных уровнях:

```text
ObjectCollection.DeleteBehavior:
  что делать со строкой в контексте владельца

ObjectType root delete:
  можно ли удалить сам объект как самостоятельную сущность
```

Пример:

```text
Документ
  Строки документа
```

У коллекции строк может быть:

```text
DeleteBehavior = Cascade
MissingItemBehavior = Delete
```

Но у типа `СтрокаДокумента` может быть правило:

```text
самостоятельное удаление запрещено
удаление возможно только через документ-владелец
```

Если строка коллекции основана на `CommonObject`, наличие полей `IsDeleted` не решает само по себе, что делать при удалении строки из коллекции. Это решается настройками коллекции и политикой target object type.

```text
ObjectCollection:
  определяет, удаляется ли строка при удалении/сохранении владельца

Target ObjectType:
  определяет, выполняется ли это как SoftDelete или HardDelete, если такая детализация нужна
```

Для `Separate`-коллекции, где строка удаляется отдельной Object Runtime операцией, `CollectionDelete` должен учитывать `DeleteCapability` target object type.

Для `WithOwner` aggregate-коллекции удаление строки в draft владельца остается поведением владельца и коллекции. При сохранении владельца runtime применяет `MissingItemBehavior` и `DeleteBehavior`; способ физического или мягкого удаления строк должен быть согласован с target object type, если строки являются самостоятельными `CommonObject`.

### Уровень 5. Получение удаленных объектов

Для обычных runtime-запросов удаленные объекты скрываются:

```text
IsDeleted = false
```

Для технических сценариев вводится режим:

```text
IncludeDeleted
```

`IncludeDeleted` не является обычным пользовательским фильтром списка. Он предназначен для административных, audit, recovery и reference integrity сценариев.

Семантика:

```text
обычный запрос:
  IsDeleted = false

IncludeDeleted = true:
  active + deleted

только удаленные:
  IncludeDeleted = true + IsDeleted = true
```

### Уровень 6. Проверка ссылок перед HardDelete

Для Common v1 hard delete требует явной проверки в object-specific validator/delete guard.

Целевое платформенное развитие - общий `ReferenceIntegrityService`, который сможет проверять входящие ссылки на основе object contracts. Но это отдельная платформенная задача и не является блокером для Common v1.

Правило Common v1:

```text
если есть сомнение в ссылочной безопасности, HardDelete запрещен
для значимых бизнес-объектов используется архивирование или SoftDelete
HardDelete применяется только для технических, временных или дочерних объектов
```

### Правила Common v1

- `IsDeleted`, `DeletedAtUtc`, `DeletedBy`, `DeleteReason` добавляются в `CommonObject`.
- `IsDeleted` является техническим признаком исключения из обычных запросов, а не бизнес-статусом.
- Object Runtime должен системно исключать `IsDeleted = true` из обычных списков, search, lookup и reference selection.
- Наличие `IsDeleted` не означает, что объект можно удалить: операция удаления должна быть явно разрешена конкретным object type.
- В object contract вводится `DeleteCapability = Disabled | SoftDelete | HardDelete`.
- Default для object type: `DeleteCapability = Disabled`.
- Root delete должен проверять отдельное право `*.Delete`, а не `*.Edit`.
- Object Runtime может материализовать стандартное действие `__runtime_delete` только при разрешенном `DeleteCapability`.
- Технический режим получения удаленных объектов называется `IncludeDeleted`.
- `IncludeDeleted` не показывается как обычный пользовательский фильтр списка.
- Для справочников и основных бизнес-объектов предпочтительно архивирование.
- Удаление строк коллекций описывается существующими настройками Object Runtime collection contract.
- Корневое удаление объекта должно быть явно разрешено конкретным object type и должно определить способ удаления: SoftDelete или HardDelete.
- Hard delete в Common v1 требует object-specific validator/delete guard.
- Hard delete не должен нарушать ссылочную целостность, workflow history, audit history и integration mapping.
- Workflow учитывается только для object type, которые сами подключили workflow/state policies. Для объектов без workflow он не участвует в решении об удалении.

## Решение 9. Граница применения CommonObject к платформенным модулям ядра

Статус: принято для Common v1.

### Контекст

В текущем коде DMP есть не только прикладные модули, но и платформенные модули ядра:

```text
DMP.Platform.Configuration
DMP.Platform.Workflow
DMP.Platform.AuditHistory
DMP.Platform.IntegrationEvents
DMP.Platform.Runtime
DMP.Platform.Settings
DMP.Platform.TenantSecurity
DMP.Platform.ValueSets
DMP.Platform.Rules
DMP.BuildingBlocks.*
```

У этих модулей уже есть собственные доменные сущности и жизненные циклы:

- `ConfigurationVersion` имеет собственный статус `Draft / Published / Archived / Failed`;
- `ConfigurationEntry` имеет собственный `IsDeleted`, который означает удаление узла конфигурации внутри версии;
- `WorkflowStateEntry` и `WorkflowHistoryEntry` описывают состояние и историю workflow;
- `AuditRecordEntry` является журналом аудита;
- `OutboxMessage` является записью outbox/event queue;
- `RuntimeSettingValue` и `UserPreferenceValue` являются значениями настроек;
- `Tenant`, `Site`, `User`, `Role`, `Permission` являются объектами IAM/tenant-security со своей security-семантикой.

При этом часть платформенных объектов может отображаться через стандартные runtime-представления: списки, карточки, lookup, actions, reference selection.

### Решение

`CommonObject` и `CommonCatalogObject` не являются обязательными базовыми классами для платформенных модулей ядра.

Базовые классы Common предназначены прежде всего для прикладных доменных объектов, которые проектируются как обычные управляемые бизнес-объекты DMP.

Платформенные и системные сущности не переводятся массово на наследование от `CommonObject`.

Object Runtime и базовые классы Common являются разными решениями:

```text
CommonObject / CommonCatalogObject
  базовая доменная модель прикладного объекта

Object Runtime
  механизм публикации object type, списков, карточек, lookup, actions, мутаций и runtime UI
```

Платформенный объект может использовать Object Runtime без наследования от `CommonObject`, если для него нужен единый runtime UI/API.

### Почему не переводим все платформенные сущности на CommonObject

Наследование платформенных сущностей от `CommonObject` может смешать разные смыслы одинаково названных признаков.

Примеры:

```text
ConfigurationEntry.IsDeleted
  удаление узла конфигурации внутри draft/version graph
  не равно SoftDelete бизнес-объекта Common

ConfigurationVersion.Status = Archived
  архивирование версии конфигурации
  не равно Common.IsArchived управляемого бизнес-объекта

WorkflowStateEntry.UpdatedAtUtc / UpdatedBy
  изменение состояния workflow
  не равно ModifiedAtUtc / ModifiedBy самого бизнес-объекта

OutboxMessage.PublishedAtUtc / DeadLetteredAtUtc
  состояние доставки события
  не равно жизненному циклу бизнес-объекта
```

Поэтому Common не должен становиться универсальным предком для всех таблиц платформы.

### Что не теряется без наследования от CommonObject

Если платформенный объект публикуется через Object Runtime descriptor/contract, он может использовать runtime-возможности без наследования от Common:

- стандартные списки и карточки;
- runtime views;
- lookup/reference selection;
- presentation/display rules;
- actions и bulk actions;
- validators;
- lifecycle handlers;
- defaults providers;
- tenant filtering через `.Tenant(...)`;
- concurrency token;
- audit history mutation event через `ObjectRuntimeAuditSink`;
- workflow projection, если object type явно подключил workflow/state binding.

То есть отсутствие наследования от базового класса Common не означает потерю runtime UI/API возможностей.

### Когда платформенный объект может подключать Common-семантику

Платформенный модуль может явно использовать отдельные Common-подходы, если их смысл совпадает с предметной семантикой объекта.

Примеры допустимого использования:

```text
Tenant / Site / Role
  могут публиковаться как управляемые runtime-административные объекты
  но не обязаны наследоваться от CommonCatalogObject

ValueSetItem
  может отображаться через runtime UI
  но его scoped value-set semantics остаются в ValueSets

AuditRecordEntry / WorkflowHistoryEntry / OutboxMessage
  могут иметь read-only admin/monitoring views
  но не становятся CommonObject
```

Если платформенному объекту действительно нужны поля `ExternalId`, `IsArchived`, `IsDeleted` или поля аудита именно в смысле Common, это решение принимается отдельно для конкретного object type.

### Правила Common v1

- Базовые классы Common обязательны только для новых прикладных объектов, если они проектируются как стандартные управляемые бизнес-объекты DMP.
- Платформенные и системные сущности не обязаны наследоваться от `CommonObject`.
- Платформенные модули могут использовать Object Runtime как фасад без наследования от базовых классов Common.
- Нельзя автоматически трактовать платформенные поля `Status`, `IsDeleted`, `UpdatedAtUtc`, `Archived` как Common-семантику.
- Для платформенных объектов Common-поля и Common-поведение подключаются только явно и только если совпадает смысл.
- `DMP.Platform.Runtime` и `DMP.BuildingBlocks.*` не должны зависеть от `DMP.Modules.Common`, чтобы не создавать обратную зависимость нижних слоев от прикладного модуля Common.

### Предварительная классификация текущих модулей

```text
TenantSecurity
  CommonObject: нет, не массово
  Object Runtime: да, выборочно для admin-managed objects

ValueSets
  CommonObject: нет или только после отдельного решения
  Object Runtime: возможно для UI value-set administration

Settings
  CommonObject: нет
  Object Runtime: обычно нет, кроме специальных UI-оберток

Configuration
  CommonObject: нет
  Object Runtime: использует собственный configuration authoring/runtime

Workflow
  CommonObject: нет
  Object Runtime: нет как Common-object; интеграция через workflow projection/state binding

AuditHistory
  CommonObject: нет
  Object Runtime: только read-only journal UI при необходимости

IntegrationEvents
  CommonObject: нет
  Object Runtime: только admin/monitoring UI при необходимости

Rules
  CommonObject: нет
  Object Runtime: пока не требуется

Platform.Runtime
  CommonObject: нет
  Object Runtime: это сам runtime

BuildingBlocks
  CommonObject: нет
  Object Runtime: нет
```

## Решение 10. Слои базовых классов и зависимостей

Статус: принято для Common v1.

### Контекст

После решения о границе применения `CommonObject` к платформенным модулям важно явно разделить:

- минимальные доменные примитивы в `BuildingBlocks`;
- платформенные и системные сущности;
- прикладные бизнес-объекты;
- базовые объекты Common как стандартную базу для прикладных управляемых объектов.

Без такого разделения есть риск ошибочно трактовать `CommonObject` как замену `BuildingBlocks.Entity` или как обязательный базовый класс для всей платформы.

### Решение

В Common v1 фиксируется следующая слоистая модель:

```text
BuildingBlocks.Entity
  минимальная доменная сущность
  содержит только базовую техническую идентичность, например Id

BuildingBlocks.TenantEntity / AuditableTenantEntity / другие примитивы
  низкоуровневые переиспользуемые примитивы
  не зависят от Common

Платформенные и системные сущности
  внутренние сущности платформенных модулей
  могут наследоваться от примитивов BuildingBlocks
  не обязаны наследоваться от CommonObject

CommonObject
  базовый класс прикладного управляемого бизнес-объекта
  задает общие поля Common: Id, ExternalId, поля аудита, архивирование, мягкое удаление

CommonCatalogObject : CommonObject
  базовый класс прикладного каталогового бизнес-объекта
  добавляет Code + Name

Объект прикладного модуля
  конкретный объект прикладного модуля
  наследуется от CommonObject или CommonCatalogObject, если он является стандартным управляемым бизнес-объектом DMP
```

`CommonObject` не заменяет `BuildingBlocks.Entity`.

`BuildingBlocks.Entity` остается самым нижним минимальным примитивом. Он может использоваться платформенными и прикладными модулями там, где Common-семантика не нужна.

### Правило зависимостей

Направление зависимостей:

```text
BuildingBlocks
  <- Platform
  <- Modules.Common
  <- Прикладные модули
```

При этом:

```text
BuildingBlocks
  не зависит от Platform
  не зависит от Modules.Common

Platform.Runtime
  не зависит от Modules.Common

Modules.Common
  может зависеть от BuildingBlocks и runtime-контрактов
  но не должен становиться нижним системным слоем для всей платформы

Прикладные модули
  могут зависеть от Modules.Common
```

### Что это означает для проектирования объектов

Для нового объекта прикладного модуля выбор базового класса делается так:

```text
если объект является обычным управляемым бизнес-объектом DMP:
  CommonObject

если объект является каталоговым бизнес-объектом с Code + Name:
  CommonCatalogObject

если объект является внутренней технической или системной сущностью:
  BuildingBlocks.Entity или специальный платформенный/доменный базовый класс

если объект является журналом, outbox, workflow state, configuration node или settings value:
  не CommonObject по умолчанию
```

Object Runtime contract может быть добавлен независимо от выбора доменного базового класса, если объект нужно показывать через стандартный runtime UI/API.

### Правила Common v1

- `CommonObject` является стандартной базой прикладного управляемого объекта, но не универсальной базой всех сущностей DMP.
- `CommonCatalogObject` используется только там, где `Code` и `Name` действительно являются обязательной каталоговой идентификацией.
- `BuildingBlocks.Entity` остается минимальным низкоуровневым базовым классом.
- Платформенные и системные сущности не переводятся на базовые классы Common ради единообразия.
- Runtime-фичи подключаются через object contract/descriptor и не требуют обязательного наследования от `CommonObject`.
- Если конкретный платформенный объект хочет использовать Common-семантику, это фиксируется отдельным решением для этого object type.
