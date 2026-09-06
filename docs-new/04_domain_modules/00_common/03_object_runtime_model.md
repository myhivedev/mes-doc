---
id: DOC-04-00-03
title: 'Runtime-модель объектов - 00 Common'
type: design
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 00_common
holder: '@axelprosoft'
created_at: 2026-08-07 10:40
created_by: '@axelprosoft'
updated_at: 2026-08-19 17:22
last_modified_by: '@VeronikaV2121'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Runtime-модель объектов - 00 Common

## 1. Назначение документа

Документ описывает, как объекты Common должны использоваться в Object Runtime.

Документ отвечает на вопросы:

- какие общие поля Common видны в runtime;
- какие системные действия и фильтры нужны объектам Common;
- как показываются технические поля;
- как Common связан с workflow, tenant, site и удалением;
- где проходит граница между доменной моделью Common и runtime-поведением.

Практическое руководство и актуальные точки реализации приведены в `src/Modules/DMP.Modules.Common/README.md`.

## 2. Базовые типы объектов

Object Runtime должен поддерживать два базовых типа Common:

```text
CommonObject
CommonCatalogObject
```

`CommonObject` публикует общие технические поля прикладного объекта.

`CommonCatalogObject` наследует поля `CommonObject` и добавляет `Code` и `Name`.

`Code` и `Name` не должны автоматически появляться у всех объектов Common. Они нужны только у справочников и каталоговых объектов.

## 3. Поля Common в runtime

Поля `CommonObject` в runtime:

| Группа | Поля | Runtime-смысл |
|---|---|---|
| Идентификация | `Id`, `ExternalId` | Идентификатор объекта и необязательный внешний идентификатор. |
| Создание | `CreatedAtUtc`, `CreatedBy` | Системные поля создания. |
| Изменение | `ModifiedAtUtc`, `ModifiedBy` | Системные поля последнего изменения. |
| Архивирование | `IsArchived`, `ArchivedAtUtc`, `ArchivedBy`, `ArchiveReason` | Системные поля архивного состояния. |
| Мягкое удаление | `IsDeleted`, `DeletedAtUtc`, `DeletedBy`, `DeleteReason` | Системные поля исключения объекта из обычной рабочей модели. |

Поля `CommonCatalogObject`:

```text
Code
Name
```

Обычные пользовательские формы создания и изменения не должны редактировать системные поля Common напрямую. Правила заполнения и изменения этих полей описаны в `05_rules.md`.

## 4. Поля инициатора

К полям инициатора относятся:

```text
CreatedBy
ModifiedBy
ArchivedBy
DeletedBy
```

В доменной модели хранится только стабильный идентификатор инициатора.

Object Runtime должен уметь показывать человекочитаемое имя инициатора через общий механизм разрешения, например IAM / ActorDisplayResolver.

В таблицах Common не должны дублироваться поля:

```text
CreatedByDisplayName
ModifiedByDisplayName
ArchivedByDisplayName
DeletedByDisplayName
```

## 5. Представление объекта

`Presentation` не является физическим полем Common.

Object Runtime должен использовать правило представления конкретного типа объекта.

Примеры правил:

```text
{Code} - {Name}
{Name}
{ExternalId} - {Name}
```

Для `CommonCatalogObject` типовым правилом может быть:

```text
{Code} - {Name}
```

Конкретный тип объекта может задать другое правило.

## 6. Tenant и Site

`TenantId` и `SiteId` не входят в `CommonObject`.

Если объект принадлежит tenant-у, tenant-поле объявляется в модели и runtime-описании конкретного типа объекта. Object Runtime должен применять tenant-ограничение централизованно.

`SiteId` добавляется только тем объектам, где site входит в предметный смысл. В Common v1 для site не вводится универсальный runtime-механизм.

## 7. Архивирование

Common задает стандартные коды:

```text
Archive
Restore
Archived
```

`Archive` и `Restore` зарегистрированы как стандартные IAM verbs с едиными переводами. Конкретные права остаются ресурсными: `{ModuleCode}.{ObjectTypeCode}.Archive` и `{ModuleCode}.{ObjectTypeCode}.Restore`. Общий verb не объединяет обработчики Configuration, Common или Workflow.

Наличие архивных полей не означает, что действие архивирования доступно автоматически.

Для объекта без workflow архивирование доступно только если тип объекта явно объявил действия:

```text
Archive
Restore
```

Для объекта с workflow архивирование должно выполняться через workflow. Архивное состояние workflow должно иметь признак:

```text
IsArchiveState = true
```

Для одного типа объекта нельзя смешивать прямое архивирование и архивирование через workflow.

Правила выполнения архивирования и восстановления описаны в `05_rules.md`.

## 8. Фильтр "Показать архивные"

Списки объектов на базе `CommonObject` должны по умолчанию скрывать архивные объекты.

Object Runtime должен поддерживать системный фильтр:

```text
Код: __runtime_include_archived
Название: Показать архивные
Тип значения: bool
Значение по умолчанию: false
```

Поведение:

```text
false или не передан
  показывать только неархивные объекты

true
  показывать неархивные и архивные объекты
```

Это фильтр списка, а не действие интерфейса.

Выбор нового ссылочного значения также должен скрывать архивные объекты по умолчанию. Уже сохраненные ссылки на архивные объекты должны продолжать отображаться.

## 9. Удаление

Наличие поля `IsDeleted` не открывает операцию удаления.

Для типа объекта должна быть задана возможность удаления:

```text
DeleteCapability
  Disabled
  SoftDelete
  HardDelete
```

Значение по умолчанию:

```text
DeleteCapability = Disabled
```

Удаление должно проверять отдельное право:

```text
{ModuleCode}.{ObjectTypeCode}.Delete
```

Право `*.Edit` не должно автоматически разрешать удаление.

Правила мягкого и физического удаления описаны в `05_rules.md`.

## 10. Стандартное действие удаления

Object Runtime может показывать стандартное действие удаления только если:

- `DeleteCapability` не равен `Disabled`;
- у пользователя есть право `*.Delete`;
- объект не заблокирован состоянием или режимом только для просмотра;
- текущий список или карточка разрешает показывать действие.

Если тип объекта управляется workflow, удаление не должно обходить жизненный цикл. Такое удаление должно быть явно разрешено правилами состояния или workflow-командой.

## 11. IncludeDeleted

Обычные запросы Object Runtime должны исключать мягко удаленные объекты.

Для технических сценариев вводится режим:

```text
IncludeDeleted
```

Он предназначен для:

- администрирования;
- аудита;
- восстановления;
- проверки ссылочной целостности;
- технических расследований.

`IncludeDeleted` не должен показываться как обычный пользовательский фильтр списка.

## 12. Удаление строк коллекций

`DeleteCapability` описывает удаление типа объекта как самостоятельной сущности.

Он не заменяет настройки удаления строк коллекций.

Для коллекций продолжают использоваться существующие настройки Object Runtime:

```text
Aggregation
SaveMode
ObjectCollectionDeleteBehavior
ObjectCollectionMissingItemBehavior
CollectionDelete
CollectionUnlink
```

Если строка коллекции является самостоятельным `CommonObject`, способ ее удаления должен быть согласован с `DeleteCapability` целевого типа объекта.

## 13. Секция "Администрирование"

Поля аудита, архива и удаления Common не должны вручную повторяться в каждой карточке.

Для объектов на базе `CommonObject` вводится системная секция:

```text
Администрирование
```

Секция добавляется автоматически только в карточку просмотра, если:

- объект построен на базе `CommonObject`;
- режим карточки - просмотр;
- пользователь имеет право видеть технические данные;
- карточка не скрыла секцию явно.

Для карточек создания и изменения секция не добавляется автоматически.

По умолчанию в секции показываются:

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

## 14. Граница с платформенными объектами

Наследование от `CommonObject` и публикация через Object Runtime - разные решения.

Платформенный объект может использовать Object Runtime без наследования от Common, если ему нужен единый runtime-интерфейс для просмотра или администрирования.

Примеры:

```text
Tenant / Site / Role
  могут публиковаться как управляемые административные объекты
  не обязаны наследоваться от CommonCatalogObject

AuditRecordEntry / WorkflowHistoryEntry / OutboxMessage
  могут иметь административные или мониторинговые представления только для чтения
  не становятся CommonObject
```

Если платформенному объекту действительно нужны поля Common, это должно быть отдельным проектным решением для конкретного типа объекта.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-07 10:40 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [dee4ebb0](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/dee4ebb02ed702461463c16315f3b252ae418edd) |
