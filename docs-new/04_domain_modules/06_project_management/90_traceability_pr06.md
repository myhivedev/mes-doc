---
id: DOC-04-06-90
title: 'Трассировка ПР06 - 06 Управление проектами'
type: requirement
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers: []
scope: domain
module: 06_project_management
holder: '@axelprosoft'
created_at: 2026-09-01 00:00
created_by: '@axelprosoft'
updated_at: 2026-09-03 10:01
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Трассировка ПР06 - 06 Управление проектами

## 1. Назначение документа

Документ связывает исходное проектное решение, обязательные источники, DMP_DATA и принятые решения PR06 с нормативными документами модуля.

## 2. Источники

| Источник | Использование |
|---|---|
| `DMP ПР06 Управление проектами_исх. ТР v0.1.docx` | Исходные требования PR06, варианты использования и ограничения MVP. |
| `../../00_governance/requirements/001_dmp_functional_requirements_and_constraints.md` | Обязательные функциональные требования и ограничения DMP. |
| `../../00_governance/00_documentation_strategy.md` | Структура и правила подготовки документации. |
| `00_module_documentation_template.md` | Обязательная структура документов прикладного модуля. |
| `00_common` | CommonObject, CommonCatalogObject, архивирование, мягкое удаление, Presentation и Object Runtime. |
| `03_platform` | Платформенные контракты Object Runtime, lookup, права, tenant-граница и concurrency. |
| `01_general_master_data` | Граница общего контура данных; внешние потребители могут ссылаться на `ProjectManagement:CostCode`, но каталогом владеет PR06. |
| `03_plant_structure` | `ProductionUnit`, `OrganizationalUnit`, `Subcontractor` и правило общего типа ссылки исполнителя. |
| `04_resource_management` | `Personnel` как внешний объект ответственного сотрудника. |
| `05_document_management` | Document links и владение документами и файлами. |
| `07_order_management` | Использование проектов и этапов в заказах; владение проектами перенесено в PR06. |
| `08_planning_scheduling` | Будущий потребитель проектов и этапов в планировании. |
| `C:\@WORK\@СИГМА\01 РАЗРАБОТКА\02 Требования\01 Проектные решения\DMP_DATA\DMP` | Структура данных `ProjectBase`, `Project`, `ProjectPhase` и исходное поле `Status`. |
| Исходный код DMP | Проверка паттернов Object Runtime, прав, lookup и стандартных операций. |

## 3. Требования и решения ПР

| Требование | Решение | Статус | Документ |
|---|---|---|---|
| `REQ-06-B-001` | PR06 ведет `Project` и `ProjectPhase`; контрольные точки исключены из MVP и относятся к будущему плану проекта. | принято частично | [01_scope.md](01_scope.md), [02_domain_model.md](02_domain_model.md) |
| `REQ-06-B-002` | Высокоуровневый план с датами и результатами не моделируется в MVP. | отложено после MVP | [01_scope.md](01_scope.md), [backlog.md](backlog.md) |
| `REQ-06-B-003` | PR06 ведет самостоятельный tenant-owned каталог `CostCode` с иерархией; прямой связи с `Project` и `ProjectPhase` нет. | принято в PR06 | [01_scope.md](01_scope.md), [02_domain_model.md](02_domain_model.md) |
| `REQ-06-B-004` | Проект и этап предоставляются как внешние аналитические ссылки; поля ссылок принадлежат потребляющим модулям. | принято частично | [03_object_runtime_model.md](03_object_runtime_model.md), [05_rules.md](05_rules.md) |
| `REQ-06-B-005` | Версии базового плана не входят в MVP. | отложено после MVP | [01_scope.md](01_scope.md), [backlog.md](backlog.md) |
| `REQ-06-B-006` | Прикладной контроль состояния проекта и этапа не входит в MVP. | отложено после MVP | [01_scope.md](01_scope.md), [backlog.md](backlog.md) |

## 4. Сущности ПР и целевая модель

| Сущность ПР / DMP_DATA | Целевое решение | Статус | Документ |
|---|---|---|---|
| `ProjectBase` | Логический общий предок `Project` и `ProjectPhase`; отдельный runtime object type не создается. | принято | [02_domain_model.md](02_domain_model.md) |
| `Project` | Самостоятельный объект PR06 и владелец верхнего уровня структуры проекта. | принято | [02_domain_model.md](02_domain_model.md) |
| `ProjectPhase` | Самостоятельный объект PR06 с обязательным `Project` и необязательным `Parent`. | принято | [02_domain_model.md](02_domain_model.md) |
| `ProjectPhase.Parent` | Сохраняется как ссылка на этап того же проекта; циклы запрещены. | принято | [05_rules.md](05_rules.md) |
| `ProductionUnitResponsible` | Переименовано в `ResponsibleProductionUnit`, ссылка на `ProductionUnit`. | принято | [02_domain_model.md](02_domain_model.md) |
| `ProductionUnitExecutor` | Заменено на `ExecutorOrganizationalUnit`, ссылка на `OrganizationalUnit`. | принято | [02_domain_model.md](02_domain_model.md) |
| `ResponsiblePerson` | Ссылка на `Personnel`, поле необязательно. | принято | [02_domain_model.md](02_domain_model.md) |
| `Status` | Не переносится; прикладной workflow проекта и этапа в MVP отсутствует. | отклонено | [02_domain_model.md](02_domain_model.md), [10_value_set_data_usage.md](10_value_set_data_usage.md) |
| `Presentation` | Не переносится как предметное поле; формируется Object Runtime. | отклонено | [02_domain_model.md](02_domain_model.md), [03_object_runtime_model.md](03_object_runtime_model.md) |
| Контрольная точка | Не является типом этапа и не входит в MVP; будущая часть плана проекта. | отложено | [01_scope.md](01_scope.md), [backlog.md](backlog.md) |
| `CostCode` | Самостоятельный объект PR06, наследник `CommonCatalogObject`, с обязательным `TenantId` и `Parent -> CostCode`; прямые поля в `Project` и `ProjectPhase` не создаются. | принято | [02_domain_model.md](02_domain_model.md), [05_rules.md](05_rules.md) |

## 5. Принятые решения

| Решение | Суть | Документ |
|---|---|---|
| Граница MVP | Только проекты и структурные этапы как аналитические объекты. | [01_scope.md](01_scope.md) |
| Общий предок | `ProjectBase` остается логическим предком без отдельного runtime type и общей пользовательской таблицы. | [02_domain_model.md](02_domain_model.md) |
| Контрольные точки | Не вводятся сейчас; будут частью будущего плана проекта с работами и связями. | [01_scope.md](01_scope.md) |
| Согласованность аналитик | Этап выбирается только из выбранного проекта; при выборе этапа проект подставляется. | [05_rules.md](05_rules.md) |
| Исполнитель | Субподрядчик допускается через `ExecutorOrganizationalUnit -> OrganizationalUnit`. | [02_domain_model.md](02_domain_model.md) |
| Workflow | Для `Project`, `ProjectPhase` и `CostCode` в MVP не вводится прикладной workflow; используется стандартное архивирование Object Runtime. | [03_object_runtime_model.md](03_object_runtime_model.md), [05_rules.md](05_rules.md) |
| `CostCode` | PR06 владеет самостоятельным tenant-owned каталогом `CostCode` с иерархией, отдельным UI, правами и меню; внешние модули используют прямые ссылки на него. Связь с `Project` и `ProjectPhase` не создается. | [02_domain_model.md](02_domain_model.md), [03_object_runtime_model.md](03_object_runtime_model.md), [06_ui_views.md](06_ui_views.md), [11_permissions.md](11_permissions.md), [15_navigation_menu.md](15_navigation_menu.md) |
| Архивирование и удаление | Внешние ссылки не блокируют архивирование, но блокируют удаление; внутренние активные дочерние объекты блокируют архивирование. | [05_rules.md](05_rules.md) |
| UI | Один пункт `Проекты`, дерево этапов внутри карточки проекта. | [06_ui_views.md](06_ui_views.md), [15_navigation_menu.md](15_navigation_menu.md) |
| Права | Отдельные стандартные permissions для `Project` и `ProjectPhase`. | [11_permissions.md](11_permissions.md) |
| Владение | PR06 владеет проектами и этапами; PR07 использует внешние ссылки. | [01_scope.md](01_scope.md) |
| Документы | Создание документа исключено; прикрепление существующего документа выполняется через PR05. | [01_scope.md](01_scope.md), [13_operations.md](13_operations.md) |

## 6. Бизнес-правила

| Правило | Документ |
|---|---|
| Все объекты и ссылки принадлежат текущему предприятию. | [05_rules.md](05_rules.md) |
| Проект и этап не образуют противоречивую пару. | [05_rules.md](05_rules.md), [13_operations.md](13_operations.md) |
| Родительский этап принадлежит тому же проекту; самоссылки и циклы запрещены. | [05_rules.md](05_rules.md) |
| `CostCode` образует tenant-ограниченное дерево; самоссылки, циклы и межtenant-ссылки запрещены. | [02_domain_model.md](02_domain_model.md), [05_rules.md](05_rules.md) |
| Ответственность и исполнитель необязательны и не наследуются автоматически. | [02_domain_model.md](02_domain_model.md), [05_rules.md](05_rules.md) |
| Архивные объекты не выбираются заново, но отображаются по сохраненным ссылкам. | [03_object_runtime_model.md](03_object_runtime_model.md), [06_ui_views.md](06_ui_views.md) |
| Удаление блокируется при внутренних и внешних зависимостях. | [05_rules.md](05_rules.md), [13_operations.md](13_operations.md) |

## 7. UI-решения

В MVP создаются пункты меню `Проекты` и `Коды затрат`, список проектов, карточка проекта с деревом этапов, карточка этапа, список и карточка `CostCode`, дерево кодов затрат, lookup проекта, этапа и кода затрат. Глобальный список этапов, календарный план, контрольные показатели и интерфейс workflow не создаются.

Подробнее: [06_ui_views.md](06_ui_views.md) и [15_navigation_menu.md](15_navigation_menu.md).

## 8. Открытые вопросы

Открытых вопросов, блокирующих проектирование MVP PR06, нет.

В backlog перенесены будущие области, которые не являются незакрытыми решениями текущего MVP:

- проектный план с работами, связями, сроками и план-фактными показателями;
- baseline и версии вариантов плана;
- контрольные точки как элементы плана;
- прикладной workflow проекта и этапа;
- отдельные назначения `CostCode` к проекту или этапу, если такая потребность появится;
- отдельный глобальный список этапов при появлении потребности.

## 9. Куда перенесены решения

| Документ | Что содержит |
|---|---|
| [00_module_overview.md](00_module_overview.md) | Карту модуля, объекты, ключевые решения и зависимости. |
| [01_scope.md](01_scope.md) | Границы владения, исключения и покрытие требований. |
| [02_domain_model.md](02_domain_model.md) | Объекты, поля, связи, логическое наследование и отличия DMP_DATA. |
| [03_object_runtime_model.md](03_object_runtime_model.md) | Runtime-типы, lookup, представления, иерархию и платформенные возможности. |
| [05_rules.md](05_rules.md) | Инварианты, согласованность, архивирование и удаление. |
| [06_ui_views.md](06_ui_views.md) | Списки, карточки, дерево и фильтры. |
| [10_value_set_data_usage.md](10_value_set_data_usage.md) | Отсутствие модульных SystemEnum и ValueSet в MVP. |
| [11_permissions.md](11_permissions.md) | Ресурсы прав, роли и матрицу доступа. |
| [13_operations.md](13_operations.md) | Составные проверки и операции дерева и зависимостей. |
| [15_navigation_menu.md](15_navigation_menu.md) | Пункт главного меню и представления. |
| [backlog.md](backlog.md) | Расширения после MVP. |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-03 10:01 +03:00 | Олег Юрьев (@axelprosoft) | Источники; Требования и решения ПР; Сущности ПР и целевая модель; Принятые решения; Бизнес-правила; UI-решения; Открытые вопросы | первые версии модулей 04, 06. правки связанной документации - шаблон и предложения по изменениям ФТ (определения терминов) | [766441e6](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/766441e68e531dadabe887393fc989e5b6e268e0) |
| 1.0 | 2026-09-02 12:21 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #56: мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [PR #56](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/56) |
| 0.1 | 2026-09-02 12:19 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs: exclude working files from versioning | [b04af06e](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/b04af06e9d1fa719ffa8d242a9ee7d33d7d0f30e) |
