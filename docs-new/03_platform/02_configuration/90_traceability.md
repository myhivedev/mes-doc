---
id: DOC-03-02-90
title: 'Трассировка — Configuration'
type: traceability
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
updated_at: 2026-09-03 14:49
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Трассировка — Configuration

[project]: ../../../src/Platform/DMP.Platform.Configuration/
[contracts]: ../../../src/Platform/DMP.Platform.Contracts/Configuration/
[api]: ../../../src/Platform/DMP.Platform.Configuration/Api/Controllers/
[canonical]: ../../../src/Platform/DMP.Platform.Configuration/Domain/Canonical/
[schemas]: ../../../src/Platform/DMP.Platform.Configuration/Domain/Schemas/
[security]: ../../../src/Platform/DMP.Platform.Configuration/Infrastructure/Security/ConfigurationSecurityCatalogManifest.cs
[runtime-navigation]: ../../../src/Frontend/packages/runtime-react/src/model/runtimeNavigationAdapter.tsx
[backlog]: ../../10_backlog/roadmap/preparation/platform_core_documentation_backlog.md

## 1. Назначение документа

Документ показывает, какие группы исходных материалов учтены при создании целевой области Configuration, что подтверждено текущим кодом MVP, какие расхождения требуют решения и куда переносится принятое содержание.

Трассировка не является архивом исходных файлов и не хранит утверждённые ссылки на старые документы как источник истины. Она нужна, чтобы старые требования, предложения и содержательные описания артефактов не потерялись при переносе в новый пакет.

В колонках `Документ-владелец` и `Куда перенести` ссылки имеют практический смысл:
ссылка на [`artifact_types/`](artifact_types/) ведёт к каталогу спецификаций
конфигурационных артефактов, а ссылка на конкретный файл, например
[`object_type.md`](artifact_types/object_type.md), ведёт к спецификации одного
типа. Эти ссылки показывают, где находится результат переноса или подробное
описание, но не создают нового владельца и не означают, что Configuration
исполняет соответствующий runtime.

## 2. Источники и требования

| Группа источников | Требование или тезис | Решение | Документ-владелец | Покрытие |
| --- | --- | --- | --- | --- |
| Configuration overview и data model | Configuration хранит версии, области, артефакты, свойства, локализации, override и эффективную конфигурацию. | Подтверждённую модель перенести в `02_architecture.md` и `04_runtime.md`; не объявлять source-тезисы без кода текущей гарантией. | `02_architecture.md`, `04_runtime.md` | Частично перенесено |
| Infrastructure/persistence и scope/version pipeline | Published configuration должна быть воспроизводимой; runtime использует только published/effective chain. | Закрепить `SystemBaseline` и точную parent chain как базовое решение; риски cache/warm-up/observability вынести в quality/operations. | `04_runtime.md`, `07_quality.md`, `08_operations.md` | Частично перенесено |
| Settings decision | Settings catalog, snapshot, runtime values, override и user preferences должны иметь отдельные границы. | В Configuration фиксируется только snapshot-связь и использование каталога; полная модель Settings и пользовательских предпочтений не переносится без решения владельца. | `02_architecture.md`, `04_runtime.md`, `CFG-DEC-04` | Частично; граница владельца не закрыта |
| Старый каталог артефактов | Каждый тип артефакта имеет самостоятельную структуру и смысл. | Классификация принята: не объединять в один документ; переносить в `artifact_types/` по одному типу после вычитки и сверки с `ConfigurationArtifactTypeCodes`/schemas. | [каталог спецификаций](artifact_types/), [03_contracts.md](03_contracts.md) | В работе; перенос не завершён |
| Condition model и editor requirements для Workflow/Action/Rule | Условия, expression format, bindings и editor UX должны быть единообразны. | Подтверждённые schema-поля описаны у `Workflow`, `Action` и `Rule`; исполнение Workflow описывается в [пакете Workflow](../04_workflow/00_platform_overview.md), а исполнение Rules относится к владельцу Rules. | [`workflow.md`](artifact_types/workflow.md), [`action.md`](artifact_types/action.md), [`rule.md`](artifact_types/rule.md), `CFG-DEC-06` | Перенесено для текущей границы; единый язык условий, editor UX и общий runtime-контракт остаются отдельными темами своих владельцев |
| UI/Studio requirements | Нужны explorer, artifact editor, context/version management и сценарии Studio. | В Configuration описывать server/editor contracts и Configuration-specific сценарии; общий shell и app runtime отнести к фронтенд-платформа. | `06_user_experience.md`, `12_frontend_platform` | Частично |
| Workflow/Rules/Report/Output/ValueSet/Numbering sources | Эти темы представлены как конфигурационные артефакты и как самостоятельные runtime capabilities. | В Configuration оставить definitions/schema/public references; исполнение и runtime guarantees передать соседним областям. | `01_scope.md`, [каталог спецификаций](artifact_types/), соседние области | Частично |
| Report/Output/Numbering artifact sources | `Report`, `Output` и `NumberingRule` имеют отдельные schema и lifecycle details. | `Report`, `NumberingRule` и `Output` оформлены в отдельных спецификациях; runtime-граница `Output` описана с отмеченными расхождениями schema/editor/runtime. | `03_contracts.md`, `04_runtime.md`, [`report.md`](artifact_types/report.md), [`numbering_rule.md`](artifact_types/numbering_rule.md), [`output.md`](artifact_types/output.md), `PCDOC-14.5` | Частично перенесено |
| Code MVP | Project, contracts, controllers, canonical model, schema registry, security manifest и migrations существуют. | Использовать код как источник приоритета 1; source-копии служат только покрытием и списком открытых решений. | Все документы пакета | Подтверждено |

## 3. Принятые решения

| Решение | Подтверждённый вариант | Основание | Документ-владелец |
| --- | --- | --- | --- |
| Граница области | Configuration владеет authoring/versioning/publishing/effective configuration, но не исполняет настраиваемое поведение. | [project][project]; [contracts][contracts]; [api][api] | [Граница](01_scope.md) |
| Каноническая модель | Основой остаются `ArtifactDocument`, `ArtifactNode`, `ArtifactValue`, `EffectiveArtifactDocument`. | [canonical][canonical] | [Обзор](00_platform_overview.md), [Архитектура](02_architecture.md) |
| Schema registry | Типы артефактов ведутся через `ConfigurationArtifactTypeCodes` и schemas. | [schemas][schemas] | [Архитектура](02_architecture.md), [Контракты](03_contracts.md), [каталог спецификаций](artifact_types/) |
| Переходная папка | `02_configuration_platform/` закрыта по `PCDOC-14.18`; переходные копии удалены и не считаются частью целевого пакета. | [backlog][backlog] | [Обзор](00_platform_overview.md) |
| Каталог артефактов | Старый каталог артефактов переносится не склейкой, а отдельными файлами в `artifact_types/`. | [backlog][backlog] | [Контракты](03_contracts.md), [каталог спецификаций](artifact_types/) |
| Production parent chain | Non-root scope не создаёт draft без опубликованной parent-base version; effective runtime дополнительно проверяет parent chain. | [project][project] | [Архитектура](02_architecture.md), [Исполнение](04_runtime.md), [Эксплуатация](08_operations.md) |
| Послойное владение артефактом | Один артефакт может использоваться Configuration, runtime, frontend и domain module, но подробное описание каждого слоя не дублируется. | Каноническая модель и schema принадлежат Configuration; runtime-интерпретация — владельцу capability; рендерер и приложения — фронтенд-платформа; предметный смысл — domain module. | [Граница](01_scope.md), [каталог спецификаций](artifact_types/), Playbook | Принято |

## 4. Расхождения и открытые решения

Ниже находятся как открытые решения, так и уже согласованные решения, перенос которых ещё не завершён. Для согласованного решения в колонке `Решение требуется` фиксируется принятый вариант; после переноса результата в документ-владелец запись остаётся здесь как маршрут трассировки и больше не считается открытым вопросом.

| ID | Тема | Статус сведения | Что ожидалось в источниках | Что подтверждено в MVP | Решение требуется | Владелец | Куда перенести |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `CFG-DEC-01` | Состав `artifact_types/` | `Принято целевое решение` | Все конфигурационные root-артефакты и их связанные типы описаны отдельно. | Коды типов подтверждены schema registry; root/embedded/reference классификация согласована для текущего пакета. ([schemas][schemas]) | **Принято:** отдельные документы создаются для `ObjectType`, `View`, `Action`, `Workflow`, `Rule`, `ValueSet`, `SystemEnum`, `Menu`, `Report`, `Output`, `NumberingRule`; дочерние типы описываются у владельца. `Navigation` является переходным названием для `Menu`; `GridUi` и `Lookup` остаются embedded-моделями или сценариями внутри владельцев; `EnumPresentation` не становится отдельным артефактом без подтверждения schema registry. `Dataset` не становится отдельным артефактом без нового решения. | Configuration | `01_scope.md`, `03_contracts.md`, [каталог спецификаций](artifact_types/) |
| `CFG-DEC-02` | Report design-time boundary | `Открытый вопрос` | Report/Output описаны внутри Configuration source. | В Configuration есть report contract/design/content endpoints; runtime execution отдельный. ([api][api]) | Развести design-time Configuration и runtime Reporting Output. | Configuration / Reporting Output | `03_contracts.md`, `11_reporting_output` |
| `CFG-DEC-03` | Dataset / Read Query owner | `Открытый вопрос` | Dataset используется View/Report/Output и должен стать общим read contract. | В contracts есть `ConfigurationDatasetRegistration`; общий read query owner не оформлен. ([contracts][contracts]) | Закрепить владельца Dataset / Read Query Capability. | Architecture / Object Runtime / Configuration | `01_scope.md`, `02_architecture.md` после выбора владельца |
| `CFG-DEC-04` | Settings boundary | `Открытый вопрос` | Settings упоминаются в configuration version snapshot. | В коде есть settings catalog snapshot integration; отдельный Settings project существует. ([project][project]) | Описать, что остаётся snapshot-связью Configuration, а что принадлежит Settings. | Configuration / Settings | `02_architecture.md`, `07_settings` |
| `CFG-DEC-05` | Frontend Studio boundary | `Открытый вопрос` | Старые UI requirements подробно описывают shell/editor/screens. | Configuration-specific Studio features и общий shell/apps уже существуют в коде; отдельный документ и окончательная граница фронтенд-платформа ещё не оформлены. | Завершить разделение Configuration-specific editor UX и общей фронтенд-платформа. | Configuration / фронтенд-платформа | `06_user_experience.md`, `12_frontend_platform` |
| `CFG-DEC-06` | Workflow/Rules execution boundary | `Принятое целевое решение` | Workflow и Rules описаны внутри configuration sources. | Artifact schemas/builders принадлежат Configuration, а runtime-владельцы отдельные. ([schemas][schemas]) | **Принято:** Configuration владеет definition/schema артефактов `Workflow` и `Rule`; Workflow владеет исполнением Workflow, а Rules — исполнением правил. В Configuration остаются ссылки на runtime-владельцев без дублирования их алгоритмов. | Configuration / Workflow / Rules | [`workflow.md`](artifact_types/workflow.md), [`rule.md`](artifact_types/rule.md), `04_workflow/`, пакет Rules |
| `CFG-DEC-07` | Audit/change boundary | `Открытый вопрос` | Source говорит об аудите изменений конфигурации. | Есть `ConfigurationChange`; общий Audit History отдельный. ([project][project]) | Развести локальный журнал изменений Configuration и platform audit/history. | Configuration / Audit History | `05_security_and_audit.md`, `09_audit_history` |
| `CFG-DEC-08` | Послойное владение конфигурационными артефактами | `Принято целевое решение` | Старые документы объединяют schema, runtime, frontend и предметные сценарии в одном описании. | Каноническая модель и schema уже находятся в Configuration; runtime materialization и frontend-потребление существуют отдельными контрактами, местами переходными. | **Принято:** документ типа артефакта описывает canonical/schema contract и Configuration lifecycle; runtime, editor UX, рендерер фронтенда и предметные сценарии описываются у своих владельцев с короткой ссылкой из artifact doc. | Configuration / Object Runtime / фронтенд-платформа / Domain Modules | Playbook, template, [каталог спецификаций](artifact_types/), backlog `PCDOC-16` |
| `CFG-DEC-09` | Регистрация `TemplateCode` и рендерер | `Принято целевое решение` | `View.TemplateCode` связывает конфигурацию с визуальным шаблоном. | Backend регистрирует код и его допустимые `ViewType`; фронтенд-платформа регистрирует рендерер для поддерживаемых приложений. | **Принято:** каждый новый `TemplateCode` добавляется в backend-каталог вместе с ограничением `ViewType`, получает рендерер фронтенда и покрывается проверкой совместимости. Текущий backend-каталог содержит `ObjectFormTemplate`; желаемое имя `FormTemplate` в текущем коде не зарегистрировано и потребует отдельной согласованной правки кода. | Configuration / фронтенд-платформа | [`view.md`](artifact_types/view.md), `12_frontend_platform` |
| `CFG-DEC-10` | View query и runtime contract | `Открытый вопрос` | `View` может ссылаться на dataset и задавать list/filter/paging/sorting semantics. | Schema и editor model существуют; Object Runtime теперь поддерживает abstract-reference lookup и concrete object identity, но общий `Read Query Capability`, production query rules и runtime contracts Dashboard/Workspace не оформлены. | Закрепить владельца query/runtime contract и описать его отдельно от schema `View`. | Architecture / Object Runtime / фронтенд-платформа | [`view.md`](artifact_types/view.md), `02_architecture.md` после выбора владельца |
| `CFG-DEC-11` | ValueSet defaults и `DisplayType` projection | `Открытый вопрос` | Definition задаёт defaults и default presentation для ValueSet. | Schema не задаёт default для `AllowTenantOverrides`; baseline builder записывает `false`, а published projection использует fallback `true` при отсутствии значения. `DisplayType` валидируется в schema, но текущая projection не переносит его в `ValueSetDataSet`. | Выбрать единый default `AllowTenantOverrides` и определить, должен ли `DisplayType` входить в published Value Sets projection/runtime response. | Configuration / Value Sets | [`value_set.md`](artifact_types/value_set.md), `02_architecture.md`, `04_runtime.md` |
| `CFG-DEC-12` | Различие policy `Configurable` и `Overrideable` | `Открытый вопрос` | Policy должна различать обычное редактирование и scoped override. | Static catalog содержит оба кода, но текущий Value Set Data Editor явно блокирует только `Fixed`; отдельная runtime-проверка различия `Configurable`/`Overrideable` не подтверждена. | Определить семантику двух policy и реализовать отдельную проверку, если различие требуется. | Value Sets / Configuration | [`value_set.md`](artifact_types/value_set.md), `04_runtime.md`, `05_security_and_audit.md` |
| `CFG-DEC-13` | Владение `SystemEnum` и разрешённое переопределение | `Принято целевое решение` | Системное перечисление должно задаваться кодом, а Configuration — менять только текстовые заголовки существующих значений. | Текущая schema содержит дополнительные поля `SourceType`, `SourceCode`, `Policy`, `DisplayType`, `Tone`, `IconCode` и `Variant`; общий редактор не выделяет для `SystemEnum` отдельную политику «только `Title`». | **Принято:** код владеет составом и техническими свойствами `SystemEnum`; редактор Configuration переопределяет только локализованный `SystemEnumValue.Title`; отдельный механизм получения значений не вводится. Лишние поля schema и специальное ограничение редактора оформить отдельной технической задачей. | Configuration | [`system_enum.md`](artifact_types/system_enum.md), `03_contracts.md`, `04_runtime.md` |
| `CFG-DEC-14` | Обработка target-типов `Menu` во frontend | `Открытый вопрос` | Пункт `NavigationItem` должен открывать `View`, внутренний маршрут, внешний URL или запускать `Action` в зависимости от `TargetType`. | Schema и runtime contract передают все четыре target-типа и их поля. Общий frontend navigation adapter сейчас строит переходы для `View` и `Route`; единое поведение для `ExternalUrl` и запуска `Action` не определено. ([runtime navigation][runtime-navigation]) | Решить, какие приложения и какой общий frontend-механизм обрабатывают `ExternalUrl` и `Action`, включая безопасность внешних адресов и параметры запуска действия. | фронтенд-платформа / приложения / Object Runtime | [`menu.md`](artifact_types/menu.md), `06_user_experience.md`, будущая фронтенд-платформа area |
| `CFG-DEC-15` | Покрытие `NumberingRule` runtime | `Расхождение кода и источника` | В общем наборе кодов объявлен `OnSave`, а переходные материалы и часть ожидаемого контракта предполагают более широкий набор моментов и операторов условий. | Текущий активный каталог вариантов, metadata V1 и object mutation hook подтверждают только `OnCreate`; runtime evaluator обрабатывает `Equals`, `NotEquals`, `In`, `IsNull`, `IsNotNull`, `Contains` и `StartsWith`. Кроме того, schema и root editor используют `InheritanceMode = Override`, а validation projection при отсутствии свойства подставляет `Inherit`. | Решить, оставить ли `OnSave` только как будущий код или расширить до него каталог, validator и runtime. Отдельно выровнять или явно закрепить различие default `Override` и validation fallback `Inherit`. До отдельного решения `OnSave`, остальные неподтверждённые варианты и единое runtime-поведение `InheritanceMode` нельзя описывать как гарантии MVP. | Configuration / Platform Numbering / Object Runtime | [`numbering_rule.md`](artifact_types/numbering_rule.md), `04_runtime.md`, backlog `PCDOC-14.9` |
| `CFG-DEC-16` | Покрытие `Output` runtime | `Расхождение schema/editor/runtime` | Старые материалы описывают несколько источников, каналов доставки, фоновые режимы, параметры и audit/security options. | Schema registry и editor поддерживают эти поля и статические каталоги; текущий runtime исполняет только `SourceType = Report`, режимы `Inline`/`Download`, mapping-источники частично и file name options частично. | Решить, какие значения объявляются MVP, а какие реализуются позже или удаляются/сужаются в schema. До решения не считать `Dataset`, `View`, `Background`, `External`, delivery channels, execution/security options, `Transform`, `Selection`, `ViewState` и `Expression` поддержанными runtime-гарантиями. | Configuration / Reporting Output / фронтенд-платформа | `artifact_types/output.md`, `03_contracts.md`, `04_runtime.md`, backlog `PCDOC-14.9` |

| `CFG-DEC-17` | Перенос Configuration между окружениями | `Будущая capability` | Старые требования описывают export по области, import с validation, preview конфликтов, merge/replace и перенос между test/staging/production. | В текущем Configuration подтверждены baseline package import и `Report design export/import`; общего environment export/import contract, conflict preview и deployment rollback не подтверждено. | Определить единицу переноса, формат пакета, совместимость, конфликтную стратегию, права, аудит, идемпотентность и rollback; затем оформить отдельный публичный контракт и operations runbook. | Configuration / Platform Operations | `03_contracts.md`, `08_operations.md`, backlog `PCDOC-14.15` |
| `CFG-DEC-18` | Общий формат условий | `Частично реализовано; общая унификация не принята` | Переходный документ `06_condition_model.md` предлагает единый JSON-формат для разных механизмов условий. | JSON `ConditionJson` валидируется Configuration и выполняется Platform Numbering; `ConditionExpression`, schema conditions, runtime filters и `RuleLogic` используют отдельные реализованные модели. | Не считать переходный документ общим нормативным стандартом. При необходимости отдельно согласовать унификацию форматов и evaluator; до этого документация описывает условия у их фактических владельцев. | Configuration / Platform Numbering / Object Runtime / Rule Engine | `artifact_types/numbering_rule.md`, `../08_numbering/90_traceability.md`, backlog `PCDOC-14.17` |

## 5. Маршрут в целевые документы

| Группа материалов | Что подтверждает | Документ-владелец | Состояние переноса |
| --- | --- | --- | --- |
| [Configuration project][project] | Components, persistence, services, migrations и implementation boundary. | `02_architecture.md`, `04_runtime.md`, `08_operations.md` | Частично перенесено |
| [Configuration contracts][contracts] | Requests/responses, baseline packages, registrations и enums. | `03_contracts.md` | Перенесено на уровне карты контрактов |
| [Configuration API][api] | HTTP routes, permissions и command/query boundary. | `03_contracts.md`, `05_security_and_audit.md` | Частично перенесено |
| [Configuration security manifest][security] | Resources, permissions, описания системных ролей и grant policies Configuration. | `05_security_and_audit.md` | Перенесено на уровне карты |
| Frontend Studio packages | Context/version management, explorer, artifact editor и локализация состояний. | `06_user_experience.md` | Перенесено на уровне карты |
| Configuration integration tests | Lifecycle, baseline, schema, editor, effective resolution и specialized artifact checks. | `07_quality.md` | Перенесено на уровне карты |
| Configuration validation pipeline | Проверки пакета, схемы, версии и операции до публикации, включая явно неподтверждённые гарантии. | `04_runtime.md`, `07_quality.md` | Матрица добавлена; полное покрытие всех контрактов и runtime-совместимости не объявляется |
| Persistence initializer, bootstrap и migrations | Database initialization, catalog seed, bootstrap/import и schema evolution. | `08_operations.md` | Перенесено на уровне карты |
| [Canonical model][canonical] | `ArtifactDocument`, `ArtifactNode`, values и эффективная конфигурация. | `02_architecture.md` | Перенесено |
| [Schema registry][schemas] | Artifact types, schema rules, property codes и value sources. | `02_architecture.md`, `03_contracts.md`, [каталог спецификаций](artifact_types/) | Все 11 спецификаций созданы; финальная вычитка и сверка отдельных расхождений ещё продолжаются |
| Старые source/requirements в переходной папке | Coverage, conflicts, UI/source expectations и соседние владельцы. | `90_traceability.md`, backlog, соседние области | Маршрут классифицирован; переходные копии удалены, старые исходники сохранены в `docs/` |
| Старый каталог артефактов | Содержательные спецификации типов конфигурационных артефактов. | [каталог спецификаций](artifact_types/) | Все 11 спецификаций созданы; финальная вычитка пакета ещё не завершена |
| Старый `ObjectType`: canonical/schema contract | Идентичность, root properties, `Members`, `MemberBehaviors`, `RuleBindings`, inheritance metadata и подтверждённые validation rules. | [`object_type.md`](artifact_types/object_type.md) | Перенесено в Configuration; требуется вычитка |
| Старый `ObjectType`: TPH, hierarchy и `CreateFromExisting` runtime semantics | Расчёт discriminator/hierarchy, mutation-ограничения, копирование объектов и lifecycle. | [`object_type.md`](artifact_types/object_type.md) только для configuration properties; целевой `03_object_runtime` для исполнения | Частично; runtime-владелец ещё не оформлен |
| Старый `ObjectType`: Artifact Editor operations | Read-only, create/delete/reorder и ownership/layer-правила редактора. | `06_user_experience.md`, editor contracts и `12_frontend_platform` для общего frontend | Частично; server-side schema перенесена, UX-маршрут требует доработки |
| Старый `ObjectType`: расширенные `RuleBinding` properties и `Display` | `EvaluationContext`, `Severity`, `FailurePolicy`, `ParameterMappings`, `SingularNominative` и другие поля старой спецификации. | [`rule.md`](artifact_types/rule.md), backlog или решение об исключении после сверки с кодом | Не подтверждено текущей schema; не переносить как MVP |
| Старый `Action`: schema и модель вызова | Идентичность, root properties, invocation, confirmation, parameters, input, result bindings, local behaviors, rule bindings, messages и schema operations. | [`action.md`](artifact_types/action.md) | Перенесено по текущему коду; требуется вычитка |
| Старый `Action`: размещение, runtime и исполнение | Placement во `View`, workflow-связи, availability, handler/DTO, mutation, audit и delivery semantics. | [`action.md`](artifact_types/action.md) только для границ; владельцы `View`, Object Runtime, Workflow, Audit History и Integration Events | Не является частью schema `Action`; переносится в документы владельцев |

## 6. Результат сверки спецификаций

Проверка выполнена по ревизии `origin/master@3e2037ac37683eef331d70223c6babfc61aa0539`.
Она проверяет наличие и сопоставление идентификаторов, но не заменяет смысловую
вычитку текста таблиц.

| Проверка | Результат | Вывод |
| --- | --- | --- |
| Коды типов артефактов | Все 50 кодов из `ConfigurationArtifactTypeCodes` встречаются в каталоге спецификаций | Пропущенных кодов типов не найдено |
| Зарегистрированные schema-типы | Зарегистрированы 47 групп схем; 11 корневых типов имеют отдельные файлы | `ViewElement` является общей группой свойств и не требует отдельного файла |
| Свойства schema | Все property codes, использованные зарегистрированными схемами, встречаются в документе владельца | Пропущенных имён свойств не найдено; обязательность, значения и влияние проверяются смысловой вычиткой |
| Статические значения | Значения из `ArtifactStaticOptionCatalog` и `ConfigurationStaticValueCodes` встречаются в пакете спецификаций | Наличие значения подтверждено; правильность привязки к конкретному свойству остаётся предметом таблиц артефактов |
| Источники значений | В документах отражены серверные, клиентские и реестровые источники там, где они заданы schema | `ArtifactValueSourceDefinition` не следует сводить к перечислению допустимых значений |
| Состав таблицы контрактов | В `03_contracts.md` добавлены пропущенные `ViewWidget` и `ViewFilterPresetValue` | Перечень вложенных типов теперь соответствует зарегистрированным группам View |

Проверка не обнаружила отсутствующих файлов спецификаций, неизвестных кодов типов
или неупомянутых property codes. Она также не объявляет все поля runtime-поддержанными:
полный schema-контракт и реально поддержанное runtime-подмножество по-прежнему
разделены в документах артефактов и в разделах расхождений выше.

## 7. Метаданные файлов

`DM-CAP-CONF-01..04,06` покрываются schema, static codes, baseline builders и
materializers; provider и binary flow находятся в отдельной
[Content capability](../13_content_storage/90_traceability.md).

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.2 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Метаданные файлов | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
