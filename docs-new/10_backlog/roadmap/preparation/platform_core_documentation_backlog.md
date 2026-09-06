---
id: DOC-10-01-01
title: 'Backlog доработки документации Platform Core'
type: requirement
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: platform_core_documentation
holder: '@axelprosoft'
created_at: 2026-08-25 15:02
created_by: '@axelprosoft'
updated_at: 2026-08-27 17:56
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
---

# Backlog доработки документации Platform Core

## 1. Назначение

Документ фиксирует отложенные работы по приведению документации платформенного ядра к понятному входу для архитектурного этапа. Он не является утверждённой архитектурой и не заменяет документы `01_concept`, `02_architecture` и `03_platform`.

## 2. Область действия

### 2.1 Входит

- нормализация верхнего слоя `01_concept` и `02_architecture`;
- связь верхнего слоя с платформенными областями `03_platform`;
- единая терминология Platform Core;
- подготовка входа для `09_two_week_platform_architecture_outcome_requirements.md`;
- правила, по которым `01_scope.md` платформенной области ссылается на требования к ядру.

### 2.2 Не входит

- переписывание прикладных модулей `04_domain_modules`;
- утверждение целевой Java/PostgreSQL-архитектуры;
- удаление исходных source-копий до завершения трассировки;
- перенос backlog-задач в нормативные документы без отдельного решения.

## 3. Принцип работы

Сырые source-копии и requirements используются как материал для анализа, но не как финальная нормативная база. Целевой поток такой:

```text
01_concept + 02_architecture
→ требования и архитектурный контекст Platform Core
→ 03_platform/<area>/01_scope.md
→ 03_platform/<area>/90_traceability.md
```

Каждая задача считается закрытой только после того, как результат появился в документе-владельце и связан с трассировкой.

Допустимые статусы задач: `Open`, `In progress`, `Blocked by <ID>` и `Completed`.

## 4. Backlog

| ID | Задача | Целевой результат | Владелец документа | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-01` | Нормализовать `01_concept/01_product_concept.md`. | Понятная продуктовая концепция DMP без source-суффикса и без смешения продукта, архитектуры и backlog. | `01_concept` | Completed |
| `PCDOC-02` | Нормализовать `01_concept/02_system_scope.md`. | Системная граница DMP: что входит, что не входит, какие крупные возможности относятся к ядру и прикладным модулям. | `01_concept` | Completed |
| `PCDOC-03` | Нормализовать `02_architecture/01_architecture_overview.md`. | Общий architecture overview: слои, границы Platform Core, Domain Modules, integrations, deployment и данные. | `02_architecture` | Completed |
| `PCDOC-04` | Выделить требования к Platform Core как вход для `03_platform`. | Список требований уровня ядра, на который могут ссылаться `01_scope.md` платформенных областей. | `02_architecture` / `00_governance` | Open |
| `PCDOC-05` | Зафиксировать правило связи platform scope с верхним слоем. | В `00_platform_documentation_template.md` понятно описано, на какие документы ссылается `01_scope.md` платформенной области. | `03_platform` | Open |
| `PCDOC-06` | Согласовать терминологию Platform Core. | `11_glossary/platform_terms.md` и тематические глоссарии содержат русские и английские термины для ядра. | `11_glossary` | In progress |
| `PCDOC-07` | Связать Tenant/Security pilot с будущим верхним слоем. | После нормализации `01_concept`/`02_architecture` обновлены ссылки в Tenant/Security `01_scope.md` и `90_traceability.md`. | `03_platform/01_tenant_and_security` | Blocked by `PCDOC-01`-`PCDOC-04` |
| `PCDOC-08` | Подготовить вход в двухнедельный архитектурный этап. | `09_two_week_platform_architecture_outcome_requirements.md` ссылается на нормализованный комплект, а не на сырые source-копии. | `10_backlog/roadmap/preparation` | Open |
| `PCDOC-09` | Определить порядок разборки остальных platform areas. | Очередность областей после Tenant/Security: Configuration, Object Runtime, Frontend Platform, Value Sets, Workflow, Audit, Integration Events и другие. | `03_platform` / `10_backlog` | Open |
| `PCDOC-10` | Выделить платформенную область Frontend Platform. | Создана или согласована область `03_platform/12_frontend_platform` для общего frontend shell, границ Admin/Runtime/Studio, навигации, session/bootstrap, access visibility и shared packages. | `03_platform/12_frontend_platform` | Open |
| `PCDOC-11` | Развести владельцев frontend runtime artifacts. | Зафиксировано, где описываются `@dmp/app-admin`, `@dmp/app-runtime`, `@dmp/app-studio`, `@dmp/runtime-contracts`, `@dmp/runtime-react`, `@dmp/ui` и интеграция модулей с этими пакетами. | `03_platform/12_frontend_platform` / соседние areas | Open |
| `PCDOC-12` | Разобрать исходную концепцию Tenant/IAM. | Содержание `docs/01 sources/07_tenant_and_security_platform.md` распределено между `01_concept`, `02_architecture` и решениями Tenant/Security без source-копий внутри `03_platform/01_tenant_and_security`. | `01_concept` / `02_architecture` / `03_platform/01_tenant_and_security` | Open |
| `PCDOC-13` | Разобрать требования к ролям, правам, локализации и Admin UI. | Содержание `docs/02 requrements/!role_permition_refactor.md`, `01_roles*.md`, `01_tenants_rules.md` и `01_users_rules.md` перенесено в целевые решения, открытые вопросы или будущие доработки без папки `requirements` внутри Tenant/Security. | `03_platform/01_tenant_and_security` / `03_platform/12_frontend_platform` / `11_glossary` | Open |
| `PCDOC-14` | Оформить целевую область `03_platform/02_configuration/`. | Создан целевой комплект Configuration Platform по стандарту: граница, архитектура, контракты, runtime, безопасность, качество, эксплуатация и трассировка без source-копий внутри области. | `03_platform/02_configuration` | In progress |
| `PCDOC-15` | Перенести каталог конфигурационных артефактов в `artifact_types/`. | Содержание `docs/03 conf_artefacts` вычитано, сверено с кодом и перенесено в `03_platform/02_configuration/artifact_types/` как отдельные спецификации артефактов, без объединения в один документ и без переноса поведения соседних областей. | `03_platform/02_configuration/artifact_types` | In progress |
| `PCDOC-16` | Закрепить послойное владение конфигурационными артефактами. | Для каждого артефакта отдельно показано: что принадлежит Configuration schema, что runtime-владельцу, что Frontend Platform/приложению и что прикладному модулю; переходные frontend registries и незакрытые runtime-контракты отражены как текущие ограничения или backlog. | `00_governance`, `03_platform`, `10_backlog` | In progress |
| `PCDOC-17` | Формализовать полный контракт свойств и источников значений артефактов. | Каждый `artifact_types/<artifact>.md` следует эталонному каркасу `object_type.md` и содержит root/child properties, enum/reference values, смысл каждого значения, влияние и владельца эффекта, server/client/hybrid value sources, refresh rules, conditional visibility/readonly, validation и schema-level CRUD/operations; identity, transport/persistence fields, effective values и editor/runtime projections не смешаны со schema properties; неподтверждённые поля старых документов вынесены в traceability или backlog. | `03_platform/02_configuration/artifact_types`, `00_governance`, `10_backlog` | In progress |
| `PCDOC-18` | Закрепить единый формат тематических глоссариев и выполнить плановую миграцию исторических таблиц. | Правило единого frontmatter и базовой таблицы `Русский термин / Preferred English / Технический алиас / Статус / Определение / Источник в коде / Нежелательные синонимы` закреплено в `00_documentation_strategy.md`, Playbook и шаблоне платформенной области; тематические глоссарии ведутся отдельными файлами `11_glossary/*_terms.md`, локальные таблицы остаются только поясняющими, а исторические `platform_terms.md` и `manufacturing_terms.md` приводятся к базовой форме отдельным плановым обновлением. | `00_governance`, `03_platform`, `11_glossary` | In progress |
| `PCDOC-19` | Применить границу Foundation к владельцам платформенных областей. | Tenant/Security и Configuration ссылаются на общий слой Foundation, показывают только фактическое использование его типов и не дублируют определения; area-specific модель, persistence и контракты остаются у своих владельцев. | `03_platform/00_foundation`, `03_platform/01_tenant_and_security`, `03_platform/02_configuration` | Completed |
| `PCDOC-20` | Уточнить общую политику поля `source` во frontmatter. | Для `authored`, `upstream` и `imported` зафиксированы однозначный смысл и область применения в Strategy; Template и Playbook не противоречат правилу. Проверка текущих документов `docs-new` показала использование `authored` и `upstream`; `imported` зарезервирован для временных копий в `offers/` и `requirements/`. Массовая замена frontmatter не требуется. | `00_governance`, проверки frontmatter | Completed |

## 4.1. Детализированные рабочие пакеты платформенных областей

Этот раздел является рабочим планом выполнения задач `PCDOC-*`. Он не заменяет
содержание документов областей и не дублирует строки `90_traceability.md`.
Каждый пакет проходит одинаковые контрольные точки: инвентаризация источников,
граница области, архитектура, контракты, runtime, безопасность, frontend,
качество, эксплуатация, трассировка, терминология и итоговая проверка.

### Foundation boundary

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-19.1` | Проверить и зафиксировать использование Foundation в Tenant/Security и Configuration. | `00_platform_overview.md`, `02_architecture.md`, `03_contracts.md`, `04_runtime.md` обеих областей; `source_revision` | Общие BuildingBlocks, application-контракты, контексты, gateway-порты и `DMP.Platform.Contracts.Common` описаны владельцем Foundation; области показывают только фактическое использование, ограничения и собственную семантику. Проверены относительные ссылки и `git diff --check`; код, тесты и backlog не изменяются при повторной проверке. | Completed |

### Configuration

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-14.1` | Повторно проверить область по зафиксированной ревизии исходников. | `00_platform_overview.md`, `01_scope.md`, `90_traceability.md`, `source_revision` | Все факты связаны с точным снимком кода; ссылки на подвижную ветку отсутствуют. | Completed |
| `PCDOC-14.2` | Добавить и проверить поясняющие схемы области. | `00`–`08` | Каждая схема находится в одном документе-владельце и показывает только подтверждённые связи или последовательности. | Completed |
| `PCDOC-14.3` | Привести документы `00`–`08` к шаблону. | `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `05_security_and_audit.md`, `06_user_experience.md`, `07_quality.md`, `08_operations.md` | Верхние разделы, названия таблиц, статусы MVP и границы владельцев соответствуют шаблону. | Completed |
| `PCDOC-14.4` | Завершить вычитку существующих спецификаций артефактов. | `artifact_types/object_type.md`, `view.md`, `action.md`, `rule.md`, `value_set.md`, `system_enum.md`, `workflow.md`, `menu.md` | Для каждого свойства указаны технический код, русский смысл, тип, обязательность/default, допустимые значения, влияние, условия и владелец эффекта. | Completed |
| `PCDOC-14.5` | Завершить сверку спецификаций артефактов, добавленных в целевой пакет. | `artifact_types/numbering_rule.md`, `report.md`, `output.md` | Для каждого типа отдельно описаны schema, root/child properties, значения, validation, lifecycle и границы с runtime/frontend/reporting; содержание подтверждено schema registry, contracts, builders и тестами. | Completed |
| `PCDOC-14.5.1` | Завершить сверку артефакта `NumberingRule`. | `artifact_types/numbering_rule.md` | Сверены `NumberingRule` schema, builder/validation, связь с ObjectType и владельцем Numbering runtime; неподтверждённые поля не включены в MVP; расхождения `OnSave`, condition operators и default `InheritanceMode` (`Override` в schema/editor против fallback `Inherit` в validation projection) вынесены в трассировку. | Completed |
| `PCDOC-14.5.2` | Завершить сверку артефакта `Report`. | `artifact_types/report.md` | Сверены root/child schema, design-time contracts, content blobs, publish validation и граница исполнения Reporting Output; старые layout/runtime утверждения отделены от текущего schema-контракта. | Completed |
| `PCDOC-14.5.3` | Завершить сверку артефакта `Output`. | `artifact_types/output.md` | Сверены root/child schema, launch/delivery/mapping options, security fields, generic schema validation и граница исполнения Output runtime; расхождения schema/editor/runtime вынесены в трассировку. | Completed |
| `PCDOC-14.6` | Завершить трассировку и отделить будущие работы. | `90_traceability.md`, backlog | Каждое расхождение имеет понятное состояние, владельца, следующий шаг и целевой документ; будущие задачи не выглядят как MVP. | Completed |
| `PCDOC-14.7` | Завершить терминологическую сверку. | `11_glossary/configuration_terms.md`, весь `02_configuration` | Русские термины совпадают с кодом/UI, английские имена сохранены как technical names, дубли и локальные синонимы устранены. | Completed |
| `PCDOC-14.8` | Провести финальную проверку пакета. | Весь `02_configuration` | Проверены покрытие текущего кода и переходных источников, ссылки, Mermaid/text-схемы, отсутствие source-ссылок, русский язык и `git diff --check`; пакет готов к пользовательской вычитке. | Completed |
| `PCDOC-14.9` | Проверить полное покрытие Configuration по коду и переходным материалам. | `90_traceability.md`, весь `02_configuration` | Для каждой существенной группы исходников указан результат: перенесено, принадлежит другой области, не подтверждено кодом или требует отдельной задачи; registry, builders, controllers и релевантные тесты сопоставлены с документацией; расхождение `CFG-DEC-15` не скрыто под описанием MVP. | Completed |

| `PCDOC-14.10` | Автоматизировать сверку спецификаций артефактов с каноническими источниками. | `artifact_types/`, schema registrations, `ConfigurationStaticValueCodes`, `ArtifactStaticOptionCatalog`, `ArtifactValueSourceDefinition`, `ConfigurationSchemaMetadataCatalog`, runtime projections | Проверка выявляет отсутствующие или лишние свойства и коллекции, расхождения кодов и UI-метаданных, а также различия между полным schema-контрактом и поддержанным runtime-подмножеством; результат направляется в документацию и `90_traceability.md`. | Completed |

| `PCDOC-14.11` | Описать человекочитаемый контракт формирования baseline package. | `03_contracts.md`, `04_runtime.md`, `08_operations.md`, `artifact_types/` | В документации различены package envelope, `SystemBaseline` scope, schema properties, identity, localizations, импорт, validation и publication; есть понятная схема формирования без зависимости от C# builder API. | Completed |

| `PCDOC-14.12` | Уточнить человекочитаемую модель идентичности Configuration. | `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `artifact_types/` | Однозначно разведены `Code`, `ArtifactCode`, `ArtifactTypeCode`, `OriginKey`, `ParentOriginKey`, `EntryId`, `OverridesEntryId` и `ModuleCode`; приведён один пример сопоставления package entry, canonical entry и persistence-записи. | Completed |

| `PCDOC-14.13` | Добавить поясняющие схемы effective configuration и разрешения локализации. | `04_runtime.md`, `05_security_and_audit.md` при необходимости, `artifact_types/` только со ссылками | Показаны scope chain `SystemBaseline → Corporate → Tenant → Site`, наследование, override, explicit null, rebase и порядок выбора локализованного значения; одинаковые правила не дублируются в каждом артефакте. | Completed |

| `PCDOC-14.14` | Свести проверки перед публикацией в одну понятную матрицу. | `04_runtime.md`, `07_quality.md`, `90_traceability.md` | Отдельно показаны schema, значения, identity, parent links, циклы, references, localizations, runtime compatibility, draft references и security context; неподтверждённые проверки не выдаются за гарантию MVP. | Completed |

| `PCDOC-14.15` | Зафиксировать будущий export и перенос Configuration между окружениями. | `03_contracts.md`, `08_operations.md`, `90_traceability.md` | Старые требования export, conflict preview и transfer между окружениями отмечены как будущая capability, если текущий код не подтверждает публичный контракт; определены владелец и следующий этап. | Completed |

| `PCDOC-14.16` | Развести переходные UI и Reporting/Output материалы по владельцам. | `01_scope.md`, `06_user_experience.md`, `artifact_types/report.md`, `artifact_types/output.md`, будущие области | Для `Navigation`, `Lookup`, `GridUi`, `EnumPresentation`, editor grouping и полного BIRT flow указано, что уже покрыто Configuration, что принадлежит Frontend или Reporting/Output, а что остаётся только источником будущих требований. | Completed |
| `PCDOC-14.17` | Разобрать общий формат условий и удалить зависимость от переходного `06_condition_model.md`. | `90_traceability.md`, `artifact_types/numbering_rule.md`, документы Platform Numbering | Реализованный `NumberingRule.ConditionJson` отделён от `ConditionExpression`, schema conditions, runtime filters и `RuleLogic`; все активные ссылки на переходный документ заменены; возможная общая унификация оставлена отдельной будущей задачей. | Completed |
| `PCDOC-14.18` | Закрыть переходную папку Configuration Platform после инвентаризации. | `02_configuration/90_traceability.md`, `00_platform_overview.md`, `01_scope.md` | Для всех групп переходных материалов зафиксирован маршрут: перенесено в целевой пакет, принадлежит другой области, сохранено как будущая задача или является дублирующей копией. Активные ссылки проверены, `git diff --check` выполнен, переходная папка удалена; старый `docs/` сохранён. | Completed |

### Tenant/Security (префикс задач: `PCDOC-TS`)

В этом разделе `TS` означает `Tenant/Security`, а не Time Series. Backlog
будущей Time Series использует отдельное обозначение `TSS`.

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-TS-01` | Повторно проверить доказательную ревизию после обновления master. | Пакет `03_platform/01_tenant_and_security` | Обновление `source_revision` выполняется только после проверки изменения поведения; найденные изменения маршрутизируются через `90_traceability.md`. | Open |
| `PCDOC-TS-02` | Завершить связь с верхними `01_concept` и `02_architecture`. | `01_scope.md`, `90_traceability.md` и верхний слой | Граница Tenant/Security с верхним слоем и Configuration согласована без дублирования. | Blocked by `PCDOC-04` |

### Object Runtime

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-OR-01` | Проверить пакет по единому стандарту платформенной области. | `03_platform/03_object_runtime/00`–`90` | Общие документы имеют единые каркасы, а runtime-модель Object Runtime не смешана с Configuration schema. | Completed |
| `PCDOC-OR-02` | Развести открытые решения и будущие runtime-доработки. | `90_traceability.md`, backlog | Решения ORT-* не переносятся в текущие гарантии без отдельного согласования; будущие работы имеют маршрут и владельца. | Completed |
| `PCDOC-OR-03` | Закрыть переходную папку `10_object_runtime`. | Реестр миграции, активные ссылки в Domain Modules и Platform Numbering, планы и документационная автоматизация | Активные ссылки переведены на `03_object_runtime`; материалы для Configuration, Frontend Platform и будущих областей имеют владельца и маршрут; переходная папка удалена. Исходные материалы сохранены в `docs/`, актуальные маршруты зафиксированы в `99_archive_migration_review.md`, проверка ссылок и `git diff --check` выполнены. | Completed |
| `PCDOC-OR-04` | Добавить сквозное объяснение стандартной операции Object Runtime. | `03_object_runtime/04_runtime.md` | Один понятный сценарий связывает запрос потребителя, `ApplicationRuntimeService`, `IObjectRuntime`, descriptor, хранилище или mutation pipeline и runtime-ответ без каталога классов. | Completed |
| `PCDOC-OR-05` | Описать фактическое выполнение ссылок и вложенных коллекций. | `03_object_runtime/04_runtime.md` | Различены lookup, отображаемые значения, `EmbeddedCollectionContext`, `WithOwner`, `Separate`, ownership и текущие ограничения; свойства Configuration и frontend-поведение не дублируются. | Completed |
| `PCDOC-OR-06` | Описать фактическое выполнение расширяемых значений. | `03_object_runtime/02_architecture.md`, `04_runtime.md`, `03_contracts.md` при необходимости | Зафиксированы условие подключения `IObjectExtensionValueStore`, чтение, фильтрация/сортировка, запись через mutation writer и владение хранением прикладным модулем. | Completed |
| `PCDOC-OR-07` | Завершить сверку старых Object Runtime источников на понятность и покрытие. | `docs/other_tech/dmp-module-object-runtime.md`, `docs/current/04_object_runtime_reference_module.md`, `docs/current/05_object_runtime_reference_relations_design.md`, `90_traceability.md` | Подтверждённые текущим кодом сведения перенесены; будущие design ideas, MES-сравнение, UI и предметные примеры имеют явный маршрут и не выдаются за MVP. | Completed |
| `PCDOC-OR-08` | Добавить минимальные JSON-примеры контрактов Object Runtime. | `03_object_runtime/03_contracts.md` | В разделе списка показаны поясняющие запрос и ответ; примеры не заменяют таблицы контрактов и не задают универсальные коды. | Completed |
| `PCDOC-OR-09` | Провести итоговую вычитку Object Runtime после содержательных дополнений. | Весь `03_object_runtime` | Повторно проверены структура, русский язык, ссылки, единственный владелец факта, source revision, отсутствие дублирования и `git diff --check`. | Completed |

### Workflow

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-WF-01` | Провести инвентаризацию источников Workflow и зафиксировать снимок кода. | `docs/03 conf_artefacts/Workflow.md`, workflow requirements и ADR, `docs-new/03_platform/04_workflow`, `src/Platform/DMP.Platform.Workflow`, workflow contracts, composition root и релевантные тесты | Для каждого существенного source-блока указано, что подтверждено кодом и тестами, что ограничено MVP, что является будущей возможностью и куда направляется. В документах зафиксирован точный `source_revision`; старый путь `03_workflow_engine` заменён каноническим `04_workflow`, исходный UI requirements-файл сохранён в `docs/`. | Completed |
| `PCDOC-WF-02` | Уточнить границу Workflow как платформенной области. | Матрица владельцев и `01_scope.md` | Разделены Configuration и артефакт `Workflow`, Workflow Runtime, Rules, Object Runtime, Audit History, Integration Events и Frontend Platform. Бизнес-смысл объекта и предметные workflow остаются у Domain Modules. | Completed |
| `PCDOC-WF-03` | Подготовить нормативный пакет `04_workflow`. | `03_platform/04_workflow/00_platform_overview.md`, `01_scope.md`, `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `05_security_and_audit.md`, `07_quality.md`, `08_operations.md`, `90_traceability.md`; `06_user_experience.md` только при наличии собственного владельческого UX | Пакет следует единому шаблону платформенной области; в него входят только подтверждённые Workflow Runtime-механизмы и публичные контракты; документы не дублируют Configuration schema, Rule Engine, Object Runtime mutation, Frontend Platform и Audit/Event Foundation. | Completed |
| `PCDOC-WF-04` | Разобрать Workflow UI и editor-материалы по владельцам. | Workflow Editor и WF Runtime UI requirements, ссылки из `Configuration`, `Frontend Platform` и domain modules | Editor UX направлен владельцу Configuration/Frontend Platform, runtime renderer и frontend contracts — Frontend Platform, а в Workflow остаются только необходимые backend-контракты и ограничения. Неподтверждённые варианты не выдаются за MVP. | Completed |
| `PCDOC-WF-05` | Завершить трассировку, терминологию и финальную проверку Workflow. | `04_workflow/90_traceability.md`, тематический glossary при необходимости, ссылки и проверки | Открытые вопросы имеют понятные темы, владельцев и следующий шаг; `CommandCode`, `Transition`, `WorkflowDefinitionRevision`, `StatePolicy`, `WorkflowHistory` и связанные термины согласованы с кодом; проверены ссылки, Mermaid/text-схемы и `git diff --check`. | Completed |
| `PCDOC-WF-06` | Зафиксировать минимальные навигационные ссылки между Workflow и готовыми областями. | Обзоры и документы границ `00_foundation`, `01_tenant_and_security`, `02_configuration`, `03_object_runtime`, `04_workflow` | В местах конкретного взаимодействия добавлены навигационные ссылки: Foundation ссылается на Workflow как на потребителя общих портов; Workflow — на Tenant/Security, Configuration и Object Runtime; Configuration и Object Runtime — на Workflow. Содержание не дублируется. | Completed |
| `PCDOC-WF-07` | Пересмотреть статус `CFG-DEC-06` после фиксации владельца Workflow Runtime. | `03_platform/02_configuration/90_traceability.md`, пакет `04_workflow` | В Configuration закреплено: schema/definition артефактов `Workflow` и `Rule` принадлежат Configuration; исполнение Workflow — `04_workflow`; исполнение Rules — владельцу Rules. Открытыми остаются только отдельные будущие темы, а не граница владельцев. | Completed |
| `PCDOC-WF-08` | Проверить устаревшую ревизию источника в Tenant/Security. | `03_platform/01_tenant_and_security` и его `source_revision` | Ссылки на код в документах Tenant/Security переведены со старого commit `2854252e...` на текущий `origin/master@3e2037ac...`; frontmatter и пути проверены. | Completed |
| `PCDOC-WF-09` | Добавить понятную схему инициализации Workflow после создания объекта. | `04_workflow/04_runtime.md` | Показан подтверждённый поток `Object Runtime` → integration event → host handler → runtime projection → `InitializeInstance` → state store; доставка события не выдана за гарантию Workflow. | Completed |
| `PCDOC-WF-10` | Уточнить схему жизненного цикла definition и revision. | `04_workflow/04_runtime.md` | На одной схеме видны effective definition из Configuration, snapshot revision и привязка экземпляра; будущая migration policy не смешана с текущим MVP. | Completed |
| `PCDOC-WF-11` | Определить место терминов Workflow. | `docs-new/11_glossary/platform_terms.md`, локальные таблицы Workflow | Термины Workflow признаны платформенными; отдельный `workflow_terms.md` не создаётся, а локальные таблицы используются только для удобства чтения. | Completed |
| `PCDOC-WF-13` | Добавить минимальный JSON-пример контракта Workflow. | `04_workflow/03_contracts.md` | В разделе выполнения команды показаны поясняющие запрос и ответ; примеры не задают обязательную definition для всех модулей. | Completed |
| `PCDOC-WF-12` | Провести итоговую вычитку Workflow после содержательных дополнений. | Весь `04_workflow` | Повторно проверены структура, русская формулировка, диаграммы, владельцы, source revision, ссылки и отсутствие дублирования с Configuration/Object Runtime. | Completed |

### Rules

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-RULE-01` | Подготовить целевой пакет `05_rules` по фактическому MVP. | `03_platform/05_rules/00`–`90`, код `DMP.Platform.Rules`, адаптер host-приложения и тесты Workflow | Описаны только `IRuleEvaluationGateway`, его типы запроса и ответа, fallback и ограниченная реализация host-приложения; схема `Rule` и привязки остаются у Configuration. | Completed |
| `PCDOC-RULE-02` | Зафиксировать границы Rules с Configuration, Object Runtime и Workflow. | `05_rules/00_platform_overview.md`, `01_scope.md`, `02_architecture.md`, `90_traceability.md` | Указано, что Rules получает эффективное описание `Rule`, читает детали объекта, возвращает результат Workflow и не владеет схемой, объектом или переходом. | Completed |
| `PCDOC-RULE-03` | Уточнить будущую модель вычисления правил. | `05_rules/90_traceability.md`, будущий контракт Rules | Отдельно приняты решения по параметрам, `DecisionTable`, `Script`, `Dsl`, вычислениям и эффектам; текущий MVP не расширен без реализации. | Open |
| `PCDOC-RULE-04` | Определить политику промышленного использования fallback и наблюдаемости Rules. | `05_rules/07_quality.md`, `08_operations.md`, Foundation / Platform Operations | Определены допустимость `NotConfiguredRuleEvaluationGateway`, обязательные метрики, трассировки и проверки работоспособности. | Open |
| `PCDOC-RULE-05` | Провести итоговую вычитку пакета Rules. | Весь `05_rules` | Проверены источники `origin/master`, термины, локальные ссылки, схемы, отсутствие дублирования и `git diff --check`; пакет подготовлен как черновик перед ревью. | Completed |

### Settings

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-SET-01` | Провести инвентаризацию источников и фактического кода Settings. | `DMP.Platform.Settings`, `DMP.Platform.Contracts/Settings`, Configuration catalog resolver, host composition, frontend feature и integration tests | Подтверждены catalog manifest, runtime values, user preferences, API, scope access, cache, audit hand-off и собственный Settings feature provider; старые материалы отделены от фактов кода. | Completed |
| `PCDOC-SET-02` | Подготовить целевой пакет `07_settings` по стандарту платформенной области. | `03_platform/07_settings/00`-`90` | Созданы документы обзора, границы, архитектуры, контрактов, runtime, безопасности и аудита, UX, качества, операций и трассировки; Configuration schema и смысл настроек модулей не продублированы. | Completed |
| `PCDOC-SET-03` | Провести формальную вычитку пакета Settings перед ревью. | Весь `07_settings` | Проверены frontmatter, единый каркас, русская формулировка, термины, локальные ссылки, таблицы, Mermaid-схемы, source revision и `git diff --check`; `review_status` остаётся `not_started` до отдельной отправки на ревью. | Completed |
| `PCDOC-SET-04` | Определить применение `RequiresRestart` и `RequiresReload`. | `07_settings/90_traceability.md`, будущий runtime/operations contract | Назначен механизм применения и сигнал для настроек с соответствующей policy; текущий MVP не расширяется без реализации. | Open |
| `PCDOC-SET-05` | Определить будущие расширения Settings runtime. | `07_settings/90_traceability.md`, будущие контракты Settings | Отдельно решены custom validators, optimistic concurrency, distributed cache invalidation и permission policy user preferences; текущие ограничения MVP сохранены. | Open |

### Audit History

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-AUD-01` | Провести инвентаризацию источников и фактического кода Audit History. | `docs/01 sources/12_audit_history_platform.md`, переходная папка `08_audit_history`, `DMP.Platform.AuditHistory`, host composition и интеграционные тесты | Разделены подтверждённые возможности общего writer, ограничения текущего MVP и неподтверждённые ожидания старого материала; зафиксирован `source_revision`. | Completed |
| `PCDOC-AUD-02` | Подготовить канонический пакет `09_audit_history` по стандарту платформенной области. | `03_platform/09_audit_history/00`-`90` без отдельного UX-документа | Описаны только `AuditRecord`, `IAuditHistoryWriter`, провайдеры хранения, runtime-путь, границы безопасности, качество, операции и трассировка; пакет не дублирует историю Workflow, business history, Integration Events или observability. | Completed |
| `PCDOC-AUD-03` | Развести Audit, business history, Workflow History, Integration Events и technical trace. | `09_audit_history/01_scope.md`, `02_architecture.md`, `05_security_and_audit.md`, `90_traceability.md` | Для каждой модели указан владелец; старые утверждения о трёх самостоятельных хранилищах не выданы за текущую реализацию. | Completed |
| `PCDOC-AUD-04` | Провести формальную вычитку пакета Audit History перед ревью. | Весь `09_audit_history` | Проверены frontmatter, единый каркас, русская формулировка, технические имена, ссылки, таблицы, схемы, source revision и `git diff --check`; `review_status` остаётся `not_started`. | Completed |
| `PCDOC-AUD-05` | Определить будущий query API, retention, atomicity/outbox, action-code registry и actor model. | `09_audit_history/90_traceability.md`, будущие контракты и эксплуатационная политика | Для каждой темы принято отдельное решение или создана будущая задача; текущий writer MVP не расширяется задним числом. | Open |
| `PCDOC-AUD-06` | Сопоставить исторические требования к audit envelope и обязательным категориям аудита с producer-операциями. | `09_audit_history/90_traceability.md`, coverage matrix Tenant/Security, Object Runtime, Workflow, Settings, Configuration, Reporting и Integration Events | Для `ObjectTypeCode`/`ObjectId`/`Result`, operation categories и обязательности записи принято решение или зафиксировано, что тема остаётся будущей; текущий `AuditRecord` не расширяется без согласованного контракта. | Open |
| `PCDOC-AUD-07` | Принять production-политику durable audit storage, append-only защиты и сквозной корреляции. | `09_audit_history/90_traceability.md`, draft ADR-010, deployment/operations policy | Определены запрет in-memory в production, database permissions/tamper protection, correlation propagation и требуемая atomicity с outbox; до решения текущие ограничения сохраняются. | Open |
| `PCDOC-AUD-08` | Развести термины Audit History и Configuration в тематическом глоссарии. | `11_glossary/platform_terms.md`, `11_glossary/configuration_terms.md` | Добавлены `AuditRecord`, `IAuditHistoryWriter` и `AuditRecord.ActionCode` как платформенные термины; `Action` Configuration переименован в `Код конфигурационного действия`; `CorrelationId` используется без дублирования; отдельный `audit_history_terms.md` не создаётся. | Completed |
| `PCDOC-AUD-09` | Закрыть переходную offer-копию Audit History после миграции. | `09_audit_history/90_traceability.md`, `99_archive_migration_review.md`, активные ссылки на `08_audit_history/offers` | Проверено отсутствие уникального содержания и активных ссылок; переходная копия удалена, исходник в `docs/01 sources` сохранён, маршрут содержания указывает на канонический пакет. | Completed |

### Value Sets

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-VS-01` | Провести инвентаризацию источников Value Sets и зафиксировать снимок кода. | `03_platform/05_value_set_data`, `DMP.Platform.ValueSets`, contracts, host composition и релевантные тесты | Разделены переходные исходники и целевая область; подтверждены владельцы `ValueSetDataSet`, `ValueSetItem`, API, scope и runtime adapter. | Completed |
| `PCDOC-VS-02` | Подготовить целевой пакет `06_value_sets` по стандарту платформенной области. | `03_platform/06_value_sets/00`–`90` | Создан единый пакет без source-копий и ссылок на старые `docs/...`; каждый документ содержит только собственное содержание области. | In progress |
| `PCDOC-VS-03` | Описать модель effective values и scoped overrides. | `06_value_sets/02_architecture.md`, `04_runtime.md`, `05_security_and_audit.md` | Однозначно описаны Global/Tenant/Site, приоритет scope, flat/hierarchical items, inherited rows и ограничения mutation. | In progress |
| `PCDOC-VS-04` | Описать публичные контракты Value Sets. | `06_value_sets/03_contracts.md` | Зафиксированы read API, data editor API, DTO, permissions, ошибки и граница с Configuration/Runtime без каталога приватных методов. | In progress |
| `PCDOC-VS-05` | Сверить исходные требования Value Sets с кодом и перенести только подтверждённое. | `06_value_sets/06_user_experience.md`, `07_quality.md`, `90_traceability.md` | Требования editor, hierarchy, display и UI отделены от подтверждённого backend; будущие решения имеют понятный маршрут. | Completed |
| `PCDOC-VS-06` | Провести итоговую вычитку пакета Value Sets. | Весь `06_value_sets` | Проверены единообразие разделов, терминология, схемы, ссылки, source revision, отсутствие дублирования и `git diff --check`; пакет готов к ревью. | In progress |
| `PCDOC-VS-07` | Закрыть переходную папку исходных материалов Value Sets. | `03_platform/05_value_set_data`, `06_value_sets/90_traceability.md` | Для каждого переходного файла зафиксирован маршрут содержания, будущий владелец или причина исключения; переходная папка удалена; исходники в `docs/` сохранены. | Completed |
| `PCDOC-VS-08` | Определить будущие расширения модели данных Value Sets. | `06_value_sets/90_traceability.md`, будущие контракты Value Sets | Приняты отдельные решения по `Fields[]`, локализации элементов, reorder/move и tombstone; текущий MVP не смешан с будущей моделью. | Open |
| `PCDOC-VS-09` | Подготовить отдельное описание frontend-редактора Value Sets. | `06_value_sets/06_user_experience.md` | Описаны подтверждённые feature provider, маршрут, навигация, View/renderer и граница между backend DTO Value Sets и общей Frontend Platform. | Completed |

### Numbering

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-NUM-01` | Подготовить целевой пакет `08_numbering` по стандарту платформенной области. | `03_platform/08_numbering/00`–`90` | Перенесены подтверждённые границы, runtime, persistence, контракты, безопасность, операции и трассировка; `NumberingRule` остаётся владельцем Configuration. | Completed |
| `PCDOC-NUM-02` | Сверить Numbering с текущим кодом и тестами. | `DMP.Platform.Numbering`, host composition, Configuration schema/validation и integration tests | В документации разделены подтверждённые сценарии, host resolver, ограничения V1 и неподтверждённые возможности; source revision зафиксирована. | Completed |
| `PCDOC-NUM-03` | Провести итоговую вычитку пакета Numbering. | Весь `08_numbering` | Проверены структура, терминология, ссылки, отсутствие дублирования и ручных строк истории; пакет подготовлен как черновик перед ревью. | Completed |
| `PCDOC-NUM-04` | Проверить конкурентность Numbering на каждом поддерживаемом production provider-е. | Numbering persistence и отдельные provider-specific tests | Для одного ключа счётчика подтверждены уникальность последовательности и отсутствие дублирующих строк при параллельной выдаче. | Open |
| `PCDOC-NUM-05` | Принять решения по будущим моментам и административным операциям Numbering. | `08_numbering/90_traceability.md`, контракты Numbering, Object Runtime и Configuration | Отдельно решены `OnSave`, ручной запрос, workflow-переходы, UI/корректировка счётчиков и общий audit/event contract; будущие возможности не выдаются за MVP. | Open |
| `PCDOC-NUM-06` | Закрыть переходную папку `11_numbering` после миграции. | `08_numbering/90_traceability.md`, `00_governance/99_archive_migration_review.md` | Для каждого исходного файла зафиксирован маршрут содержания, активные ссылки переведены на `08_numbering`, переходная копия удалена, исходные материалы в `docs/` не затронуты. | Completed |

### Reporting and Output

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-RO-01` | Провести инвентаризацию Reporting/Output и зафиксировать снимок исходников. | `DMP.Platform.Configuration`, `DMP.Platform.Runtime`, `DMP.ReportService`, frontend contracts и тесты | Отдельная сборка не заявлена; зафиксированы владельцы design-time, runtime, renderer и frontend. | Completed |
| `PCDOC-RO-02` | Подготовить границы сквозной capability Reporting/Output. | `11_reporting_output/00_platform_overview.md`, `01_scope.md`, `90_traceability.md` | Разделены Configuration schema, runtime execution, Java Report Service, frontend и соседние владельцы; дублирование не добавлено. | Completed |
| `PCDOC-RO-03` | Описать архитектуру и межкомпонентные контракты Reporting/Output. | `02_architecture.md`, `03_contracts.md` | Описаны только подтверждённые HTTP, application ports, DTO и renderer contract; внутренние методы не выданы за публичные контракты. | Completed |
| `PCDOC-RO-04` | Описать runtime-сценарии формирования отчётного результата. | `04_runtime.md`, `08_operations.md` | Зафиксирован путь от опубликованных `Report`/`Output` до PDF/Excel или ошибки, включая preview и эксплуатационные ограничения. | Completed |
| `PCDOC-RO-05` | Описать безопасность, качество и frontend-границу. | `05_security_and_audit.md`, `06_user_experience.md`, `07_quality.md` | Разделены реальные проверки прав, тестовое свидетельство, Studio и runtime UI от общей Frontend Platform. | Completed |
| `PCDOC-RO-06` | Провести трассировку старых Reporting/Output материалов. | `90_traceability.md` | Для каждого существенного источника указан маршрут: перенесено, принадлежит Configuration, принадлежит другой области, будущая доработка или не подтверждено. | Completed |
| `PCDOC-RO-07` | Провести формальную проверку пакета Reporting/Output. | Весь `11_reporting_output` | Проверены frontmatter, стандартные разделы, терминология, ссылки, отсутствие ручной истории и `git diff --check`. | Completed |
| `PCDOC-RO-08` | Выполнить пользовательскую вычитку пакета Reporting/Output перед отправкой на ревью. | Весь `11_reporting_output` | Пользователь проверил границы области, MVP, будущие решения и формулировки; замечания обработаны. | Open |

### Integration Events

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-IE-01` | Провести инвентаризацию Integration Events и зафиксировать снимок кода. | `DMP.Platform.IntegrationEvents`, `DMP.Platform.Contracts/IntegrationEvents`, host composition, producer-ы и релевантные integration tests | Разделены текущие outbox/processor-механизмы, контракты producer-ов, найденные handlers и неподтверждённые свойства; в документах указан точный `source_revision`. | Completed |
| `PCDOC-IE-02` | Подготовить целевой пакет `10_integration_events` по стандарту платформенной области. | `03_platform/10_integration_events/00`–`90` | Описаны границы, envelope, persistence, порты, runtime, безопасность, качество, операции и трассировка; не добавлены внешний broker, inbox или event registry без подтверждения кодом. | In progress |
| `PCDOC-IE-03` | Провести формальную и содержательную вычитку пакета Integration Events. | Весь `10_integration_events` и ссылка из `02_architecture/07_integration_architecture.md` | Проверены стандартные разделы, frontmatter, терминология, маршруты владельцев, таблицы диагностики, локальные ссылки и отсутствие ручной истории. | Open |
| `PCDOC-IE-04` | Провести пользовательскую вычитку перед передачей пакета на ревью. | `10_integration_events` | Пользователь подтвердил понятность границ, текущих гарантий и открытых вопросов; после этого пакет можно отправлять на ревью. | Open |
| `PCDOC-IE-05` | Закрыть переходный материал `07_event_foundation` после миграции. | `10_integration_events/90_traceability.md`, `00_governance/99_archive_migration_review.md` | Для каждого переходного файла зафиксирован маршрут содержания, активные ссылки переведены, переходная копия удалена только после проверки отсутствия ссылок. | Completed |
| `PCDOC-IE-06` | Определить будущую надёжность доставки событий. | `10_integration_events/90_traceability.md`, архитектурные решения и тесты | Отдельно решены atomicity с producer transaction, inbox/dedup, claim/lock, внешний transport, replay, retention и наблюдаемость; решения не выдаются за текущий MVP. | Open |
| `PCDOC-IE-07` | Определить политику event contract versioning. | `10_integration_events/03_contracts.md`, `90_traceability.md` и документы владельцев payload | Зафиксированы registry `EventTypeCode`, версия payload, совместимость и правила изменения producer/consumer контрактов. | Open |
| `PCDOC-IE-08` | Зафиксировать терминологию Integration Events в общем глоссарии. | `11_glossary/platform_terms.md` | Русские и английские термины для integration event, event envelope, outbox, dead letter и `CorrelationId` согласованы с локальным пакетом и кодом. | Completed |
| `PCDOC-IE-09` | Сверить требования старых ADR и platform integration rules с текущим outbox. | `10_integration_events/90_traceability.md`, ADR-0014, ADR-010 и module integration rules | Разделены принятые целевые решения, фактический код и неподтверждённые гарантии `after commit`, idempotency, tenant/correlation metadata и production-in-memory запрета. | In progress |
| `PCDOC-IE-10` | Определить единый контракт идентичности, версий и fan-out событий. | `10_integration_events/03_contracts.md`, `90_traceability.md` и документы владельцев payload | Решены `EventId`, `PayloadVersion`, stable/external identity, registry `EventTypeCode`, multi-consumer delivery и совместимость payload. | Open |
| `PCDOC-IE-11` | Определить эксплуатационную политику хранения outbox. | `10_integration_events/07_quality.md`, `08_operations.md`, `90_traceability.md` | Установлены retention, cleanup, архивирование, диагностика processed/dead-letter записей и правила ручного восстановления. | Open |

### Integration Capability (будущая область)

Эта работа не входит в `10_integration_events`. Она использует его как механизм
доставки событий, но добавляет обмен с внешними системами и собственное runtime-
состояние. Таблица ниже является кратким индексом. Подробный backlog будущей
capability, включая дополнительные рабочие пакеты и открытые решения, ведётся в
[подробном backlog Integration Capability](../../platform_capabilities/integration_capability_backlog.md).
Одинаковые ID в двух документах обозначают одни и те же задачи, а не разные
работы.

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-IC-01` | Сформировать будущую границу Integration Capability. | [Integration Capability backlog](../../platform_capabilities/integration_capability_backlog.md) и отдельный scope-документ | Разделены Integration Events, Integration Service, connectors, внешние системы, flows, mappings и domain/application contracts; будущая capability не объявляется реализованной. | Open |
| `PCDOC-IC-02` | Определить конфигурационную модель интеграций. | Будущие спецификации `ExternalSystem`, `ConnectorDefinition`, `IntegrationFlow`, `IntegrationContractBinding`, `MappingDefinition`, `RetryPolicy`, `ErrorPolicy`, `TriggerDefinition`, `CredentialRef` | Для каждого типа определены владелец, lifecycle, scope, versioning, publish/effective rules, секреты и граница с Configuration. | Open |
| `PCDOC-IC-03` | Определить operational runtime-модель интеграций. | Будущие контракты `IntegrationMessage`, `IntegrationAttempt`, `ExternalObjectMapping`, `SyncState`, `Inbox` и manual reprocess | Разделены входящий и исходящий поток, сообщение, попытка, состояние синхронизации, идемпотентность, дубликаты и внешний идентификатор. | Open |
| `PCDOC-IC-04` | Подготовить первые сквозные сценарии Integration Capability. | Один inbound flow и один outbound flow с выбранным внешним контуром и connector | Описаны trigger, mapping, вызов domain/application contract, ошибки, retry, диагностика, безопасность и критерий повторной обработки. | Open |
| `PCDOC-IC-05` | Создать отдельную каноническую платформенную область Integration Capability. | Новый свободный номер и пакет `00`–`90` после утверждения границы и наличия реализации | Пакет создаётся только после выбора владельца, номера, кода, контрактов, persistence и тестов; переходный `06_integration/offers` после переноса в backlog не является источником истины. | Open |

## 4.2. Решения перед отдельными работами

| ID | Тема | Принятое решение | Основание | Связанная задача | Статус |
| --- | --- | --- | --- | --- | --- |
| `PCDOC-DEC-01` | Минимальные JSON-примеры в `03_contracts.md` | Принято: добавлять по одному минимальному поясняющему примеру запроса и ответа для Object Runtime и Workflow. | Примеры помогают читать контракт; таблицы остаются нормативным описанием полей. | `PCDOC-OR-08`, `PCDOC-WF-13` | Принято |
| `PCDOC-DEC-02` | Отдельный тематический глоссарий Workflow | Принято: считать термины Workflow платформенными и вести их в `platform_terms.md` с локальными пояснениями; отдельный файл не создавать. | Термины описывают механизм платформы, а не предметную модель прикладного модуля. | `PCDOC-WF-11` | Принято |

## 4.3. Нормативные правила понятного описания

| ID | Задача | Целевой результат | Владелец документа | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-NORM-01` | Проверить и при необходимости уточнить единое правило поясняющих схем для платформенных областей. | Playbook и Template однозначно закрепляют: схема нужна, когда она добавляет понимание границы, связи или порядка; схема не обязательна в каждом файле; самостоятельные pipeline и межобластные события получают сквозную схему; `07_quality.md` и `90_traceability.md` используют таблицы, если схема не добавляет новой информации. | `00_governance/02_platform_area_documentation_playbook.md`, `03_platform/00_platform_documentation_template.md`, при противоречии `00_documentation_strategy.md` | Completed |
| `PCDOC-NORM-02` | Добавить в контрольную проверку платформенной области понятный читательский путь. | Для каждой области проверяется, что архитектор может проследить хотя бы один основной сценарий от входа до результата, понять владельцев и увидеть ограничения MVP; это не требует каталога классов или JSON-примера для каждого контракта. | `00_governance/02_platform_area_documentation_playbook.md`, `03_platform/00_platform_documentation_template.md` | Completed |

## 4.4. Сквозная архитектура взаимодействия Platform Core

Этот пакет фиксирует отдельную работу верхнего архитектурного слоя. Общая карта
взаимодействий не принадлежит Foundation: Foundation описывает общие технические
примитивы и контракты, а сквозная архитектура модулей должна находиться в
`02_architecture/07_integration_architecture.md`. Подробные контракты и runtime-
сценарии остаются у владельцев соответствующих платформенных областей.

| ID | Задача | Целевой результат | Владелец документа | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-ARCH-01` | Подготовить канонический документ сквозной архитектуры взаимодействия ядра. | Создан `docs-new/02_architecture/07_integration_architecture.md` с картой модулей ядра, легендой каналов, матрицей взаимодействий, правилами владения и ссылками на документы областей. В документе различены HTTP API, внутренний C# application port/gateway и integration event через `Integration Events`; подробности не дублируются. | `02_architecture` | Completed |
| `PCDOC-ARCH-02` | Провести инвентаризацию межмодульных связей по коду и текущей документации. | Для каждой существенной связи указаны инициатор, получатель или внешний потребитель, точный контракт, триггер, синхронность, результат, ошибки/повторы, владелец семантики и документ-подробность. Проверяются публикации и обработчики integration events, composition root, C#/HTTP/event contracts, host adapters, registrations и релевантные тесты; ожидания старых материалов отмечены отдельно. Первичный проход по текущему host выполнен; требуется формальная сверка с владельцами будущих областей и проверка полного каталога HTTP/event schemas. | `02_architecture` / владельцы платформенных областей | In progress |
| `PCDOC-ARCH-03` | Описать сквозной сценарий `TenantCreated -> Configuration Tenant scope`. | В общей архитектуре показан поток после создания Tenant: сохранение в Tenant Security, публикация `TenantCreated`, обработка в Configuration и создание или поиск `ConfigurationScopeType.Tenant` под `Corporate`. Отдельно указано, что configuration version автоматически не публикуется, а доставка и повторная обработка принадлежат Integration Events. | `02_architecture`, `03_platform/01_tenant_and_security`, `03_platform/02_configuration` | Completed |
| `PCDOC-ARCH-04` | Свести в общую архитектуру сценарий `Object create -> Workflow initialization`. | В общей карте и sequence diagram показан подтверждённый поток `Object Runtime` -> `Workflow.InstanceInitializationRequested` -> host handler -> `RuntimeWorkflowProjectionService` -> `WorkflowRuntimeService` -> initial state. Детали payload и runtime уже остаются в `03_object_runtime` и `04_workflow`; будущие гарантии доставки не добавляются. | `02_architecture`, `03_platform/03_object_runtime`, `03_platform/04_workflow` | Completed |
| `PCDOC-ARCH-05` | Сверить остальные зависимости платформенных областей с реальными каналами взаимодействия. | Связи Object Runtime/Configuration/Tenant Security/Workflow/Rules/Value Sets/Settings/Numbering/Reporting/Audit History/Integration Events и Domain Modules классифицированы как HTTP API, application port, host adapter, integration event или extension point. Объявленные, но не подключённые порты и `NoOp`/fallback-реализации отмечены отдельно; направления, которые пока подтверждены только зависимостью, не выдаются за реализованный сценарий. Классификация текущего host выполнена; остаётся сверка будущих областей и общих нормативных правил. | `02_architecture` / владельцы платформенных областей | In progress |
| `PCDOC-ARCH-06` | Добавить минимальные навигационные ссылки на общую архитектуру взаимодействия. | В обзоры Foundation, Tenant/Security, Configuration, Object Runtime и Workflow добавлена по одной навигационной ссылке на `02_architecture/07_integration_architecture.md`; локальные документы сохраняют только собственные контракты, алгоритмы и ограничения. Для будущих областей правило применяется при создании их пакетов. | `03_platform/00_foundation`, подготовленные platform areas | Completed |
| `PCDOC-ARCH-07` | Отдельно классифицировать опубликованные события без подтверждённого внутреннего потребителя. | `TenantUpdated`, `TenantStatusChanged`, события пользователей, ролей и прав, `ObjectRuntime.MutationCompleted`, `Workflow.CommandExecuted` и `Workflow.InstanceReassigned` перечислены как published events; для каждого указано, найден ли внутренний handler, есть ли типизированный payload, кто владеет схемой и является ли событие текущим MVP-контрактом или точкой для будущих потребителей. | `02_architecture` / владельцы событий | Completed |
| `PCDOC-NORM-03` | Проверить нормативное правило для сквозной карты взаимодействий. | Strategy, Playbook и Template однозначно определяют: общая карта API/event-взаимодействий находится в `02_architecture/07_integration_architecture.md`; `03_platform/<area>` содержит локальные карты и подробности владельца; отдельный `05_contracts` не создаётся; runtime-поведение описывается в `04_runtime.md` соответствующей области. Одинаковые таблицы взаимодействий не дублируются в пакетах областей. | `00_governance`, `02_architecture`, `03_platform` | Completed |

## 4.5. Верхний концептуальный и архитектурный слой

Этот пакет закрывает переходные source-копии в `01_concept` и `02_architecture`.
Он касается только концептуальных и общих документов DMP. Детальные требования к
платформенным областям, Object Runtime, Configuration, Workflow, Rules,
Reporting/Output, Integration Events и прикладным модулям остаются у владельцев
`03_platform` и `04_domain_modules`.

| ID | Задача | Целевой результат | Владелец документа | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-CON-01` | Закрыть staging-папки `01_concept/01_product_concept.md/` и `01_concept/02_system_scope.md/`. | Вместо папок с source-копиями существуют нормализованные документы `01_product_concept.md` и `02_system_scope.md` с frontmatter; исходные материалы остаются в `docs/01 sources`. | `01_concept` | Completed |
| `PCDOC-CON-02` | Добавить недостающие концептуальные документы верхнего слоя. | Созданы `03_business_goals.md`, `04_stakeholders.md`, `05_success_metrics.md` и `06_system_composition.md`; документы не содержат детальных требований к ядру и модулям. | `01_concept` | Completed |
| `PCDOC-CON-03` | Закрыть staging-папку `02_architecture/01_architecture_overview.md/`. | Вместо папки с source-копией существует нормализованный `01_architecture_overview.md`; исходный `05_architecture.md` разобран по целевым документам. | `02_architecture` | Completed |
| `PCDOC-CON-04` | Создать общий пакет архитектуры DMP, кроме уже подготовленного `07_integration_architecture.md`. | Созданы `02_architecture_principles.md`, `03_logical_architecture.md`, `04_service_architecture.md`, `05_data_architecture.md`, `06_deployment_architecture.md` и `08_security_architecture.md`; `07_integration_architecture.md` остаётся канонической картой взаимодействий. | `02_architecture` | Completed |
| `PCDOC-CON-05` | Зафиксировать границу между верхней архитектурой и платформенными областями. | В документах `01_concept` и `02_architecture` есть только общая рамка и ссылки на `03_platform`; подробные контракты, runtime, эксплуатация и трассировка не дублируются. | `01_concept`, `02_architecture`, `03_platform` | Completed |
| `PCDOC-CON-06` | Проверить будущие каталоги `diagrams/` и `12_appendices/`. | Пустые каталоги не создаются; `diagrams/` появляется только при наличии диаграмм, `12_appendices/imported_materials/` используется только если принято хранить импортированные копии вне `docs`. | `02_architecture`, `12_appendices` | Open |
| `PCDOC-CON-07` | Исключить `05_contracts/` из целевой структуры. | Контракты описываются у владельцев в `03_platform` и `04_domain_modules`; старые материалы `docs/05 contracts` используются только как источники сверки и не переносятся в `docs-new` как отдельный тип документации. | `00_governance`, владельцы областей | Completed |
| `PCDOC-CON-08` | Зафиксировать назначение `07_operations/`. | Раздел не является обязательной частью текущего комплекта. Он допускается как будущий сквозной раздел эксплуатации, окружений, deployment, monitoring, incident response и platform-wide runbooks, но физически не создаётся без такого содержания и не дублирует `03_platform/*/08_operations.md`. | `00_governance`, `03_platform` | Completed |
| `PCDOC-CON-09` | Зафиксировать назначение `08_testing/`. | Раздел не является обязательной частью текущего комплекта. Он допускается как будущий сквозной раздел QA/test strategy, acceptance, contract/integration/E2E/security/performance testing и test data strategy, но физически не создаётся без такого содержания и не дублирует `03_platform/*/07_quality.md`. | `00_governance`, `03_platform` | Completed |
| `PCDOC-CON-10` | Зафиксировать назначение `09_decisions/`. | ADR не являются обязательной частью текущего комплекта проектной документации. Раздел `09_decisions` допускается как будущий механизм для самостоятельных архитектурных решений, но физически не создаётся без утверждённого ADR. Текущие решения ведутся в документах-владельцах, `90_traceability.md` и `10_backlog`. | `00_governance`, `10_backlog` | Completed |

## 4.6. Разбор оставшихся source/offer/requirements зон

Этот пакет фиксирует поштучный разбор переходных материалов, которые ещё
остаются в `docs-new`. Цель — для каждой зоны принять одно из решений:
перенести подтверждённое содержание в документ-владелец, оставить как активный
рабочий источник до отдельной миграции, перенести в appendix/reference или
удалить переходную копию после трассировки. Исходная папка `docs/` этим пакетом
не изменяется.

### 4.6.1. Правило для будущих возможностей

Будущие возможности, идеи, расширения и работы без подтверждённого текущего
владельца реализации не остаются в `offers/` как активные source-копии. Их
содержание переносится в `10_backlog` как рабочие пакеты с критериями
готовности.

`90_traceability.md` соответствующего владельца при этом хранит только короткий
маршрут: тема, статус `Будущая доработка` или `Владелец другой области`, ссылка
на backlog ID и документ, который нужно обновить после решения. Трассировка не
является архивом исходника и не должна требовать открыть удалённый `offers` или
старый `docs/...` файл, чтобы понять смысл будущей работы.

Если будущая возможность пока не имеет собственного владельца в `03_platform`,
она ведётся в `10_backlog` до появления решения о границе, номере каталога,
кодовой реализации и составе нормативного пакета.

| ID | Зона | Что нужно решить | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-SRC-01` | `03_platform/00_platform_core.md/` | Можно ли закрыть staging-копию `03_platform_core.md (01 sources).md` после нормализации Foundation и верхней архитектуры. | Все значимые тезисы Platform Core отражены в `01_concept`, `02_architecture`, `03_platform/00_foundation`, `90_traceability.md` или backlog; активные ссылки на staging-путь отсутствуют либо переведены на целевые документы; после этого переходная папка удалена. | Completed |
| `PCDOC-SRC-02` | `03_platform/06_integration/offers/` | Оставлять ли offer до создания отдельной Integration Capability или перенести в backlog/appendix. | Решение: future-content перенесён в [Integration Capability backlog](../../platform_capabilities/integration_capability_backlog.md); текущий нормативный владелец outbox/envelope — `10_integration_events`, будущие external systems/connectors/flows/mappings — backlog до утверждения отдельной capability; переходная папка удалена. | Completed |
| `PCDOC-SRC-03` | `03_platform/07_event_foundation/offers/` | Можно ли закрыть переходный Event Foundation после подготовки `10_integration_events`. | Подтверждён маршрут содержания в `10_integration_events`, активные ссылки переведены, `PCDOC-IE-05` закрыт. | Completed |
| `PCDOC-SRC-04` | `03_platform/09_common_application_contracts/offers/` | Как разнести общий API/contracts source между Foundation и владельцами областей. | Решение: source-документ признан устаревшим обзором; открытые решения перенесены в [backlog открытых решений Platform API и контрактов](../../contract_governance/platform_api_contracts_backlog.md); Foundation владеет общими application/result/context/stable-code формами, Object Runtime — object/action/workflow API semantics; отдельный `05_contracts` не создаётся; переходная папка удалена. | Completed |
| `PCDOC-SRC-05` | `06_runtime/*` source/offers/requirements | Нужен ли отдельный runtime-пакет и какие файлы становятся нормативными runtime conventions. | Решение: отдельный пакет `06_runtime` исключён из целевой структуры. Runtime-поведение описывается в `04_runtime.md` конкретных platform areas; общие application/context/domain primitives — в Foundation; frontend runtime — во Frontend Platform; configuration/settings runtime — у своих владельцев; нерешённые сквозные вопросы остаются в `90_traceability.md` владельцев и backlog. Staging-папка удалена. | Completed |
| `PCDOC-SRC-06` | `04_domain_modules/*/offers` и папки-шаблоны `.md/` | Что относится к шаблону модульной документации, индексу модулей и offer-зонам отдельных модулей. | Решение: `00_module_documentation_template.md/` нормализован в настоящий файл `00_module_documentation_template.md`; `00_module_index.md/` закрыт и заменён на `01_module_index.md`; root/module offers обрабатываются миграцией модульной документации, а предметные module offers остаются у владельцев модулей до их трассировки. | Completed |
| `PCDOC-SRC-07` | `05_contracts/offers/` | Нужен ли отдельный contract staging в `docs-new`. | Решение: не нужен. Смысл Report Service уже покрыт `03_platform/11_reporting_output`, Tenant Security — `03_platform/01_tenant_and_security`, Rule/Condition — `03_platform/05_rules` и `03_platform/02_configuration`; старые machine/source-файлы остаются в `docs/05 contracts` как источники сверки, а копии в `docs-new` удалены. | Completed |
| `PCDOC-SRC-08` | `09_decisions/* (adr drafts).md` | Нужны ли старые ADR drafts в текущем `docs-new`. | Решение: не нужны. Файлы `* (adr drafts).md` были точными копиями старых `docs/adr/drafts`, а не целевыми ADR. Копии удалены из `docs-new`; старые drafts остаются в `docs/adr/drafts` как источники сверки. | Completed |

## 4.7. Закрытие переходного пакета `06_runtime`

| ID | Тема | Решение | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-RUN-01` | Базовые runtime conventions | Не создавать отдельные документы `06_runtime`; подтверждённые primitives/context/result/domain-event сведения уже находятся в Foundation, tenant/security context — в Tenant/Security, runtime-сценарии — в `04_runtime.md` владельцев. | В стратегии нет целевого `06_runtime`; source-копия `14_platform_runtime_conventions.md` не хранится в `docs-new`. | Completed |
| `PCDOC-RUN-02` | Reference patterns | Не переносить как нормативный документ: это инженерные примеры и шаблоны реализации, а не проектное решение. При необходимости правила модулей уточняются в `04_domain_modules/00_module_documentation_template.md`. | Source-копия `14.1_platform_runtime_reference_patterns.md` удалена из `docs-new`; старый источник остаётся в `docs/01 sources`. | Completed |
| `PCDOC-RUN-03` | Runtime offers из `docs/04 runtime` | `configuration_scope_version_pipeline.md` и `runtime_data_flow.md` уже покрыты у Configuration, Object Runtime и Frontend Platform либо остаются в их `90_traceability.md`; `runtime_settings_authoring_guide.md` покрыт Settings и `SET-DEC-*`. | `docs-new/06_runtime/offers` удалён; в трассировках нет ссылки на staging-копии как источник истины. | Completed |
| `PCDOC-RUN-04` | Frontend runtime requirements | Не переносить в `06_runtime`; подтверждённое содержание уже в `03_platform/12_frontend_platform`, будущие темы — `FE-DEC-*`. | `docs-new/06_runtime/requirements` удалён; Frontend Platform traceability остаётся владельцем. | Completed |
| `PCDOC-RUN-05` | Закрыть source/offers/requirements в `06_runtime` | Удалить весь переходный каталог после правки стратегии, concept/architecture и migration review. | `Test-Path docs-new/06_runtime` возвращает `False`, активные ссылки переведены. | Completed |

## 4.8. Модульная документация и source/offers

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-MOD-01` | Нормализовать индекс прикладных модулей. | `04_domain_modules/01_module_index.md` | Source-копия `04_module_map.md` сверена с текущим утверждённым составом `01_concept/06_system_composition.md`; platform blocks не смешаны с domain modules; staging-каталог `.md/` удалён, source-копия в `docs-new` не хранится. | Completed |
| `PCDOC-MOD-02` | Нормализовать шаблон документации прикладного модуля. | `04_domain_modules/00_module_documentation_template.md` | Текущий шаблон и source `16_module_architecture_standard.md` сведены без противоречий; стандарт модульной архитектуры не дублирует Foundation, Object Runtime, Workflow и Rules; staging-каталог `.md/` удалён, source-копия в `docs-new` не хранится. | Completed |
| `PCDOC-MOD-03` | Закрыть корневую переходную папку `offers` прикладных модулей. | Шаблон документации прикладного модуля и общая интеграционная архитектура | Reference map не переносится как нормативный документ: это инструкция по demo/reference-коду, а исходный материал остаётся в `docs/01 sources`; правила межмодульного взаимодействия закреплены в шаблоне прикладного модуля и общей интеграционной архитектуре. Переходная папка удалена, активных ссылок на неё нет. | Completed |
| `PCDOC-MOD-04` | Разобрать переходные материалы прикладных модулей. | Реестр `PCDOC-MOD-04.1` — `PCDOC-MOD-04.5` ниже | Offer Product Definition удалён после сверки и маршрутизации содержания в документы модуля 02; оставшиеся материалы обрабатываются последовательно по отдельным записям. | In progress |

### Реестр кандидатов на удаление

В этот реестр попадают только копии и материалы внутри `docs-new`, которые
могут быть закрыты после проверки содержания. Файл или каталог удаляется после проверки содержания, записи маршрута в
канонических документах или backlog и проверки отсутствия активных ссылок.
Исходные материалы в старом каталоге `docs` этой процедурой не удаляются.

| ID | Кандидат | Что в нём находится | Условие удаления | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-MOD-04.1` | `04_domain_modules/00_common/offers/` | Дубликат плана `TimeAmountFormat`; актуальная копия находится в `plans/030_time_amount_format_implementation_plan.md`, а смысл уже отражён в документах `00_common`. | Дубликат удалён; актуальный план и документы Common сохранены; активных ссылок на удалённую копию нет. | Completed |
| `PCDOC-MOD-04.2` | `04_domain_modules/08_planning_scheduling/offers/` | Два содержательных исходных материала будущего Planning & Scheduling. | Сначала создать пакет документации или отдельный backlog будущего модуля и распределить содержание; до этого удалять нельзя. | Open |

Папки `_working/` являются обязательной рабочей частью каждого модуля и в
реестр кандидатов на удаление не включаются. Их содержание можно разбирать и
переносить в канонические документы, но сами папки сохраняются. Это относится
к `00_common`, `02_product_process_definition`, `03_plant_structure` и
`04_resource_management`.

Source-копия `04_module_map.md` закрыта задачей `PCDOC-MOD-01`; исходник
остаётся в старом каталоге `docs/01 sources`, а целевой индекс находится в
`04_domain_modules/01_module_index.md`. Source-копия
`16_module_architecture_standard.md` закрыта задачей `PCDOC-MOD-02`; исходник
остаётся в старом каталоге `docs/01 sources`.

## 4.9. Закрытие `05_contracts`

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-CONTRACT-01` | Проверить Report Service source package. | `docs/05 contracts/report_service/*`, `03_platform/11_reporting_output` | Нормативный смысл рендеринга, preview, ошибок, quality и operations уже покрыт владельцем Reporting/Output; копии OpenAPI/JSON Schema/XSD/examples не создают новый раздел `docs-new` и остаются в старом `docs` как источники сверки. | Completed |
| `PCDOC-CONTRACT-02` | Проверить Tenant Security catalog source package. | `docs/05 contracts/tenant_security/*`, `03_platform/01_tenant_and_security` | Security catalog, manifest, localization, inventory и SQL snapshot сверены по смыслу с Tenant/Security; копии в `docs-new` не нужны. | Completed |
| `PCDOC-CONTRACT-03` | Проверить rule and condition model source. | `docs/05 contracts/rule_and_condition_model_platform_contract.md`, `03_platform/05_rules`, `03_platform/02_configuration` | Подтверждённые runtime-границы находятся у Rules; schema/design model — у Configuration; неподтверждённые возможности не объявлены MVP. Копия в `docs-new` удалена. | Completed |
| `PCDOC-CONTRACT-04` | Закрыть `05_contracts` как целевой раздел. | `docs-new/05_contracts/` | Верхний раздел и новые contract artifact folders не используются; активные ссылки переведены на документы владельцев и старые источники `docs/05 contracts`. | Completed |

## 4.10. ADR в текущем пакете

| ID | Рабочий пакет | Файлы или результат | Критерий готовности | Статус |
| --- | --- | --- | --- | --- |
| `PCDOC-ADR-01` | Решить, обязательны ли ADR для текущего комплекта. | `00_governance`, текущий backlog | Решение: не обязательны. Текущий комплект проектной документации ведётся через документы-владельцы, `90_traceability.md` и `10_backlog`. | Completed |
| `PCDOC-ADR-02` | Не нормализовать старые ADR drafts автоматически. | `docs/adr/drafts/*` | Старые drafts остаются вне `docs-new` как источники; дубль `ADR-011`, конфликт `ADR-0014`/`ADR-014` и статусы не блокируют текущий пакет, потому что drafts не являются целевыми ADR. | Completed |
| `PCDOC-ADR-03` | Не создавать физический `docs-new/09_decisions` без утверждённого ADR. | `docs-new/09_decisions` | `Test-Path docs-new/09_decisions` возвращает `False`; раздел появится только после отдельного решения о конкретном ADR. | Completed |
| `PCDOC-ADR-04` | Оставить ADR как допустимый будущий механизм. | `00_documentation_strategy.md` | Стратегия допускает `09_decisions/*.md` только если раздел создан отдельным решением; draft-копии не переносятся автоматически. | Completed |
| `PCDOC-ADR-05` | Проверить, что source-суффиксы ADR не остались в `docs-new`. | `docs-new` | Активные `docs-new/09_decisions/* (adr drafts).md` отсутствуют; старые `docs/adr/drafts` сохранены. | Completed |
| `PCDOC-ADR-06` | Удалить из `docs-new` draft-копии, которые не стали утверждёнными ADR. | `docs-new/09_decisions/* (adr drafts).md`, `docs/adr/drafts/*` | Старые draft-файлы остаются в `docs/adr/drafts` как источники; в `docs-new/09_decisions` больше нет source-копий. В будущем сюда добавляются только утверждённые ADR с frontmatter, нормальной нумерацией и ссылками из документов-владельцев. | Completed |

### Следующие области

После завершения текущего пакета Configuration порядок определяется отдельным
решением по зависимостям и готовности источников. Кандидаты: Frontend Platform,
Workflow, Rules, Value Sets, Audit History и Integration Events. Для каждой
новой области сначала создаётся такой же рабочий пакет, а затем выполняется
Playbook с контрольными точками `Нужно вычитать` и `Нужно решение`.

## 5. Зависимости

| Зависимость | Почему важна |
| --- | --- |
| `03_platform/01_tenant_and_security` | Эталон применения стандарта к одной платформенной области. |
| `03_platform/00_platform_documentation_template.md` | Правила структуры, владельцев информации, терминологии и ссылок на открытые решения. |
| `03_platform/12_frontend_platform` | Будущий владелец общей frontend-платформы, трёх приложений и runtime artifacts. |
| `03_platform/02_configuration` | Целевой пакет Configuration Platform; переходная папка закрыта после разбора исходных материалов. |
| `docs/03 conf_artefacts` | Старые, но содержательные описания конфигурационных артефактов; используются как источник для `artifact_types/`, а не склеиваются в один общий документ. |
| `docs/01 sources/07_tenant_and_security_platform.md` и `docs/02 requrements/01_*` | Старые исходные материалы для будущей концепции и целевых решений; не копируются внутрь Tenant/Security pilot package. |
| `10_backlog/roadmap/preparation/09_two_week_platform_architecture_outcome_requirements.md` | Описывает ожидаемый результат двухнедельного архитектурного этапа. |
| `11_glossary/platform_terms.md` и `11_glossary/security_terms.md` | Терминологическая база для Platform Core и Tenant/Security. |

## 6. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 17:56 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: объединить master и обновления PR-статусов | [12a94ce7](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/12a94ce7f297c36f0d543918eb0e9ca8be294813) |
