---
id: DOC-04-00-20
title: 'Бэклог реализации — 00 Common v1'
type: appendix
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 00_common
holder: '@axelprosoft'
created_at: 2026-08-06 14:02
created_by: '@axelprosoft'
updated_at: 2026-09-02 11:50
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Бэклог реализации — 00 Common v1

## 1. Назначение документа

Документ задает скорректированный бэклог реализации `Common v1` на основе:

- `00_module_overview.md`;
- `01_domain_model.md`;
- `03_behavior_rules.md`;
- `14_object_runtime_usage.md`;
- `90_traceability_pr00.md`;
- `_working/decision_log.md` (решения 1–10, прочитан полностью);
- анализа текущих реализаций `BusinessEntity`, `Nomenclature` и Object Runtime;
- решений, принятых при согласовании миграции.

Документ является планом реализации. Нормативная семантика Common остается в дизайн-документах, перечисленных выше.

## 2. Зафиксированные решения

### 2.1. Миграция BusinessEntity

Переход выполняется как breaking change без совместимого alias:

```text
BusinessEntity
  удаляется после перевода существующих наследников

CommonObject
  новый базовый тип прикладных управляемых объектов

CommonCatalogObject : CommonObject
  новый базовый тип каталоговых объектов
```

Код object type `Common:BusinessEntity` не сохраняется как alias.

Единственный текущий доменный наследник `BusinessEntity` — `Nomenclature`. Он должен быть переведен на `CommonCatalogObject` до удаления `BusinessEntity`.

После миграции удаляются:

- доменный класс `BusinessEntity`;
- `BusinessEntityObjectContract`;
- `BusinessEntityBaselineConfiguration`;
- `CommonCodes.BusinessEntity`;
- регистрация object type `BusinessEntity`;
- все ссылки на `Common:BusinessEntity` в baseline-конфигурации, тестах и документации.

### 2.2. Замена внешнего идентификатора Nomenclature

Существующее поле `Nomenclature.ExternalCode` и новое поле `CommonObject.ExternalId` имеют одинаковый бизнес-смысл.

В новой модели используется только `ExternalId` с правилами Common:

```text
normalized = Trim(ExternalId)

если normalized пуст:
  ExternalId = null

если длина normalized <= 100:
  ExternalId = normalized

если длина normalized > 100:
  ExternalId = первые 100 символов normalized
```

После переноса данных удаляются:

- свойство `Nomenclature.ExternalCode`;
- поле `ExternalCode` в mutation-моделях;
- runtime member и stable code `ExternalCode`;
- колонка `ExternalCode`;
- связанные элементы форм, списков, фильтров, команд и тестов.

Перенос значений из `ExternalCode` не требуется, поскольку обновленная система запускается на пустой БД.

### 2.3. Запуск на пустой БД

Обновленная система запускается на пустой БД.

Следствия:

- перенос данных `ExternalCode → ExternalId` не выполняется;
- новая persistence-схема должна сразу содержать целевые поля и ограничения Common;
- начальные записи, если они создаются bootstrap/seeding-механизмами, проходят обычные правила создания Common и получают реального инициатора операции.

Целевая модель с обязательным actor context остается отдельной будущей доработкой.

## 3. Текущее состояние реализации

На момент подготовки бэклога:

- `BusinessEntity` содержит только `Id`, `Code`, `Name`;
- `BusinessEntityObjectContract` публикует только эти три поля;
- `Nomenclature` является единственным наследником `BusinessEntity`;
- в contract/descriptor отсутствует `DeleteCapability`;
- list, lookup и details не применяют системные фильтры `IsArchived` и `IsDeleted`;
- runtime delete направляется storage writer-у без общей политики `Disabled / SoftDelete / HardDelete`;
- API list/details не содержит технического режима `IncludeDeleted`;
- стандартные Archive/Restore для Common не реализованы;
- системная секция «Администрирование» не материализуется автоматически;
- разрешение отображаемого имени actor-а для полей Common не реализовано.

## 4. Приоритеты и оценки

Приоритеты:

```text
P0 — блокирует безопасный переход на Common v1
P1 — обязательное runtime-поведение Common v1
P2 — завершение внедрения, тестирование и сопровождение
Future — целевая архитектура за пределами Common v1
```

Относительные оценки:

```text
S  — локальная доработка
M  — несколько связанных компонентов
L  — сквозная доработка модуля или runtime pipeline
XL — изменение нескольких подсистем с миграцией и интеграционными тестами
```

## 5. Бэклог

### COM-01. Реализовать доменный класс CommonObject

Приоритет: `P0`  
Оценка: `M`

Состав работ:

- [x] добавить абстрактный `CommonObject`;
- [x] унаследовать `CommonObject` от минимального доменного примитива `BuildingBlocks.Entity`, не дублируя низкоуровневую модель идентичности;
- [x] добавить `Id` и `ExternalId`;
- [x] добавить поля создания и изменения;
- [x] добавить поля архивирования;
- [x] добавить поля мягкого удаления;
- [x] реализовать нормализацию `ExternalId`;
- [x] реализовать доменные инварианты системных состояний.

Критерии готовности:

- `ExternalId` имеет тип `string?`, выполняет `Trim`, преобразует пустое значение в `null` и ограничен 100 символами;
- `CreatedBy`, `ModifiedBy`, `ArchivedBy`, `DeletedBy` хранят только стабильный `Guid` actor-а;
- доменные методы не допускают частично заполненного архивного или удаленного состояния;
- `Status`, `TenantId`, `SiteId`, `SourceSystemCode` и физическое поле `Presentation` отсутствуют;
- `CommonObject` не заменяет `BuildingBlocks.Entity` и не применяется автоматически к платформенным сущностям;
- unit-тесты покрывают нормализацию и инварианты.

### COM-02. Реализовать CommonCatalogObject

Приоритет: `P0`  
Оценка: `S`

Состав работ:

- [x] добавить `CommonCatalogObject : CommonObject`;
- [x] перенести каталоговые поля `Code` и `Name`;
- [x] сохранить обязательность и нормализацию `Code` и `Name`.

Критерии готовности:

- `Code` и `Name` отсутствуют в `CommonObject`;
- `Code` и `Name` обязательны в `CommonCatalogObject`;
- текущая семантика нормализации `BusinessEntity` сохранена.

Зависимость: `COM-01`.

### COM-03. Добавить stable codes и базовые runtime-контракты Common

Приоритет: `P0`  
Оценка: `L`

Состав работ:

- [x] добавить `CommonCodes.CommonObject`;
- [x] добавить `CommonCodes.CommonCatalogObject`;
- [x] реализовать `CommonObjectObjectContract`;
- [x] реализовать `CommonCatalogObjectObjectContract` с наследованием контракта CommonObject;
- [x] зарегистрировать оба base object type;
- [x] добавить baseline-конфигурации общих полей и локализаций;
- [x] обеспечить типовой поиск и фильтрацию по `ExternalId` без глобального правила сравнения и уникальности.

Критерии готовности:

- системные поля Common публикуются как read-only;
- `Code` и `Name` наследуются только от `CommonCatalogObject`;
- object type codes не используют `BusinessEntity`;
- прикладной контракт может наследовать любой из двух новых базовых контрактов;
- effective configuration корректно разрешает цепочку наследования;
- `ExternalId` может быть скрыт view policy конкретного object type, оставаясь частью базового контракта;
- Common не вводит отдельную `ExternalIdUniquenessPolicy`: уникальность задается общим unique constraint, object-specific validator-ом или DB index-ом.

Зависимости: `COM-01`, `COM-02`.

### COM-04. Расширить object contract и descriptor возможностями Common

Приоритет: `P0`  
Оценка: `L`

Состав работ:

- [x] добавить метаданные принадлежности к Common base object;
- [x] добавить `DeleteCapability` со значениями `Disabled`, `SoftDelete`, `HardDelete`;
- [x] установить `Disabled` по умолчанию;
- [x] добавить декларативное включение прямых действий Archive/Restore;
- [x] отличать прямое архивирование от архивирования через workflow;
- [x] передавать метаданные в runtime descriptor и effective configuration;
- [x] выразить Common capability через нейтральные runtime contracts/descriptor metadata без зависимости `DMP.Platform.Runtime → DMP.Modules.Common`;
- [x] добавить metadata `AdministrationSectionDefault = Auto | Hidden` и `AdministrationSectionMode = Inherit | Auto | Hidden | Explicit`.

Критерии готовности:

- наличие полей Common не включает Archive, Restore или Delete автоматически;
- отсутствие явной настройки означает `DeleteCapability = Disabled`;
- startup validation обнаруживает несовместимые настройки capability;
- contract/descriptor-тесты покрывают наследование и значения по умолчанию;
- `DMP.Platform.Runtime` и `DMP.BuildingBlocks.*` не ссылаются на `DMP.Modules.Common`;
- платформенный object type может использовать Object Runtime без подключения Common-семантики.

Зависимость: `COM-03`.

### COM-05. Реализовать заполнение системных полей при create/update

Приоритет: `P0`  
Оценка: `L`

Состав работ:

- [x] добавить общий lifecycle hook Common;
- [x] при create устанавливать `CreatedAtUtc` и `CreatedBy`;
- [x] при update устанавливать `ModifiedAtUtc` и `ModifiedBy` только при фактическом изменении;
- [x] запретить изменение системных полей обычным mutation payload;
- [x] использовать единый clock и текущий идентификатор пользователя;
- [x] возвращать стабильную ошибку при отсутствии допустимого инициатора.

Критерии готовности:

- поля создания после create не меняются;
- при create поля изменения равны `null`;
- no-op update не меняет поля аудита;
- обычный пользователь не может подменить системные поля;
- `Guid.Empty` не записывается;
- отсутствие допустимого инициатора не подменяется неявным системным пользователем.

Зависимости: `COM-03`, `COM-04`.

### COM-06. Реализовать централизованную видимость удаленных объектов

Приоритет: `P0`  
Оценка: `L`

Состав работ:

- [x] применять `IsDeleted = false` к list, search, lookup и details;
- [x] применять фильтр к загрузке embedded/owned collections;
- [x] применять тот же системный scope к runtime-отчетам и другим стандартным query pathways Common;
- [x] добавить технический режим `IncludeDeleted`;
- [x] определить эквивалентный системный query context для custom readers;
- [x] ограничить использование `IncludeDeleted` административными и техническими сценариями.

Критерии готовности:

- обычные запросы не возвращают удаленные объекты;
- обычный details по идентификатору удаленного объекта возвращает not found;
- `IncludeDeleted = true` возвращает активные и удаленные записи;
- сочетание `IncludeDeleted = true` и `IsDeleted = true` возвращает только удаленные записи;
- `IncludeDeleted` не материализуется как обычный пользовательский фильтр.

Зависимость: `COM-04`.

### COM-07. Реализовать DeleteCapability

Приоритет: `P0`  
Оценка: `XL`

Состав работ:

- [x] запретить delete при `Disabled`;
- [x] реализовать заполнение полей при `SoftDelete`;
- [x] реализовать физическое удаление при `HardDelete`;
- [x] добавить отдельную проверку права `{ModuleCode}.{ObjectTypeCode}.Delete`;
- [x] не считать право `*.Edit` достаточным для удаления;
- [x] добавить object-specific validator/delete guard для проверки ссылочной безопасности hard delete;
- [x] добавить системное действие `__runtime_delete` с `StandardActionKind = Delete` и границей выполнения `ObjectMutation`;
- [x] согласовать самостоятельное удаление объекта с правилами удаления строк коллекции;
- [x] учесть workflow state policies.

Критерии готовности:

- delete невозможен без capability и отдельного права;
- soft delete не выполняет физическое удаление;
- hard delete невозможен при небезопасных ссылках;
- общий `ReferenceIntegrityService` не является блокером Common v1: при отсутствии доказанной безопасности hard delete запрещается object-specific guard-ом;
- удаление workflow-объекта не обходит правила состояния;
- операции попадают в Audit History;
- определены стабильные runtime issue codes.

Зависимости: `COM-04`, `COM-05`, `COM-06`.

### COM-08. Перевести Nomenclature на CommonCatalogObject

Приоритет: `P0`  
Оценка: `XL`

Состав работ:

- [x] изменить наследование `Nomenclature`;
- [x] заменить базовый runtime-контракт;
- [x] заменить stable codes наследуемых полей;
- [x] добавить Common-поля в persistence mapping;
- [x] обновить mutation mapping и read projection;
- [x] удалить `ExternalCode` из доменной и runtime-моделей;
- [x] использовать наследуемый `ExternalId`;
- [x] обновить baseline, формы, списки, фильтры и команды;
- [x] обновить архитектурные и интеграционные тесты.

Критерии готовности:

- `Nomenclature : CommonCatalogObject`;
- contract использует новый базовый контракт;
- в модели и runtime отсутствует `ExternalCode`;
- `ExternalId` доступен как поле Common;
- CRUD номенклатуры работает с новыми системными полями;
- tenant isolation и concurrency behavior не изменены.

Зависимости: `COM-01`–`COM-05`.

### COM-09. Подготовить целевую persistence-схему Nomenclature

Приоритет: `P0`  
Оценка: `M`

Состав работ:

- [x] добавить в persistence model все поля `CommonObject`;
- [x] задать максимальную длину `ExternalId` 100 символов;
- [x] удалить колонку `ExternalCode` из целевой схемы;
- [x] добавить обязательность и ограничения Common-полей сразу в целевой схеме;
- [x] обновить EF migrations и model snapshot для развертывания на пустой БД;
- [x] проверить создание новой БД с нуля.

Критерии готовности:

- новая БД создается без промежуточной legacy-схемы `BusinessEntity`/`ExternalCode`;
- таблица Nomenclature содержит целевые Common-поля и не содержит `ExternalCode`;
- `ExternalId` допускает `null` и ограничен 100 символами;
- обязательные Common-поля имеют корректные DB constraints;
- bootstrap-записи создаются через согласованный runtime/seeding-сценарий с допустимым инициатором.

Зависимость: `COM-08`.

### COM-11. Удалить BusinessEntity и старый object type

Приоритет: `P0`  
Оценка: `M`

Состав работ:

- [x] удалить код доменной модели, runtime contract и baseline `BusinessEntity`;
- [x] удалить `CommonCodes.BusinessEntity`;
- [x] удалить регистрацию object type;
- [x] заменить все конфигурационные ссылки;
- [x] обновить README и техническую документацию;
- [x] удалить или переписать тесты, закрепляющие старую модель.

Критерии готовности:

- в production-коде нет класса `BusinessEntity`;
- в публикуемых baseline-артефактах нет `Common:BusinessEntity`;
- `rg` по production-коду и актуальной документации не находит старый object type за исключением явно исторических документов;
- startup validation и импорт baseline проходят без alias;
- архитектурный тест запрещает повторное появление `Common:BusinessEntity`.

Зависимости: `COM-08`, `COM-09`.

### COM-12. Реализовать прямые действия Archive и Restore

Приоритет: `P1`  
Оценка: `L`

Состав работ:

- [x] добавить стандартные коды `Archive`, `Restore`, `Archived`;
- [x] реализовать handlers/lifecycle behavior;
- [x] проверять opt-in capability, права и объектные правила;
- [x] устанавливать или очищать архивные поля;
- [x] принимать необязательную причину архивирования и сохранять ее в `ArchiveReason`;
- [x] блокировать обычное изменение архивного объекта;
- [x] добавить Audit History.

Критерии готовности:

- объект без явно объявленных действий нельзя архивировать или восстановить;
- базовый контракт `CommonObject` не добавляет actions Archive/Restore автоматически;
- повторное действие обрабатывается предсказуемо;
- архивный объект доступен на просмотр по правам, но read-only;
- Archive и Restore не меняют поля soft delete;
- ошибки имеют стабильные коды.

Зависимости: `COM-04`, `COM-05`.

### COM-13. Реализовать системный фильтр архивных объектов

Приоритет: `P1`  
Оценка: `M`

Состав работ:

- [x] добавить `__runtime_include_archived`;
- [x] применять `IsArchived = false` по умолчанию;
- [x] материализовать фильтр «Показать архивные» для пользовательских списков Common;
- [x] применить поведение к search и lookup;
- [x] материализовать query parameter/filter metadata: boolean checkbox, `FilterPanel`, default `false`.

Критерии готовности:

- отсутствие фильтра эквивалентно `false`;
- `false` возвращает только неархивные объекты;
- `true` возвращает активные и архивные объекты;
- системный фильтр нельзя подменить обычным полем с тем же кодом.

Зависимости: `COM-04`, `COM-06`.

### COM-14. Разделить lookup и разрешение сохраненных ссылок

Приоритет: `P1`  
Оценка: `L`

Состав работ:

- [x] скрывать архивные объекты при выборе нового значения;
- [x] продолжать разрешать сохраненную ссылку на архивный объект;
- [x] исключать удаленные объекты из обычного lookup и details;
- [x] предусмотреть техническое разрешение удаленных ссылок для аудита и расследований.

Критерии готовности:

- архивное значение исчезает из вариантов выбора;
- существующая ссылка на него продолжает иметь display presentation;
- удаленное значение не открывается как рабочий объект;
- tenant и permission scopes применяются во всех режимах.

Зависимости: `COM-06`, `COM-13`.

### COM-15. Интегрировать архивное состояние с workflow

Приоритет: `P1`  
Оценка: `XL`

Состав работ:

- [x] добавить `IsArchiveState` в workflow state metadata;
- [x] добавить `IsArchiveState` в `ConfigurationSchemaPropertyCodes.WorkflowState`, schema, node builder, published configuration resolver и `WorkflowDefinitionState`;
- [x] при входе в архивное состояние устанавливать архивные поля Common;
- [x] при выходе очищать их;
- [x] запретить смешивание прямых Archive/Restore с workflow-архивом;
- [x] обеспечить согласованность transaction, workflow history и Audit History.

Критерии готовности:

- workflow является единственным источником архивного состояния для workflow-managed object type;
- `IsArchived` материализуется атомарно с переходом;
- startup validation отвергает смешанную конфигурацию;
- для Common v1 workflow содержит не более одного состояния с `IsArchiveState = true`, если несколько состояний не обоснованы отдельным решением object type;
- rollback перехода не оставляет частично обновленные Common-поля.

Зависимости: `COM-04`, `COM-05`, `COM-12`.

### COM-16. Добавить стабильные ошибки и расширить Audit History

Приоритет: `P1`  
Оценка: `M`

Состав работ:

- [x] определить issue codes для Common capabilities;
- [x] различать create, update, Archive, Restore, SoftDelete и HardDelete;
- [x] включать actor, object identity и change set;
- [x] для workflow-архива сохранять связь с переходом workflow.

Минимальный набор ошибок:

```text
ActorRequired
ArchiveNotSupported
RestoreNotSupported
ArchivedObjectReadOnly
DeleteDisabled
DeletePermissionRequired
DeletedObjectNotAvailable
HardDeleteReferenceUnsafe
CommonCapabilityConfigurationInvalid
```

Критерии готовности:

- runtime не возвращает только неструктурированные исключения для этих сценариев;
- audit-запись создается в той же transaction policy, что и изменение;
- audit не содержит `Guid.Empty` как actor.

Зависимости: `COM-05`, `COM-07`, `COM-12`, `COM-15`.

### COM-17. Реализовать ActorDisplayResolver для Common

Приоритет: `P1`  
Оценка: `M`

Состав работ:

- [x] добавить общий resolver отображаемого имени actor-а;
- [x] интегрировать его с IAM;
- [x] поддержать системных пользователей;
- [x] определить fallback для недоступного или исторического actor-а;
- [x] не сохранять display name в таблицах Common.

Критерии готовности:

- все четыре actor-поля могут отображаться человекочитаемо;
- изменение имени пользователя не требует обновления бизнес-таблиц;
- системные actor-ы, явно переданные вызывающим сценарием, отображаются как системные инициаторы;
- недоступность IAM не повреждает доменные данные.

### COM-18. Реализовать системную секцию «Администрирование»

Приоритет: `P1`  
Оценка: `L`

Состав работ:

- [x] автоматически добавлять секцию для Common-объектов;
- [x] показывать поля создания, изменения, архива и удаления;
- [x] разрешать actor display через `ActorDisplayResolver`;
- [x] реализовать object-level режим `AdministrationSectionDefault = Auto | Hidden`;
- [x] реализовать view-level режим `AdministrationSectionMode = Inherit | Auto | Hidden | Explicit`;
- [x] условно скрывать архивную группу для активного объекта и delete-группу вне `IncludeDeleted`/admin/recovery сценария.

Критерии готовности:

- секция появляется только в карточке просмотра;
- секция отсутствует в create/edit по умолчанию;
- требуется право просмотра технических данных;
- object type может явно скрыть секцию;
- `Explicit` отключает автоматическую секцию и позволяет полностью описать ее в View;
- системные поля остаются read-only.

Зависимости: `COM-03`, `COM-17`.

### COM-19. Обновить права и стандартные UI actions

Приоритет: `P1`  
Оценка: `M`

Состав работ:

- [x] зарегистрировать permissions Delete/Archive/Restore;
- [x] материализовать стандартные действия только при наличии capability и права;
- [x] скрывать или блокировать действия с объяснимой причиной;
- [x] не показывать обычное Edit для архивного объекта;
- [x] не публиковать IncludeDeleted как обычный UI-фильтр.

Критерии готовности:

- UI-проекция и серверная авторизация используют одинаковые правила;
- скрытие действия на UI не заменяет серверную проверку;
- права `Edit` и `Delete` независимы.

Зависимости: `COM-07`, `COM-12`, `COM-13`.

### COM-20. Добавить полный набор тестов Common v1

Приоритет: `P2`  
Оценка: `XL`

Состав работ:

- [x] unit-тесты доменной модели;
- [x] contract/descriptor tests;
- [x] persistence migration tests;
- [x] integration tests list/details/lookup;
- [x] mutation pipeline tests;
- [x] permission tests;
- [x] workflow archive tests;
- [x] audit tests;
- [x] архитектурные тесты удаления BusinessEntity;
- [x] архитектурные тесты направления зависимостей и отсутствия ссылки Platform.Runtime/BuildingBlocks на Modules.Common.

Обязательная матрица:

```text
active / archived / deleted
list / details / lookup / saved reference
default / includeArchived / includeDeleted
permission granted / denied
direct archive / workflow archive
soft delete / hard delete / disabled
human actor / missing actor / explicit system actor
```

Критерии готовности:

- тесты подтверждают все правила `03_behavior_rules.md`;
- тест создания пустой БД проверяет целевую схему, ограничения и отсутствие `ExternalCode`;
- regression tests подтверждают tenant isolation и concurrency Nomenclature.

Зависимости: все задачи `P0` и `P1`.

### COM-21. Обновить документацию и руководство миграции модулей

Приоритет: `P2`  
Оценка: `M`

Состав работ:

- [x] обновить README и актуальные runtime-документы;
- [x] описать выбор `CommonObject` или `CommonCatalogObject`;
- [x] описать persistence mapping и migrations;
- [x] описать opt-in Archive/Delete capabilities;
- [x] описать взаимодействие с tenant, workflow, IAM и Audit History;
- [x] описать границу применения Common к платформенным и системным сущностям.

Критерии готовности:

- актуальная документация не представляет `BusinessEntity` как действующую модель;
- новые модули могут внедрить Common без копирования реализации Nomenclature;
- historical/archive документы явно не считаются текущим руководством.

Зависимость: `COM-20`.

### COM-F01. Ввести обязательный Actor Context

Приоритет: `Future`  
Оценка: `XL`

Желаемая целевая модель:

```text
IActorContext
  ActorId
  ActorKind
  OnBehalfOfUserId?
```

Состав будущей доработки:

- [ ] отличать пользователя, service principal и системный процесс;
- [ ] требовать явный actor для каждой mutation;
- [ ] поддержать `OnBehalfOfUserId` для фоновых операций, запущенных пользователем;
- [ ] зарегистрировать отдельные identities для интеграций, scheduler и background workers;
- [ ] исключить неявные подстановки системного пользователя.

До выполнения этой задачи каждый системный сценарий обязан явно передавать поддерживаемого инициатора существующим runtime-контекстом.

### COM-F02. Реализовать общий ReferenceIntegrityService

Приоритет: `Future`  
Оценка: `XL`

Состав будущей доработки:

- [ ] построить общий поиск входящих ссылок на основе object contracts;
- [ ] предоставлять delete guard-ам единый результат проверки ссылочной безопасности;
- [ ] учитывать tenant scope, workflow history, Audit History и integration mappings;
- [ ] определить расширяемую политику Restrict/Cascade для root hard delete.

До выполнения этой задачи Common v1 использует object-specific validator/delete guard и запрещает hard delete при сомнении в ссылочной безопасности.

## 6. Рекомендуемая последовательность реализации

```text
COM-01 → COM-02 → COM-03 → COM-04
                           ├─ COM-05
                           ├─ COM-06 → COM-07
                           └─ COM-08

COM-08 → COM-09 → COM-11
COM-04/05 → COM-12 → COM-13 → COM-14
                    └────────→ COM-15

COM-05/07/12/15 → COM-16
COM-17 → COM-18
COM-07/12/13 → COM-19

COM-11/14/15/16/18/19 → COM-20 → COM-21

Future: COM-F01, COM-F02
```

## 7. Предлагаемые инкременты поставки

### Инкремент 1. Модель и контракты

```text
COM-01 — COM-04
```

Результат: новые базовые типы и descriptor metadata доступны для прикладных модулей.

### Инкремент 2. Безопасные mutations и delete policy

```text
COM-05 — COM-07
```

Результат: системные поля заполняются централизованно, удаленные объекты скрываются, безусловный delete закрыт capability-политикой.

### Инкремент 3. Breaking migration Nomenclature

```text
COM-08, COM-09, COM-11
```

Результат: Nomenclature работает на `CommonCatalogObject`, пустая БД создается сразу в целевой схеме, `BusinessEntity` полностью удален.

Инкремент должен поставляться атомарно: состояние, в котором baseline уже ссылается на новый object type, а код или БД еще используют `BusinessEntity`, недопустимо.

### Инкремент 4. Архивирование и runtime presentation

```text
COM-12 — COM-19
```

Результат: реализовано полное поведение архива, ссылок, workflow, аудита, actor display и UI Common.

### Инкремент 5. Завершение Common v1

```text
COM-20 — COM-21
```

Результат: полная тестовая матрица и документация для внедрения Common в другие модули.

## 8. Definition of Done Common v1

Common v1 считается завершенным, когда:

- реализованы `CommonObject` и `CommonCatalogObject`;
- `BusinessEntity` и `Common:BusinessEntity` полностью удалены без alias;
- `Nomenclature` мигрирована на `CommonCatalogObject`;
- `ExternalCode` удален, а новая модель использует только `ExternalId`;
- новая система разворачивается на пустой БД сразу с целевой persistence-схемой;
- системные поля создаются и изменяются только runtime-механизмами;
- Platform.Runtime и BuildingBlocks не зависят от Modules.Common;
- архивные и удаленные объекты фильтруются централизованно;
- Archive, Restore и Delete доступны только через явно объявленные capabilities и отдельные права;
- workflow archive не смешивается с прямыми Archive/Restore;
- сохраненные ссылки на архивные объекты продолжают отображаться;
- Audit History различает все значимые операции Common;
- секция «Администрирование» и actor display работают по правилам документации;
- тесты покрывают доменную модель, runtime, persistence, permissions, workflow, audit и breaking migration;
- актуальная документация не содержит `BusinessEntity` как действующей архитектурной модели.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 11:50 +03:00 | Олег Юрьев (@axelprosoft) | 1. Миграция BusinessEntity; Инкремент 1. Модель и контракты | мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [4cbd3069](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/4cbd3069ec4a3421adbad1f0c9801b00f28405d7) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-06 14:02 +04:00 | Александр Жигалин | Содержание документа | Создание документа | [7407b480](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7407b480b0e62dfe5d077171d6bf74c126df33aa) |
