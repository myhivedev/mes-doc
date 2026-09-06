---
id: DOC-04-04-90
title: 'Трассировка требований ПР04 - 04 Управление ресурсами'
type: requirement
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers: []
scope: domain
module: 04_resource_management
holder: '@axelprosoft'
created_at: 2026-09-03 08:57
created_by: '@axelprosoft'
updated_at: 2026-09-03 10:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Трассировка требований ПР04 - 04 Управление ресурсами

## 1. Назначение документа

Документ связывает исходное проектное решение ПР04, структуру `DMP_DATA`,
обязательные источники и принятые решения с целевыми документами модуля
`04_resource_management`.

Трассировка показывает, где зафиксировано каждое решение и почему отдельные
исходные элементы не переносятся в целевую модель. Она не заменяет описания
объектов, правил, runtime, UI и операций в соответствующих документах.

## 2. Источники

| Источник | Использование |
|---|---|
| `C:\@WORK\@СИГМА\01 РАЗРАБОТКА\02 Требования\01 Проектные решения\DMP ПР1\DMP ПР04 Управление ресурсами.docx` | Исходные объекты, поля, формы, варианты использования, бизнес-операции и алгоритмы ПР04. |
| `C:\@WORK\@СИГМА\01 РАЗРАБОТКА\02 Требования\01 Проектные решения\DMP_DATA` | Логическая и физическая структура исходных данных, связи и исходное наследование. |
| `docs-new/00_governance/00_documentation_strategy.md` | Стратегия состава и границ проектной документации. |
| `docs-new/00_governance/requirements/001_dmp_functional_requirements_and_constraints.md` | Обязательные функциональные требования и ограничения DMP. |
| `docs-new/04_domain_modules/00_module_documentation_template.md` | Структура и содержание документов прикладного модуля. |
| `docs-new/04_domain_modules/00_common` | Общие типы, архивирование, удаление, права, коллекции и Object Runtime. |
| `docs-new/03_platform` | Платформенные контракты Object Runtime, tenant, lookup, baseline и безопасность. |
| `docs-new/04_domain_modules/01_general_master_data` | Внешние справочники и номенклатурные ссылки. |
| `docs-new/04_domain_modules/02_product_process_definition` | Нормативные ресурсные ссылки, условия выбора и альтернативные наборы PR02. |
| `docs-new/04_domain_modules/03_plant_structure` | `ProductionUnit`, `OrganizationalUnit`, `Subcontractor` и производственная структура. |
| `docs-new/04_domain_modules/05_document_management` | Граница документов и связанных файлов. |
| `docs-new/04_domain_modules/06_project_management` | Межмодульные ссылки и единый стандарт описания проектных модулей. |
| Исходный код DMP | Проверка фактических паттернов контрактов, permissions, lookup и runtime. |
| [`_working/decision_log.md`](_working/decision_log.md) | Рабочие решения, расхождения и открытый вопрос по `ShiftRotationModel`. |

## 3. Граница модуля

| Область ПР04 | Решение для 04 | Статус | Где зафиксировано |
|---|---|---|---|
| Рабочие места и персонал | Модуль владеет `WorkPlace` и `Personnel`, их группами, местоположениями и связанными данными. | Входит | [`01_scope.md`](01_scope.md), [`02_domain_model.md`](02_domain_model.md) |
| Графики и календари | Модуль владеет `Holiday`, `DayType`, `WorkSchedule`, описаниями и `ShiftRotationModel`. | Входит | [`01_scope.md`](01_scope.md), [`02_domain_model.md`](02_domain_model.md) |
| Назначения графиков | Модуль владеет назначениями графиков конкретным ресурсам, группам и внешнему `OrganizationalUnit`. | Входит с межмодульным контекстом | [`01_scope.md`](01_scope.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| Инструмент и оснастка | Модуль владеет самостоятельными типами `Tooling` и `Gage`. | Входит | [`01_scope.md`](01_scope.md), [`02_domain_model.md`](02_domain_model.md) |
| Параметры операций | Модуль владеет настройками параметров `ProductionUnit`, `WorkPlace` и `Personnel`; PR10 использует эффективные значения. | Входит с границей | [`01_scope.md`](01_scope.md), [`13_operations.md`](13_operations.md) |
| Материально ответственное лицо | Модуль владеет назначением `Personnel` на `ProductionUnit`; складские операции остаются во внешнем контуре. | Входит с границей | [`01_scope.md`](01_scope.md), [`05_rules.md`](05_rules.md) |
| Фактическое движение инструмента | Выдача, возврат, ремонт, списание и эксплуатация инструмента не входят в 04. | Не входит | [`01_scope.md`](01_scope.md), [`_working/decision_log.md`](_working/decision_log.md) |
| Автоматический подбор ресурсов | Подбор ресурсов для операции и согласованные альтернативные наборы принадлежат PR02, планированию или операционному контуру. | Не входит | [`01_scope.md`](01_scope.md), [`13_operations.md`](13_operations.md) |

## 4. Требования и сущности исходного ПР04

| Элемент ПР04 / DMP_DATA | Целевое решение | Статус | Документ |
|---|---|---|---|
| `ResourceBase` | Логический абстрактный тип для общих свойств `WorkPlace`, `Personnel`, `Tooling` и `Gage`; отдельный объект и общий lookup ресурсов не создаются. | Принято | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `WorkPlace` | Самостоятельный объект рабочего места с внешней ссылкой на `EquipmentUnit`, если она задана. | Входит | [`02_domain_model.md`](02_domain_model.md) |
| `Personnel` | Самостоятельный объект сотрудника; логически наследует общие свойства ресурса. | Входит | [`02_domain_model.md`](02_domain_model.md) |
| `ResourceGroupBase` | Логический базовый тип групп; `Parent` раскрывается в конкретных однотипных группах. | Принято | [`02_domain_model.md`](02_domain_model.md), [`05_rules.md`](05_rules.md) |
| `WorkPlaceGroup` | Самостоятельная иерархическая группа рабочих мест. | Входит | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `PersonnelGroup` | Самостоятельная иерархическая группа сотрудников. | Входит | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `ResourceAllocationType` и `AllocationType` | 04 определяет тип распределения группы и проверяет его согласованность с составом; количество, альтернативные наборы и фактический подбор принадлежат PR02. | Принято с границей | [`02_domain_model.md`](02_domain_model.md), [`05_rules.md`](05_rules.md), [`_working/decision_log.md`](_working/decision_log.md) |
| `ResourceInResourceGroupBase` | Логический общий контракт элементов членства; отдельный тип и общий lookup не создаются. | Принято | [`02_domain_model.md`](02_domain_model.md) |
| `WorkPlaceInWorkPlaceGroup`, `PersonnelInPersonnelGroup` | Элементы коллекций соответствующих групп. | Входит через владельца | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `ResourceLocationBase` | Логический общий тип местоположения; самостоятельные типы `WorkPlaceLocation` и `PersonnelLocation` сохраняются раздельно. | Принято | [`02_domain_model.md`](02_domain_model.md) |
| `WorkPlaceLocation`, `PersonnelLocation` | История местоположений конкретного рабочего места или сотрудника. `Previous` связывает только основные постоянные места одного типа и одного ресурса. | Входит | [`05_rules.md`](05_rules.md), [`13_operations.md`](13_operations.md) |
| `MaterialResponsiblePerson` | Назначение сотрудника материально ответственным лицом для производственной единицы без ссылки на место хранения. | Входит с границей | [`02_domain_model.md`](02_domain_model.md), [`05_rules.md`](05_rules.md) |
| `Profession`, `PaymentGroup`, `PersonnelAllowanceType`, `AbsenceReason` | Самостоятельные справочники модуля 04, используемые в данных персонала. | Входит | [`02_domain_model.md`](02_domain_model.md), [`10_value_set_data_usage.md`](10_value_set_data_usage.md) |
| `PersonnelQualification`, `PersonnelAllowance`, `PersonnelAbsence` | Данные сотрудника, публикуемые через коллекции `Personnel`. | Входит через владельца | [`02_domain_model.md`](02_domain_model.md), [`05_rules.md`](05_rules.md) |
| `ToolBase` | Логический абстрактный reference-only тип для полиморфной ссылки PR02. | Принято | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `Tooling`, `Gage` | Два самостоятельных типа объектов с раздельными карточками и представлениями. Для внешней ссылки PR02 используется общий lookup `ToolBase`. | Входит | [`02_domain_model.md`](02_domain_model.md), [`06_ui_views.md`](06_ui_views.md) |
| `Holiday`, `DayType`, `DayTypeDescription` | Календарные объекты и описания типов рабочих дней. | Входит | [`02_domain_model.md`](02_domain_model.md), [`05_rules.md`](05_rules.md) |
| `WorkSchedule`, `WorkScheduleDescription` | График и его описание; график выбирает тип рабочего дня по календарному правилу. | Входит | [`02_domain_model.md`](02_domain_model.md), [`13_operations.md`](13_operations.md) |
| `ShiftRotationModel`, `ShiftRotationDescription` | Отдельная модель чередования смен и ее описания. Полная совместимость модели с назначением сотруднику остается открытой. | Входит с открытым вопросом | [`02_domain_model.md`](02_domain_model.md), [`_working/decision_log.md`](_working/decision_log.md) |
| `ResourceWorkSchedule` | Логический базовый тип назначения; самостоятельная общая запись и lookup не создаются. | Принято | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `OrganizationalUnitWorkSchedule` | Единый целевой тип назначения внешнему `OrganizationalUnit`; конкретный вид определяется внешним контрактом PR03. | Входит с межмодульным контекстом | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `WorkPlaceGroupWorkSchedule`, `WorkPlaceWorkSchedule` | Раздельные назначения графиков группам рабочих мест и рабочим местам. | Входит через владельца | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `PersonnelGroupWorkSchedule`, `PersonnelWorkSchedule` | Раздельные назначения графиков группам сотрудников и сотрудникам. | Входит через владельца | [`02_domain_model.md`](02_domain_model.md), [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `OperationParametersBase` | Логический общий тип параметров. | Принято | [`02_domain_model.md`](02_domain_model.md) |
| `ProductionUnitOperationParameters` | Настройки операций производственной единицы; данные принадлежат 04 и доступны из контекста PR03. | Входит с межмодульным контекстом | [`01_scope.md`](01_scope.md), [`13_operations.md`](13_operations.md) |
| `WorkPlaceOperationParameters`, `PersonnelOperationParameters` | Одиночные настройки операций рабочего места и сотрудника с наследованием эффективных значений от `ProductionUnit`. | Входит | [`02_domain_model.md`](02_domain_model.md), [`13_operations.md`](13_operations.md) |
| Причины отсутствия и отсутствие сотрудника | `AbsenceReason` является самостоятельным справочником; `PersonnelAbsence` входит в коллекцию сотрудника. | Входит | [`02_domain_model.md`](02_domain_model.md), [`05_rules.md`](05_rules.md) |

## 5. Функции, UI и алгоритмы ПР04

| Содержание ПР04 | Куда перенесено | Статус |
|---|---|---|
| Списки, карточки и вкладки рабочих мест, групп, персонала, графиков и справочников | [`06_ui_views.md`](06_ui_views.md) | Перенесено с принятыми корректировками |
| Меню модуля и связь пунктов с представлениями | [`15_navigation_menu.md`](15_navigation_menu.md) | Перенесено в целевую структуру |
| Валидация полей, периодов, иерархий и ссылочной целостности | [`05_rules.md`](05_rules.md) | Перенесено как проверяемые правила |
| Назначение и расчет эффективного графика | [`13_operations.md`](13_operations.md) | Перенесено с принятой иерархией владельцев |
| Корректировка истории основных местоположений | [`13_operations.md`](13_operations.md) | Перенесено с решением по `Previous` и открытой границе периода |
| Чтение и разрешение эффективных параметров операций | [`13_operations.md`](13_operations.md) | Перенесено |
| Lookup и проверки ссылок для PR02 | [`03_object_runtime_model.md`](03_object_runtime_model.md), [`13_operations.md`](13_operations.md) | Перенесено как межмодульный контракт |
| Общие REST-требования и формат ошибок | Common и `03_platform` | Не дублируется в модуле |

## 6. Прикладная операция исходного ПР04, исключенная из v1

ПР04 описывает автоматическое создание рабочих мест по внешним объектам
оборудования. Алгоритм должен был выбрать внешние объекты по производственной
единице или конкретному объекту, исключить уже связанные с рабочим местом,
создать рабочее место и основное постоянное местоположение.

Операция не входит в целевую модель 04 v1. Не подтверждены владелец внешнего
справочника оборудования, окончательный межмодульный контракт и правила
сопоставления. В текущей версии можно только сохранить ссылку на уже
существующий `EquipmentUnit` в `WorkPlace`; автоматическое создание и
синхронизация не выполняются.

Это не является пропуском при переносе требований: решение об исключении
зафиксировано в [`_working/decision_log.md`](_working/decision_log.md) и
отражено в [`01_scope.md`](01_scope.md) и [`13_operations.md`](13_operations.md).

## 7. Ключевые принятые решения

| Решение | Суть | Основание и документ |
|---|---|---|
| Граница ресурсов | `WorkPlace`, `Personnel`, группы и связанные данные принадлежат 04; движение ресурсов не принадлежит 04. | ПР04, Q1; [`01_scope.md`](01_scope.md) |
| Логическое наследование | Базовые типы описывают общие свойства и контракты; самостоятельные записи создаются только конкретными типами. | ПР04, DMP_DATA, Q2/Q3; [`02_domain_model.md`](02_domain_model.md) |
| Общий ресурсный lookup | Общий lookup `Resource` для рабочих мест, персонала и инструментов не создается. | Q3/Q15; [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| `ToolBase` | `ToolBase` является reference-only контрактом; общий lookup объединяет только `Tooling` и `Gage`, которые имеют отдельные карточки и представления. | ПР04, DMP_DATA, Q3/Q12; [`02_domain_model.md`](02_domain_model.md) |
| Группы | `WorkPlaceGroup` и `PersonnelGroup` являются разными типами; `Parent` ссылается только на группу того же типа. | Q3; [`05_rules.md`](05_rules.md) |
| Местоположения | `WorkPlaceLocation` и `PersonnelLocation` раздельны; `Previous` используется только для цепочки основных постоянных мест. | ПР04, Q8; [`13_operations.md`](13_operations.md) |
| Открытая граница периода | В целевой модели `ValidTo = null`; максимальная техническая дата из исходной модели не используется. | Q8; [`05_rules.md`](05_rules.md) |
| Графики | `WorkSchedule` и его назначения являются разными объектами; назначения конкретных владельцев остаются раздельными. | ПР04, DMP_DATA, Q7; [`02_domain_model.md`](02_domain_model.md) |
| `AssignmentType` | `Permanent`, `Temporary` и `Change` являются значениями назначения; отдельный объект изменения не создается. | ПР04, Q7; [`05_rules.md`](05_rules.md), [`13_operations.md`](13_operations.md) |
| `ShiftRotationModel` | Модель остается отдельным объектом и применяется только к сотруднику или группе сотрудников; совместимость с назначением пока не определена полностью. | ПР04, DMP_DATA, Q7; [`_working/decision_log.md`](_working/decision_log.md) |
| Параметры операций | Владелец настроек - 04; PR10 только использует эффективные значения при управлении операциями. | ПР04, Q11; [`01_scope.md`](01_scope.md), [`13_operations.md`](13_operations.md) |
| МОЛ | `MaterialResponsiblePerson` остается назначением сотрудника на `ProductionUnit`; складские операции выполняются внешним контуром. | ПР04, Q10; [`05_rules.md`](05_rules.md) |
| SystemEnum и ValueSet | Фиксированные значения ПР04 описываются как SystemEnum; собственные `ValueSetData` в v1 не вводятся. | ПР04, DMP_DATA, Q14; [`10_value_set_data_usage.md`](10_value_set_data_usage.md) |
| Workflow | Прикладной workflow 04 v1 не создается; используются стандартные механизмы Common/Object Runtime. | Q7/Q13; [`03_object_runtime_model.md`](03_object_runtime_model.md) |
| Права | Роли разделены по предметным веткам, область действия - tenant; вложенные коллекции защищаются через владельца. | Q13; [`11_permissions.md`](11_permissions.md) |

## 8. Межмодульные зависимости

| Модуль или компонент | Что предоставляет или получает | Граница |
|---|---|---|
| Common | Общие поля, Object Runtime, архивирование, удаление, tenant и стандартные права. | Common не владеет прикладными таблицами 04 и не определяет предметные правила ресурсов. |
| PR01 / GMD | Внешние справочные объекты и номенклатурные ссылки. | 04 не изменяет объекты GMD. |
| PR02 | Нормативные ссылки на ресурсы, количество, условия применения и альтернативные наборы. | 04 предоставляет lookup и проверки отдельных ссылок; подбор и нормы принадлежат PR02. |
| PR03 | `ProductionUnit`, `OrganizationalUnit`, `Subcontractor` и их доступность. | 04 хранит свои назначения и параметры с внешними ссылками; PR03 не владеет настройками 04. |
| PR05 | Документы и файлы, если они появятся во внешних сценариях. | 04 не создает собственную документную модель. |
| PR09 | Использование назначения МОЛ в складском и производственном учете. | 04 не владеет складскими объектами и движением запасов. |
| PR10 | Использование эффективных параметров операций. | PR10 не изменяет конфигурацию параметров 04. |
| APS и планирование | Возможное использование сведений о графиках и ресурсах. | Автоматический подбор ресурсов не принадлежит 04. |

## 9. Открытые вопросы

| Вопрос | Статус | Влияние | Источник фиксации |
|---|---|---|---|
| Как `ShiftRotationModel` совместно используется с `WorkSchedule` в назначении `Personnel` или `PersonnelGroup`? | Открыт | Определяет правила выбора модели, наследования и расчета эффективной смены. | [`_working/decision_log.md`](_working/decision_log.md), вопрос 7 |

До ответа архитектора модель не расширяется дополнительными полями или
правилами наследования `ShiftRotationModel`. Остальные решения, отраженные в
целевых документах, считаются принятыми для версии v1.

## 10. Распределение решений по документам

| Документ | Содержание |
|---|---|
| [`00_module_overview.md`](00_module_overview.md) | Назначение, состав модуля и основные зависимости. |
| [`01_scope.md`](01_scope.md) | Граница владения, исключения и покрытие требований. |
| [`02_domain_model.md`](02_domain_model.md) | Объекты, поля, логическое наследование, ссылки и коллекции. |
| [`03_object_runtime_model.md`](03_object_runtime_model.md) | Runtime-типы, lookup, коллекции, обработчики, иерархии и baseline. |
| [`05_rules.md`](05_rules.md) | Инварианты, уникальности, периоды, архивирование и удаление. |
| [`06_ui_views.md`](06_ui_views.md) | Списки, карточки, вкладки, фильтры и формы строк. |
| [`10_value_set_data_usage.md`](10_value_set_data_usage.md) | Отсутствие собственных `ValueSetData` и правила фиксированных enum. |
| [`11_permissions.md`](11_permissions.md) | Роли, защищаемые объекты и правила доступа. |
| [`13_operations.md`](13_operations.md) | Прикладные операции, функции и алгоритмы. |
| [`15_navigation_menu.md`](15_navigation_menu.md) | Целевая структура меню и связь с представлениями. |
| [`_working/decision_log.md`](_working/decision_log.md) | Рабочая фиксация решений, расхождений и открытого вопроса. |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 10:04 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #59: первые версии прикладных модулей 04 и 06 | [PR #59](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/59) |
| 0.1 | 2026-09-03 10:01 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | первые версии модулей 04, 06. правки связанной документации - шаблон и предложения по изменениям ФТ (определения терминов) | [766441e6](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/766441e68e531dadabe887393fc989e5b6e268e0) |
