---
id: DOC-00-00-99
title: 'Реестр сопоставления исходной папки docs'
type: appendix
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: governance
module: governance
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 17:29
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
---

# Реестр сопоставления исходной папки docs

## 1. Назначение

Этот файл фиксирует методологическую карту переноса материалов из исходной папки `docs/` в целевую структуру `docs-new/`, заданную документом `00_documentation_strategy.md`.

Реестр является рабочим migration review, а не нормативным проектным
документом. Исторические пути, source-суффиксы и старые названия папок в нём
сохраняются только для трассировки миграции и не означают, что такие файлы или
папки существуют в текущем комплекте `docs-new`.

Цель реестра — не перенести файлы автоматически, а показать для каждого исходного материала:

| Что фиксируем | Зачем |
| --- | --- |
| Исходный документ | Чтобы не потерять ни один файл из `docs/`. |
| Место в структуре `00` | Чтобы перенос шел в целевую иерархию, а не в случайные папки. |
| Решение | Чтобы сразу видеть: перенос однозначный, спорный, требует разделения, является архивом или машинным контрактом. |
| Краткое содержание | Чтобы ревьюер быстро понимал смысл файла без открытия каждого документа. |
| Объяснение переноса | Чтобы было понятно, почему выбран именно этот раздел. |
| Вопрос | Чтобы спорные случаи не прятались внутри будущих документов. |

Контроль полноты: в таблицу внесены все 145 файлов из `docs/`. Исходная папка `docs/` этим реестром не изменяется.

Целевая структура берется из `docs-new/00_governance/00_documentation_strategy.md`:

| Раздел `docs-new` | Назначение при переносе |
| --- | --- |
| `00_governance/` | Правила ведения документации, стратегия, реестры миграции и traceability. |
| `01_concept/` | Продуктовая концепция, границы системы, модель состава и scope. |
| `02_architecture/` | Общая архитектура, слои, интеграционные принципы и архитектурные overview. |
| `03_platform/` | Платформенные capability: tenant/IAM, configuration, workflow, rules, value sets, integration, events, audit, object runtime и общие platform contracts. |
| `04_domain_modules/` | Прикладные модули и их документация по единому шаблону. |

| `07_operations/` | Будущий сквозной раздел эксплуатации; в текущем комплекте не создаётся без общего runbook/operations policy. |
| `08_testing/` | Будущий сквозной раздел QA/test strategy; в текущем комплекте не создаётся без общей стратегии проверки. |
| `10_backlog/` | Roadmap, gaps, открытые работы, требования и планы. |
| `11_glossary/` | Термины, допустимые формы, запрещенные синонимы и связи с документами. |
| `12_appendices/` | Архив, research, imported materials и вспомогательные материалы, которые не являются нормативной базой. |

### 1.1 Целевая структура `03_platform`

Цель: привести платформенную документацию к одинаковой структуре по всем platform capabilities.

Имена папок и файлов остаются на английском. Заголовки внутри документов пишутся по-русски. Английский термин добавляется в скобках, если он нужен для связи с кодом, API или общепринятой технической терминологией.

```text
docs-new/03_platform/
├── README.md
├── 00_foundation/
├── 01_tenant_and_security/
├── 02_configuration/
├── 03_object_runtime/
├── 04_workflow/
├── 05_rules/
├── 06_value_sets/
├── 07_settings/
├── 08_numbering/
├── 09_audit_history/
├── 10_integration_events/
├── 11_reporting_output/
└── 12_frontend_platform/
```

В каждой платформенной области используется один и тот же реестр документов из `00_platform_documentation_template.md`:

```text
00_platform_overview.md
01_scope.md
02_architecture.md
03_contracts.md
04_runtime.md
05_security_and_audit.md
06_user_experience.md
07_quality.md
08_operations.md
90_traceability.md
```

`00_platform_overview.md` и `01_scope.md` обязательны. Остальные документы создаются только при наличии предметного содержания; пустые файлы и пустые каталоги не создаются. `offers/` и `requirements/` остаются временной зоной импортированных исходных материалов и не входят в целевой комплект области.

#### Актуальное состояние удалённых переходных копий

Исторические таблицы ниже сохраняют исходные пути переходных копий, чтобы не
терялась история миграции. Эти пути не означают, что файлы существуют в текущем
дереве. Для поиска актуальной информации нужно использовать следующую таблицу.

| Удалённая переходная копия | Подтверждение удаления | Где находится актуальная информация | Где искать ещё не разобранные ожидания |
| --- | --- | --- | --- |
| Старый комплект `docs-new/03_platform/00_platform_core.md` и областей `01_tenant_and_security`–`10_object_runtime` — 49 документов | Коммит `7cbb9d0899de87c0d8b4b0890a22a3566b5191e1` от 2026-08-07 | Новые канонические области `docs-new/03_platform/00_foundation/`–`04_workflow/`; остальные области создаются по целевой структуре при наличии подтверждённого содержания | Исходные материалы в `docs/`, а нерешённые и будущие темы — в соответствующих `90_traceability.md` и `10_backlog/` |
| Промежуточные `Tenant/Security` `offers/`, `pilot_review_package/` и `requirements/` — 9 документов | Коммит `0acaa17f01d55f4bda65bf30ba7176cfb8e5253c` от 2026-08-25 | `docs-new/03_platform/01_tenant_and_security/` | Исходники и требования в `docs/`; исторические материалы не считаются действующими правилами без сверки с кодом |
| `docs-new/03_platform/02_configuration_platform/` — 47 source/offer/requirement-копий | Коммит `ca13b19bd17dd297927c1e66a97f95c29735b971` от 2026-08-26 | `docs-new/03_platform/02_configuration/`; отдельные темы Workflow, Reporting и Output — в соответствующих канонических областях | Исходные материалы в `docs/01 sources`, `docs/02 requrements` и `docs/03 conf_artefacts`; нерешённые темы — в traceability и backlog владельца |
| `docs-new/03_platform/10_object_runtime/` — 11 документов, включая offers и requirements | Коммит `09373981332b775cb15ad9e10a36e6587714e257` от 2026-08-26 | `docs-new/03_platform/03_object_runtime/`; будущие задачи — в `docs-new/10_backlog/roadmap/preparation/platform_core_documentation_backlog.md` | `docs/current/04_object_runtime_reference_module.md`, `docs/current/05_object_runtime_reference_relations_design.md`, `docs/other_tech/dmp-module-object-runtime.md`, исходные requirements и backlog-файлы в `docs/` |
| `docs-new/03_platform/03_workflow_engine/requirements/` — копия WF Runtime UI requirements | Удалено в текущей рабочей ветке; отдельного коммита ещё нет | Серверные сведения разобраны в `docs-new/03_platform/04_workflow/`; UI renderer, layout и пользовательские flow ожидают владельца Frontend Platform | `docs/02 requrements/06_wf_runtime_ui_requirements_and_flows_revised_list_ui.md` |
| `docs-new/03_platform/05_value_set_data/` — 3 переходные копии | Удалена в текущей рабочей ветке; отдельного коммита ещё нет | `docs-new/03_platform/06_value_sets/`; будущие темы — `PCDOC-VS-08` и `PCDOC-VS-09` | Исходники сохранены в `docs/01 sources/10_valuesets.md` и `docs/02 requrements/03_12_*_value_set*` |

Общий порядок поиска такой: сначала открывается канонический пакет новой области,
затем её `90_traceability.md` и backlog. Если нужной темы там нет, используется
исходный файл из `docs/`, на который ссылается эта таблица. Исходник является
историческим материалом и источником ожиданий, а не автоматически действующим
требованием. После анализа его подтверждённые части переносятся в нормативный
документ, будущие части — в backlog, а сведения другого владельца — в пакет этого
владельца.

#### Сравнение структуры из `00` и унифицированной структуры

##### Было — структура из `00_documentation_strategy.md`

```text
docs-new/03_platform/
├── 00_platform_core.md
├── 01_tenant_and_security/
│   ├── 01_overview.md
│   ├── 02_tenant_model.md
│   ├── 03_iam_model.md
│   └── 04_authorization_contracts.md
├── 02_configuration_platform/
│   ├── 01_overview.md
│   ├── 02_domain_model.md
│   ├── 03_data_configuration.md
│   ├── 04_workflow_configuration.md
│   ├── 05_rule_configuration.md
│   ├── 06_condition_model.md
│   ├── 07_ui_configuration.md
│   ├── 08_reporting_configuration.md
│   ├── 09_output_configuration.md
│   ├── 10_versioning_and_publish.md
│   └── 11_import_export.md
├── 03_workflow_engine/
├── 04_rule_engine/
├── 05_value_set_data/
├── 06_integration/
├── 07_event_foundation/
├── 08_audit_history/
├── 10_object_runtime/
└── 11_numbering/
```

([источник дерева](00_documentation_strategy.md#74-03_platform))

##### Стало — унифицированная структура

```text
docs-new/03_platform/
├── 00_platform_documentation_template.md
├── 00_foundation/            [единый комплект]
├── 01_tenant_and_security/   [единый комплект]
├── 02_configuration/         [единый комплект]
├── 03_object_runtime/        [единый комплект]
├── 04_workflow/              [единый комплект]
├── 05_rules/                 [единый комплект]
├── 06_value_sets/            [единый комплект]
├── 07_settings/              [единый комплект]
├── 08_numbering/             [единый комплект]
├── 09_audit_history/         [единый комплект]
├── 10_integration_events/    [единый комплект]
├── 11_reporting_output/      [единый комплект]
└── 12_frontend_platform/     [единый комплект]

[единый комплект]/
├── 00_platform_overview.md
├── 01_scope.md
├── 02_architecture.md
├── 03_contracts.md
├── 04_runtime.md
├── 05_security_and_audit.md
├── 06_user_experience.md
├── 07_quality.md
├── 08_operations.md
└── 90_traceability.md
```

([источник целевой структуры](../03_platform/00_platform_documentation_template.md#6-состав-документов-платформенной-области))

##### Сопоставление «было → стало»

| Область | Было | Стало | Что будет внутри | Почему |
| --- | --- | --- | --- | --- |
| Основа платформы | `00_platform_core.md` | `00_foundation/` | Общие правила платформы, execution context и применение общих контрактов. | Одного файла недостаточно для механизмов, от которых зависят остальные capabilities. |
| Tenant и безопасность | `01_tenant_and_security/` | `01_tenant_and_security/` | Tenant/site context, identity, роли, права и authorization decisions. | Граница сохраняется; меняется не область, а единый состав документов внутри неё. |
| Configuration | `02_configuration_platform/` | `02_configuration/` | Определения конфигурационных артефактов, версии, публикация и хранение. | Workflow, Rules, Reporting и Output отделяются от жизненного цикла конфигурации. |
| Object Runtime | `10_object_runtime/` | `03_object_runtime/` | Единое исполнение операций над объектами и связанные runtime services. | Это базовый runtime-механизм; он должен читаться до зависимых capabilities. |
| Workflow | `03_workflow_engine/` + workflow-файлы Configuration | `04_workflow/` | Определения и исполнение состояний, переходов и истории workflow. | Модель и исполнение одной capability собираются вместе. |
| Rules | `04_rule_engine/` + rule-файлы Configuration | `05_rules/` | Контракт вычисления правил и его связь с конфигурацией. | Конфигурация и runtime правил больше не разорваны по двум папкам. |
| Value Sets | `05_value_set_data/` | `06_value_sets/` | ValueSet, SystemEnum, элементы значений, overrides и runtime-чтение. | Граница сохраняется, название приводится к платформенной терминологии. |
| Settings | Отдельной области нет | `07_settings/` | Каталог настроек, области действия, значения, переопределения и effective value. | У Settings есть собственные модель, контракты и runtime. |
| Numbering | `11_numbering/` | `08_numbering/` | Правила нумерации, счётчики, конкурентность и интеграция с Object Runtime. | Граница сохраняется, меняется место в порядке чтения; после миграции переходная копия `11_numbering/` удалена. |
| Audit и History | `08_audit_history/` | `09_audit_history/` | Audit records, workflow/object history, actor/context и трассировка. | Граница сохраняется и получает единый комплект документов. |
| Integration Events | `07_event_foundation/` | `10_integration_events/` | Конверт, публикация, хранение и outbox интеграционных событий. | Название отражает подтверждённую событийную границу. |
| Reporting и Output | Файлы внутри `02_configuration_platform/` | `11_reporting_output/` | Report/output definitions и runtime формирования результата. | У capability собственный runtime; делить Reporting и Output пока рано. |
| Frontend Platform | Отдельной области нет | `12_frontend_platform/` | Frontend contracts, runtime packages, shell и platform applications. | Общие frontend-механизмы используются несколькими capabilities и модулями. |
| Common Application Contracts | `09_common_application_contracts/` | `00_foundation/` + владельцы областей + backlog | Foundation ведёт общие application-формы; area-specific контракты остаются у владельцев. | Контракты не становятся самостоятельным общим слоем логики и не создают верхний раздел `05_contracts`. |
| Полная Integration Foundation | `06_integration/` | `10_backlog/platform_capabilities/` | Внешние системы, коннекторы, интеграционные потоки и external id mapping. | Полная capability остаётся планируемой до подтверждения реализацией и контрактами. |
| Time Series | В структуре `00` нет | `10_backlog/platform_capabilities/` | Временные ряды, хранение, запросы, агрегации и retention. | Материалы сохраняются, но не выдаются за реализованную platform capability. |

**Итог:** для миграции используется структура из `docs-new/03_platform/00_platform_documentation_template.md`. Структура из `00_documentation_strategy.md` остаётся исходной рамкой и не изменяется этим документом. ([структура из `00`](00_documentation_strategy.md#74-03_platform); [целевая структура](../03_platform/00_platform_documentation_template.md#6-состав-документов-платформенной-области))

<details>
<summary>Логика вывода</summary>

В структуре из `00_documentation_strategy.md` состав файлов зависит от области: часть тем разделена по смыслу, часть смешивает configuration и runtime. В платформенном шаблоне каждая capability получает одинаковые точки входа: обзор, граница, архитектура, контракты, исполнение, безопасность, пользовательский слой, качество, эксплуатация и трассировка. Это упрощает ревью и не требует пустых документов: неприменимый файл не создаётся, а номер остаётся закреплённым за типом документа. ([структура из `00`](00_documentation_strategy.md#74-03_platform); [правила шаблона](../03_platform/00_platform_documentation_template.md#6-состав-документов-платформенной-области))

</details>

`00_platform_overview.md`:

```markdown
# Обзор платформенной области — <Название>

## 1. Назначение области
## 2. Место в платформе
## 3. Основные возможности
## 4. Ключевые решения
## 5. Зависимости
## 6. Статус реализации
## 7. Состав документов
## История изменений
```

`01_scope.md`:

```markdown
# Граница платформенной области — <Название>

## 1. Назначение документа
## 2. Что входит
## 3. Что не входит
## 4. Граница с соседними областями и модулями
## 5. Соответствие требованиям
## 6. Ограничения версии
## История изменений
```

`02_architecture.md`:

```markdown
# Архитектура — <Название>

## 1. Назначение документа
## 2. Граница и компоненты
## 3. Модель и инварианты
## 4. Данные и хранение
## 5. Зависимости и точки расширения
## 6. Технические ограничения
## История изменений
```

`03_contracts.md`:

```markdown
# Контракты — <Название>

## 1. Назначение документа
## 2. Публичные интерфейсы
## 3. Ошибки
## 4. События и результаты
## 5. Конфигурация
## 6. Совместимость
## 7. Интеграции и потребители
## История изменений
```

`04_runtime.md`:

```markdown
# Исполнение — <Название>

## 1. Назначение документа
## 2. Основные сценарии
## 3. Операции и алгоритмы
## 4. Правила
## 5. Жизненные циклы
## 6. Согласованность
## 7. Сбои и восстановление
## История изменений
```

`05_security_and_audit.md`:

```markdown
# Безопасность и аудит — <Название>

## 1. Назначение документа
## 2. Ресурсы и права
## 3. Принятие решения и управление
## 4. Контроли и риски
## 5. Аудит
## 6. Покрытие и пробелы
## История изменений
```

`06_user_experience.md`:

```markdown
# Пользовательский слой — <Название>

## 1. Назначение документа
## 2. Frontend-контракты
## 3. Пользовательские поверхности
## 4. Навигация
## 5. Локализация
## 6. Ограничения
## История изменений
```

`07_quality.md`:

```markdown
# Качество — <Название>

## 1. Назначение документа
## 2. Надёжность
## 3. Наблюдаемость
## 4. Производительность
## 5. Проверки и результаты
## 6. Неподтверждённые свойства
## История изменений
```

`08_operations.md`:

```markdown
# Эксплуатация — <Название>

## 1. Назначение документа
## 2. Запуск и готовность
## 3. Миграции
## 4. Диагностика
## 5. Восстановление
## 6. Откат и эскалация
## История изменений
```

`90_traceability.md`:

```markdown
# Трассировка — <Название>

## 1. Назначение документа
## 2. Источники и требования
## 3. Принятые решения
## 4. Расхождения
## 5. Открытые вопросы
## 6. Маршрут в целевые документы
## История изменений
```

#### Правила наполнения

1. Одноимённые документы всех platform capabilities используют одинаковые разделы `##` из `00_platform_documentation_template.md`.
2. Предметные детали добавляются только через `###` и `####` внутри закреплённого раздела.
3. Неприменимый документ не создаётся; его номер не получает другое назначение.
4. Неподтвержденная архитектура не добавляется в target-документы. Она фиксируется как открытый вопрос в `90_traceability.md` или уходит в `10_backlog/`.
5. Каждый технический тезис должен ссылаться на код, контракт, тест, миграцию, исходный документ или явно оформленную логику вывода.

Стандартная таблица трассировки:

| ID | Тезис | Раздел документа | Код | Тест | Контракт | Commit | Статус |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |  | confirmed / partial / not confirmed / obsolete |

Порядок миграции:

1. Сохранить исходные материалы в `offers/` или `requirements/`, если они ещё не разобраны в target-документы.
2. Для каждой платформенной области заполнить только документы из реестра `00_platform_documentation_template.md`.
3. Не создавать пустые target-файлы: `00_platform_overview.md` и `01_scope.md` обязательны, остальные появляются при наличии предметного содержания.
4. Контракты вести в документах владельцев; старые материалы `docs/05 contracts` использовать только как источники сверки.
5. Runtime-поведение фиксировать у владельца platform area в `04_runtime.md`;
   сквозные вопросы вести в `90_traceability.md` владельцев и backlog.
6. Runbook-материалы вести у владельцев; `07_operations/` создавать только для сквозных platform-wide runbooks.
7. Детальные тестовые сценарии вести у владельцев; `08_testing/` создавать только для сквозной QA/test strategy.
8. ADR drafts не копировать в текущий `docs-new`; использовать как источники сверки.
9. Планируемые или неподтвержденные capabilities вынести в `10_backlog/`.
10. Legacy/research/imported material вынести в `12_appendices/`.

## 2. Легенда решений

| Решение | Значение |
| --- | --- |
| Однозначно | Файл целиком ложится в один целевой документ или раздел. |
| Разделить | Один исходник содержит несколько разных сущностей и должен быть разнесен по нескольким документам. |
| Спорно | Место или смысл требуют решения владельца методологии/архитектуры. |
| Не переносить | Файл учитывается как источник, но не становится нормативным документом. |
| Архив | Исторический материал переносится только как legacy/reference. |
| Контракт | Файл описывает контракт или машинный пример; в `docs-new` он становится нормативным только через документ владельца. |
| Backlog | Материал описывает будущие работы, план или разрыв и должен храниться в `10_backlog/`. |

## 3. Сводка по разделам `docs`

| Раздел `docs` | Количество файлов | Основной целевой раздел `docs-new` | Методологическое решение |
| --- | ---: | --- | --- |
| `docs/01 sources` | 27 | `01_concept`, `02_architecture`, `03_platform`, `04_domain_modules`, `10_backlog` | Основной массив для структурирования. Переносить последовательно, по одному смысловому блоку. |
| `docs/02 requrements` | 30 | `10_backlog/requirements`, документы владельцев `03_platform` и `04_domain_modules` | Это требования и UX-черновики. Нельзя превращать их в утвержденный design без анализа; сквозной `08_testing` не создаётся без общей QA-стратегии. |
| `docs/03 conf_artefacts` | 18 | `03_platform/02_configuration_platform` | Описания конфигурационных артефактов. Схемный смысл фиксируется в документах Configuration и `artifact_types`. |
| `docs/04 runtime` | 3 | `03_platform/02_configuration`, `03_platform/03_object_runtime`, `03_platform/07_settings`, `03_platform/12_frontend_platform`, `10_backlog` | Runtime-материалы являются исходниками для владельцев platform areas, а не для отдельного целевого раздела. |
| `docs/05 contracts` | 16 | Не переносить как отдельный раздел `docs-new` | Старые контракты и примеры использовать как источники сверки документов владельцев. |
| `docs/10 modules_req` | 3 | `04_domain_modules` | Предметные материалы прикладных модулей. |
| `docs/10_backlog` | 10 | `10_backlog` | Бэклог, roadmap и оценки. Не смешивать с утвержденной архитектурой. |
| `docs/adr` | 19 | Не переносить автоматически | ADR drafts использовать как источники сверки; отдельный `09_decisions` создаётся только после решения о конкретном утверждённом ADR. |
| `docs/architecture` | 1 | `02_architecture` или `04_domain_modules` | Правила интеграции модулей; возможно частично войдет в шаблон модуля. |
| `docs/archive` | 6 | `12_appendices/legacy_docs` | Только архив, не нормативная база. |
| `docs/current` | 5 | `12_appendices/research` и выборочно `03_platform`/`10_backlog` | Текущий срез реализации. Использовать как источник сверки, не как целевую архитектуру. |
| `docs/other_tech` | 4 | `12_appendices/research`, выборочно `03_platform/10_object_runtime` | Исследовательские материалы по Object Runtime и MES. |
| `docs/reconciliation` | 1 | `12_appendices/imported_materials` или `00_governance` | Служебный раздел сверки. |
| Корневые файлы `docs` | 2 | `00_governance`, `10_backlog` | Стратегию обновлять через `00`, баги — через backlog/issue. |

### Правило промежуточной папки `offers/`

Папка `offers/` создаётся только на уровне родительского целевого раздела, если один исходный файл ещё предстоит разделить по нескольким дочерним документам этого раздела. Это промежуточное хранилище для одной source-копии до смысловой раскладки.

Если исходный файл однозначно относится к одному конечному узлу, source-копия размещается сразу в этом узле, а `offers/` не создаётся. После завершения разделения исходника его копия также должна быть удалена из `offers/`; в промежуточной папке не должны оставаться дубли конечных документов.

| Ситуация | Размещение | Пример |
| --- | --- | --- |
| Один источник распределяется по нескольким дочерним документам | `целевой-раздел/offers/` | Configuration Platform Data Model → data, conditions, output, versioning и import/export |
| Один источник имеет один конечный документ | `целевой-документ/` | MVP Foundation Plan → `10_backlog/roadmap/`; ADR drafts остаются вне `docs-new`, пока не принято отдельное решение о целевом ADR |

<details>
<summary><strong>Аудит текущих папок <code>offers/</code></strong></summary>

| Родительский раздел | Почему `offers/` оставлена | Статус |
| --- | --- | --- |
| `03_platform/01_tenant_and_security/` | Один исходный документ будет разделён на overview, tenant model, IAM model и authorization contracts. | Соответствует правилу |
| `03_platform/02_configuration_platform/` | Исходники Configuration Platform и конфигурационные артефакты распределяются между data, workflow, rules, UI, reporting, output, contracts и runtime. | Соответствует правилу |
| `03_platform/05_value_set_data/` | Один исходник распределяется между overview, моделью ValueSet, элементами и tenant override. | Соответствует правилу |
| `10_backlog/platform_capabilities/integration_capability_backlog.md` | Future-content исходника `11_1_integration_capability_architecture.md` перенесён в backlog; будущий нормативный пакет появится только после подтверждения capability. | Соответствует правилу; переходный `06_integration/offers` закрывается |
| `03_platform/10_integration_events/` | Исходник `11_integration_event_platform.md` разделён на overview, event envelope, outbox/runtime, quality, operations и traceability. | Соответствует правилу; переходный `07_event_foundation` закрыт |
| `03_platform/09_common_application_contracts/` | Переходная source-папка закрыта: подтверждённое содержание разнесено между Foundation, Object Runtime, владельцами областей и backlog. | Соответствует актуальному решению |
| `03_platform/10_object_runtime/` | Исследовательский исходник ещё нужно разделить на нормативные Object Runtime-паттерны и сравнительный материал. | Соответствует правилу |
| `04_domain_modules/` и его подразделы | Исходники module map, integration rules и предметных модулей требуют раскладки по нескольким дочерним документам. | Соответствует правилу |
| Старый `05_contracts/` | Раздел исключён из целевой структуры; contract-source материалы остаются в `docs/05 contracts` и сверяются с документами владельцев. | Соответствует актуальному решению |
| `10_backlog/roadmap/preparation/platform_core_documentation_backlog.md` | Переходный `06_runtime` закрывается: подтверждённые сведения уже у владельцев `03_platform`, reference-примеры не переносятся как нормативные документы. | Соответствует правилу; `06_runtime` удаляется |

Однозначные материалы из этой проверки вынесены из `offers/`: MVP Foundation Plan, reference patterns, Reporting Configuration и Output Configuration. ADR drafts не переносятся в `docs-new` как source-копии. Дубли исходной модели Configuration Platform объединены в одну source-копию `offers/` и будут удалены после окончательной раскладки.

</details>

## 4. Фактическое сопоставление

Эта таблица показывает не предложения, а уже фактически разложенные копии исходных материалов в `docs-new/`. Она нужна для быстрой проверки без открытия каждой карточки.

| # | Исходный файл | Фактическое место в `docs-new` | Размещение |
| ---: | --- | --- | --- |
| 1 | `docs/01 sources/15_mvp_foundation_plan.md` | `10_backlog/roadmap/preparation/09_two_week_platform_architecture_outcome_requirements.md`; `10_backlog/roadmap/preparation/platform_core_documentation_backlog.md` | Содержание маршрутизировано в актуальные плановые документы; source-копия в `docs-new` удалена |
| 2 | `docs/01 sources/01_concept.md` | `docs-new/01_concept/01_product_concept.md/01_concept.md (01 sources).md` | Source-копия в целевом узле |
| 3 | `docs/01 sources/02_system_scope.md` | `docs-new/01_concept/02_system_scope.md/02_system_scope.md (01 sources).md` | Source-копия в целевом узле |
| 4 | `docs/01 sources/05_architecture.md` | `docs-new/02_architecture/01_architecture_overview.md/05_architecture.md (01 sources).md` | Source-копия в целевом узле |
| 5 | `docs/01 sources/03_platform_core.md` | `01_concept/02_system_scope.md`, `02_architecture/01_architecture_overview.md`, `03_platform/*/`, `03_platform/00_foundation/90_traceability.md` | Staging-копия закрыта после разнесения по целевым владельцам |
| 6 | `docs/01 sources/07_tenant_and_security_platform.md` | Не копируется в Tenant/Security. Содержание разбирается через backlog `PCDOC-12`. | Source остаётся в старом `docs` |
| 7 | `docs/02 requrements/!role_permition_refactor.md` | Не копируется в Tenant/Security. Содержание разбирается через backlog `PCDOC-13`. | Requirement остаётся в старом `docs` |
| 8 | `docs/02 requrements/01_roles.md` | Не копируется в Tenant/Security. Содержание разбирается через backlog `PCDOC-13`. | Requirement остаётся в старом `docs` |
| 9 | `docs/02 requrements/01_roles_localization.md` | Не копируется в Tenant/Security. Содержание разбирается через backlog `PCDOC-13`. | Requirement остаётся в старом `docs` |
| 10 | `docs/02 requrements/01_roles_rules.md` | Не копируется в Tenant/Security. Содержание разбирается через backlog `PCDOC-13`. | Requirement остаётся в старом `docs` |
| 11 | `docs/02 requrements/01_tenants_rules.md` | Не копируется в Tenant/Security. Содержание разбирается через backlog `PCDOC-13`. | Requirement остаётся в старом `docs` |
| 12 | `docs/02 requrements/01_users_rules.md` | Не копируется в Tenant/Security. Содержание разбирается через backlog `PCDOC-13`. | Requirement остаётся в старом `docs` |
| 13 | `docs/01 sources/18_system_module_settings_decision.md` | `docs-new/03_platform/02_configuration_platform/03_data_configuration.md/18_system_module_settings_decision.md (01 sources).md` | Source-копия в целевом узле |
| 14 | `docs/01 sources/06.3.2_workflow_action_model_glossary_platform_contract.md` | `docs-new/03_platform/02_configuration_platform/04_workflow_configuration/06.3.2_workflow_action_model_glossary_platform_contract.md (01 sources).md` | Целевой файл |
| 15 | `docs/02 requrements/03_06_page_workflow_editor_ui_v2.md` | `docs-new/03_platform/02_configuration_platform/04_workflow_configuration/requirements/03_06_page_workflow_editor_ui_v2.md (02 requrements).md` | Requirement-копия |
| 16 | `docs/02 requrements/03_08_page_action_model_and_action_ux_v2.md` | `docs-new/03_platform/02_configuration_platform/04_workflow_configuration/requirements/03_08_page_action_model_and_action_ux_v2.md (02 requrements).md` | Requirement-копия |
| 17 | `docs/02 requrements/03_09_page_rule_editor_ui.md` | `docs-new/03_platform/02_configuration_platform/05_rule_configuration/requirements/03_09_page_rule_editor_ui.md (02 requrements).md` | Requirement-копия |
| 18 | `docs/02 requrements/02_ui.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/02_ui.md (02 requrements).md` | Requirement-копия |
| 19 | `docs/02 requrements/03_03_page_configuration_explorer_ui.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_03_page_configuration_explorer_ui.md (02 requrements).md` | Requirement-копия |
| 20 | `docs/02 requrements/03_04_page_data_inspector_ui.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_04_page_data_inspector_ui.md (02 requrements).md` | Requirement-копия |
| 21 | `docs/02 requrements/03_05_property_editor_ui.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_05_property_editor_ui.md (02 requrements).md` | Requirement-копия |
| 22 | `docs/02 requrements/03_07_page_view_editor_ui.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_07_page_view_editor_ui.md (02 requrements).md` | Requirement-копия |
| 23 | `docs/02 requrements/03_10_page_navigation_editor_ui.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_10_page_navigation_editor_ui.md (02 requrements).md` | Requirement-копия |
| 24 | `docs/02 requrements/03_11_page_workplace_editor_ui.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_11_page_workplace_editor_ui.md (02 requrements).md` | Requirement-копия |
| 25 | `docs/02 requrements/03_13_page_context_version_management_ui_v4.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_13_page_context_version_management_ui_v4.md (02 requrements).md` | Requirement-копия |
| 26 | `docs/02 requrements/03_config_app_ui_req_02_v4.md` | `docs-new/03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_config_app_ui_req_02_v4.md (02 requrements).md` | Requirement-копия |
| 27 | `docs/10_backlog/report_birt_pipeline_backlog.md` | `docs-new/03_platform/02_configuration_platform/08_reporting_configuration.md/report_birt_pipeline_backlog.md (10_backlog).md` | Source-копия в целевом узле |
| 28 | `docs/01 sources/06.3.0_configuration_platform_data_model.md` | `docs-new/03_platform/02_configuration_platform/offers/06.3.0_configuration_platform_data_model.md (01 sources).md` | Offer-копия |
| 29 | `docs/01 sources/06.3.0_configuration_platform_infrastructure_persistence_requirements.md` | `docs-new/03_platform/02_configuration_platform/offers/06.3.0_configuration_platform_infrastructure_persistence_requirements.md (01 sources).md` | Offer-копия |
| 30 | `docs/01 sources/06.3.1_configuration_platform_data.md` | `docs-new/03_platform/02_configuration_platform/offers/06.3.1_configuration_platform_data.md (01 sources).md` | Offer-копия |
| 31 | `docs/01 sources/06.3.4_configuration_platform_ui.md` | `docs-new/03_platform/02_configuration_platform/offers/06.3.4_configuration_platform_ui.md (01 sources).md` | Offer-копия |
| 32 | `docs/01 sources/06.3.5_configuration_platform_rep.md` | `docs-new/03_platform/02_configuration_platform/08_reporting_configuration.md/06.3.5_configuration_platform_rep.md (01 sources).md` | Source-копия в целевом узле |
| 33 | `docs/01 sources/06.3.6_configuration_platform_output.md` | `docs-new/03_platform/02_configuration_platform/09_output_configuration.md/06.3.6_configuration_platform_output.md (01 sources).md` | Source-копия в целевом узле |
| 34 | `docs/01 sources/06.3.7_configuration_platform_report_output_birt.md` | `docs-new/03_platform/02_configuration_platform/offers/06.3.7_configuration_platform_report_output_birt.md (01 sources).md` | Offer-копия |
| 35 | `docs/03 conf_artefacts/Action.md` | `docs-new/03_platform/02_configuration_platform/offers/Action.md (03 conf_artefacts).md` | Offer-копия |
| 36 | `docs/03 conf_artefacts/Artefact_classification.md` | `docs-new/03_platform/02_configuration_platform/offers/Artefact_classification.md (03 conf_artefacts).md` | Offer-копия |
| 37 | `docs/03 conf_artefacts/Configurator_ui_structure.md` | `docs-new/03_platform/02_configuration_platform/offers/Configurator_ui_structure.md (03 conf_artefacts).md` | Offer-копия |
| 38 | `docs/03 conf_artefacts/EnumPresentation.md` | `docs-new/03_platform/02_configuration_platform/offers/EnumPresentation.md (03 conf_artefacts).md` | Offer-копия |
| 39 | `docs/03 conf_artefacts/GridUi.md` | `docs-new/03_platform/02_configuration_platform/offers/GridUi.md (03 conf_artefacts).md` | Offer-копия |
| 40 | `docs/03 conf_artefacts/Lookup.md` | `docs-new/03_platform/02_configuration_platform/offers/Lookup.md (03 conf_artefacts).md` | Offer-копия |
| 41 | `docs/03 conf_artefacts/Navigation.md` | `docs-new/03_platform/02_configuration_platform/offers/Navigation.md (03 conf_artefacts).md` | Offer-копия |
| 42 | `docs/03 conf_artefacts/NumberingRule.md` | `docs-new/03_platform/02_configuration_platform/offers/NumberingRule.md (03 conf_artefacts).md` | Offer-копия |
| 43 | `docs/03 conf_artefacts/ObjectNavigationPolicy.md` | `docs-new/03_platform/02_configuration_platform/offers/ObjectNavigationPolicy.md (03 conf_artefacts).md` | Offer-копия |
| 44 | `docs/03 conf_artefacts/ObjectType.md` | `docs-new/03_platform/02_configuration_platform/offers/ObjectType.md (03 conf_artefacts).md` | Offer-копия |
| 45 | `docs/03 conf_artefacts/Output.md` | `docs-new/03_platform/02_configuration_platform/offers/Output.md (03 conf_artefacts).md` | Offer-копия |
| 46 | `docs/03 conf_artefacts/Report.md` | `docs-new/03_platform/02_configuration_platform/offers/Report.md (03 conf_artefacts).md` | Offer-копия |
| 47 | `docs/03 conf_artefacts/Rules.md` | `docs-new/03_platform/02_configuration_platform/offers/Rules.md (03 conf_artefacts).md` | Offer-копия |
| 48 | `docs/03 conf_artefacts/SystemEnum.md` | `docs-new/03_platform/02_configuration_platform/offers/SystemEnum.md (03 conf_artefacts).md` | Offer-копия |
| 49 | `docs/03 conf_artefacts/ValueSet.md` | `docs-new/03_platform/02_configuration_platform/offers/ValueSet.md (03 conf_artefacts).md` | Offer-копия |
| 50 | `docs/03 conf_artefacts/View.md` | `docs-new/03_platform/02_configuration_platform/offers/View.md (03 conf_artefacts).md` | Offer-копия |
| 51 | `docs/03 conf_artefacts/Workflow.md` | `docs-new/03_platform/02_configuration_platform/offers/Workflow.md (03 conf_artefacts).md` | Offer-копия |
| 52 | `docs/03 conf_artefacts/artifact_property_regrouping.md` | `docs-new/03_platform/02_configuration_platform/offers/artifact_property_regrouping.md (03 conf_artefacts).md` | Offer-копия |
| 53 | `docs/02 requrements/01_admin_list_newfilters_config.md` | `docs-new/03_platform/02_configuration_platform/requirements/01_admin_list_newfilters_config.md (02 requrements).md` | Requirement-копия |
| 54 | `docs/02 requrements/03_02_shell_configurator_vs_admin.md` | `docs-new/03_platform/02_configuration_platform/requirements/03_02_shell_configurator_vs_admin.md (02 requrements).md` | Requirement-копия |
| 55 | `docs/02 requrements/04_artifact_editor_exploer_diff_promts.md` | `docs-new/03_platform/02_configuration_platform/requirements/04_artifact_editor_exploer_diff_promts.md (02 requrements).md` | Requirement-копия |
| 56 | `docs/02 requrements/04_reg_and_scenario_config_product_definition.md` | `docs-new/03_platform/02_configuration_platform/requirements/04_reg_and_scenario_config_product_definition.md (02 requrements).md` | Requirement-копия |
| 57 | `docs/02 requrements/05_configuration_studio_end_to_end_scenarios_corporate_tenant_lifecycle.md` | `docs-new/03_platform/02_configuration_platform/requirements/05_configuration_studio_end_to_end_scenarios_corporate_tenant_lifecycle.md (02 requrements).md` | Requirement-копия |
| 58 | `docs/02 requrements/06_wf_runtime_ui_requirements_and_flows_revised_list_ui.md` | Канонические серверные сведения: `docs-new/03_platform/04_workflow/03_contracts.md`, `04_runtime.md`, `01_scope.md`, `90_traceability.md`; UI renderer, layout и пользовательские flow остаются исходными требованиями для будущего владельца Frontend Platform | Переходная копия удалена; исходник сохранён в `docs` |
| 59 | `docs/01 sources/10_valuesets.md` | `docs-new/03_platform/05_value_set_data/offers/10_valuesets.md (01 sources).md` | Offer-копия |
| 60 | `docs/02 requrements/03_12_1_value_set_data_editor_design.md` | `docs-new/03_platform/05_value_set_data/requirements/03_12_1_value_set_data_editor_design.md (02 requrements).md` | Requirement-копия |
| 61 | `docs/02 requrements/03_12_page_value_set_editor_ui.md` | `docs-new/03_platform/05_value_set_data/requirements/03_12_page_value_set_editor_ui.md (02 requrements).md` | Requirement-копия |
| 62 | `docs/01 sources/11_1_integration_capability_architecture.md` | `docs-new/10_backlog/platform_capabilities/integration_capability_backlog.md` | Future backlog; offer-копия закрыта после переноса |
| 63 | `docs/01 sources/11_integration_event_platform.md` | `docs-new/03_platform/10_integration_events/`, `docs-new/03_platform/10_integration_events/90_traceability.md` | Offer-копия закрыта после миграции |
| 64 | `docs/01 sources/12_audit_history_platform.md` | `docs-new/03_platform/09_audit_history/00_platform_overview.md`, `01_scope.md`, `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `05_security_and_audit.md`, `07_quality.md`, `08_operations.md`, `90_traceability.md` | Переходная offer-копия удалена; исходник сохранён в `docs` |
| 65 | `docs/01 sources/13_platform_api_and_contracts.md` | `docs-new/10_backlog/contract_governance/platform_api_contracts_backlog.md`; `03_platform/00_foundation`; `03_platform/03_object_runtime` | Future backlog; offer-копия закрыта после переноса |
| 66 | `docs/01 sources/14.1_platform_runtime_reference_patterns.md` | Не переносить в `docs-new` как нормативный документ; при необходимости использовать старый источник при нормализации шаблона модулей | Reference-примеры, source-копия удаляется |
| 67 | `docs/other_tech/dmp-module-object-runtime.md` | `docs-new/03_platform/10_object_runtime/offers/dmp-module-object-runtime.md (other_tech).md` | Offer-копия |
| 68 | `docs/02 requrements/03_14_objectlist_bounded_runtime_layout_model.md` | `docs-new/03_platform/10_object_runtime/requirements/03_14_objectlist_bounded_runtime_layout_model.md (02 requrements).md` | Requirement-копия |
| 69 | `docs/02 requrements/03_15_runtime_list_query_filters_requirements.md` | `docs-new/03_platform/10_object_runtime/requirements/03_15_runtime_list_query_filters_requirements.md (02 requrements).md` | Requirement-копия |
| 70 | `docs/01 sources/16_module_architecture_standard.md` | `docs-new/04_domain_modules/00_module_documentation_template.md` | Содержание сверено с шаблоном прикладного модуля; source-копия в `docs-new` удалена, исходник остаётся в `docs` |
| 71 | `docs/01 sources/04_module_map.md` | `docs-new/04_domain_modules/01_module_index.md` | Содержание сверено с индексом прикладных модулей; source-копия в `docs-new` удалена, исходник остаётся в `docs` |
| 72 | `docs/10 modules_req/01 ProductDefinition/nomenclature_architecture_decision_v2.md` | `docs-new/04_domain_modules/02_product_process_definition/90_traceability_pr02.md`, `backlog.md` | Offer-копия удалена после сверки; текущая граница и маршрут будущих частей зафиксированы в документах модуля, исходник в `docs` сохранён |
| 73 | `docs/10 modules_req/08 PlanningScheduling/planning_scheduling_architecture_revised.md` | `docs-new/04_domain_modules/08_planning_scheduling/offers/planning_scheduling_architecture_revised.md (10 modules_req).md` | Offer-копия |
| 74 | `docs/10 modules_req/08 PlanningScheduling/planning_scheduling_scenarios_outline.md` | `docs-new/04_domain_modules/08_planning_scheduling/offers/planning_scheduling_scenarios_outline.md (10 modules_req).md` | Offer-копия |
| 75 | `docs/01 sources/17_product_definition_reference_map.md` | Не переносится как нормативный документ; исходник остаётся в `docs/01 sources`, а предметные решения ведутся в документах `02_product_process_definition` | Demo/reference-инструкция; переходная offer-копия удалена |
| 76 | `docs/architecture/module-integration-rules.md` | `docs-new/04_domain_modules/00_module_documentation_template.md` и `docs-new/02_architecture/07_integration_architecture.md` | Правила межмодульного взаимодействия перенесены по владельцам; переходная offer-копия удалена |
| 77 | `docs/05 contracts/report_service/*` | Не переносить в `docs-new` как отдельный раздел | Старые Report Service OpenAPI/JSON Schema/XSD/XML/examples остаются источниками сверки; нормативный смысл описан у Reporting/Output |
| 78 | `docs/05 contracts/report_service_api_contract.md` | Не переносить в `docs-new` как отдельный документ | Подтверждённые сведения покрыты `03_platform/11_reporting_output/03_contracts.md`, `04_runtime.md`, `07_quality.md`, `08_operations.md` |
| 79 | `docs/05 contracts/rule_and_condition_model_platform_contract.md` | Не переносить в `docs-new` как отдельный документ | Подтверждённые границы покрыты Rules и Configuration; неподтверждённые возможности не объявляются MVP |
| 80 | `docs/05 contracts/tenant_security/*` | Не переносить в `docs-new` как отдельный раздел | Security catalog и inventory используются как источники сверки `03_platform/01_tenant_and_security` |
| 93 | `docs/01 sources/14_platform_runtime_conventions.md` | Не переносить в `docs-new` как отдельный документ; подтверждённые сведения уже у Foundation, Tenant/Security, Object Runtime и владельцев `03_platform/*/04_runtime.md` | Source-копия удаляется |
| 94 | `docs/04 runtime/configuration_scope_version_pipeline.md` | `03_platform/02_configuration/04_runtime.md`, `07_quality.md`, `08_operations.md`, `90_traceability.md` | Переходная копия удаляется |
| 95 | `docs/04 runtime/runtime_data_flow.md` | `03_platform/03_object_runtime`, `03_platform/02_configuration`, `03_platform/12_frontend_platform` | Переходная копия удаляется |
| 96 | `docs/04 runtime/runtime_settings_authoring_guide.md` | `03_platform/07_settings`, `03_platform/07_settings/90_traceability.md` | Переходная копия удаляется |
| 97 | `docs/02 requrements/03_01_runtime_engine_for_admin_config.md` | `03_platform/12_frontend_platform/90_traceability.md` | Requirement-копия удаляется |
| 98 | `docs/02 requrements/03_12_admin_runtime_refactor_plan.md` | `03_platform/12_frontend_platform/90_traceability.md` | Requirement-копия удаляется |
| 99-117 | `docs/adr/drafts/*.md` | Не копируются в `docs-new` | Старые ADR drafts остаются в `docs/adr/drafts`; текущий комплект `docs-new` не создаёт целевые ADR. |
| 118 | `docs/00_documentation_strategy.md` | `docs-new/00_governance/00_documentation_strategy.md` | Целевой документ |
| 119 | `docs/01 sources/06_configuration_platform.md` | `docs-new/03_platform/02_configuration_platform/01_overview.md/06_configuration_platform.md (01 sources).md` | Source-копия в целевом узле |

Если документ неоднозначно относится к нескольким целевым разделам, для него используется `offers/` в родительском целевом разделе: это зона разборки, а не финальный нормативный документ. Если конечный узел определён однозначно, `offers/` не используется.

## 5. Предложения по сопоставлению

Свернутые блоки ниже сохраняют методологические предложения: куда файл логически должен попасть, почему выбран этот раздел и какие вопросы остаются перед финальной структуризацией.

### 5.1 Предложения по сопоставлению: корень `docs`

<details>
<summary><strong>Предложения по сопоставлению: корень `docs`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/!bugs_fix.md` | `10_backlog/requirements` или issue tracker | Спорно | Список исправлений/замечаний без нормативной структуры. | Это не архитектурный документ, а рабочий список дефектов. В `docs-new` стоит переносить только устойчивые решения или ссылку на backlog. | Нужно ли хранить этот список в Git-документации или перенести в issue tracker? |
| `docs/00_documentation_strategy.md` | `00_governance/00_documentation_strategy.md` | Однозначно | Исходная стратегия docs-as-code, структура папок, типы документов, процесс review и проверки. | Это прямой предшественник целевого `00`. Нужно сравнить с актуальным `docs-new/00_governance/00_documentation_strategy.md` и перенести только отсутствующие правила. | Есть ли правила, которые уже устарели после свежего `00`? |

</details>

### 5.2 Предложения по сопоставлению: `docs/01 sources`

<details>
<summary><strong>Предложения по сопоставлению: `docs/01 sources`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/01 sources/00_release_plan_manifest.md` | `00_governance/requirements/00_index.md` или `10_backlog/roadmap/roadmap-platform-foundation.md` | Не переносить как нормативный документ | План приоритизации выпуска документов Platform Foundation. | Файл полезен как источник очередности, но не как итоговый документ. Содержание лучше использовать в реестре миграции и roadmap. | Оставляем только как источник или формируем отдельный roadmap-план? |
| `docs/01 sources/01_concept.md` | `01_concept/01_product_concept.md/01_concept.md (01 sources).md` | Однозначно | Верхнеуровневая продуктовая концепция DMP, цель, контекст и ожидаемый эффект. | Полностью соответствует разделу продуктовой концепции. Нужны шапка, ключевые понятия, связи и история. | Нет. |
| `docs/01 sources/02_system_scope.md` | `01_concept/02_system_scope.md/02_system_scope.md (01 sources).md` | Однозначно | Границы системы, входящие и не входящие функции, интеграции и владение данными. | Это scope-документ уровня системы. Нужно сохранить исходные границы и вынести вопросы по данным отдельно. | Уточнить границы `operational data`, `execution data` и `reference data`. |
| `docs/01 sources/03_platform_core.md` | `01_concept/02_system_scope.md`, `02_architecture/01_architecture_overview.md`, `03_platform/*/`, `03_platform/00_foundation/90_traceability.md` | Разнесено по владельцам | Состав и границы платформенного ядра, MVP и общие платформенные механизмы. | Это обзор Platform Core: верхние тезисы принадлежат концепции и архитектуре, detail capabilities — соответствующим platform areas, Foundation — только общим technical/application контрактам. | Staging-копия закрыта; дальнейшие изменения идут через документы-владельцы. |
| `docs/01 sources/04_module_map.md` | `04_domain_modules/01_module_index.md` | Разнесено по владельцу | Карта прикладных модулей и их назначение. | Прикладные модули отражены в индексе модулей; платформенные блоки ведутся в `03_platform`; source-копия в `docs-new` удалена. | Нет. |
| `docs/01 sources/05_architecture.md` | `02_architecture/01_architecture_overview.md/05_architecture.md (01 sources).md` | Однозначно | Архитектурный обзор: уровни, сервисы, данные, интеграции и развертывание. | Это общий architecture overview. Термины слоев и сервисов должны быть унифицированы через глоссарий. | Открыты границы `Reference Data Platform`/`Reference Data Service`, `Event Store`, `Hybrid Multi-Tenant`. |
| `docs/01 sources/06_configuration_platform.md` | `03_platform/02_configuration_platform/01_overview.md/06_configuration_platform.md (01 sources).md` | Однозначно | Обзор платформы конфигурирования, границы, виды конфигурации, уровни применения, публикация и интеграции. | Source-копия сохранена внутри целевого узла Configuration Platform. Дочерние документы должны ссылаться на неё, но не дублировать обзор. | Открыты вопросы по baseline-формам, уровням `plant/module/object type`, ролям и правам. |
| `docs/01 sources/06.3.0_configuration_platform_data_model.md` | `03_platform/02_configuration_platform/03_data_configuration.md`; `06_condition_model.md`; `09_output_configuration.md`; `10_versioning_and_publish.md`; `11_import_export.md` | Разделить через offer-копию | Логическая модель Configuration Platform: scope, version, entry/property, binding, localization, runtime resolution, validation before publish и import identity. | Единая source-копия хранится в `03_platform/02_configuration_platform/offers/`; содержательные блоки будут разнесены по целевым документам после анализа. | Нужно согласовать, какие части останутся в общей модели данных, а какие уйдут в lifecycle, conditions, output и import/export. |
| `docs/01 sources/06.3.0_configuration_platform_infrastructure_persistence_requirements.md` | `03_platform/02_configuration/02_architecture.md`; `03_platform/02_configuration/07_quality.md`; `03_platform/02_configuration/08_operations.md`; backlog | Разделить | Требования к хранению, индексации, производительности, кэшированию, конкурентности, импорту/экспорту и наблюдаемости. | Один файл содержит несколько типов решений: design хранения, quality/operations и будущие production-гарантии. Переносить одним документом нельзя. | Какие production-гарантии остаются future backlog, а какие подтверждены кодом Configuration? |
| `docs/01 sources/06.3.1_configuration_platform_data.md` | `03_platform/02_configuration_platform/03_data_configuration.md`; `03_platform/02_configuration_platform/10_versioning_and_publish.md` | Разделить | Настройка данных, поля, привязки источников значений, уровни применения, публикация и проверки. | Модель данных и правила публикации должны быть разведены. | Какой номер версионирования канонический в текущем `00`: `10_versioning_and_publish.md`. |
| `docs/01 sources/06.3.2_workflow_action_model_glossary_platform_contract.md` | `03_platform/02_configuration`, `03_platform/04_workflow` и владельцы stable codes | Разделить | Модель workflow, команды, переходы, бизнес-действия, runtime/API и glossary. | Файл смешивает конфигурацию жизненного цикла, исполнение Workflow Engine и контракты действий. | Нужно окончательно развести `CommandCode` и `ActionCode`. |
| `docs/01 sources/06.3.4_configuration_platform_ui.md` | `03_platform/02_configuration_platform/07_ui_configuration.md`; `03_platform/02_configuration_platform/10_versioning_and_publish.md` | Разделить | UI Configuration: представления, шаблоны, компоненты, runtime-применение, публикация и MVP. | UI-модель должна быть отдельным design-документом; публикация и версии — в общем документе публикации. | Как фиксировать стабильную идентичность вложенных UI-элементов? |
| `docs/01 sources/06.3.5_configuration_platform_rep.md` | `03_platform/02_configuration_platform/08_reporting_configuration.md` | Однозначно | Reporting configuration: dataset, KPI, отчеты, runtime, UI и output-связи. | Это design-документ reporting-конфигурации. | Нужно определить границу report dataset и доменных данных. |
| `docs/01 sources/06.3.6_configuration_platform_output.md` | `03_platform/02_configuration_platform/09_output_configuration.md` | Однозначно | Output/export configuration: выходные формы, каналы доставки, точки запуска и runtime-использование. | Это design-документ output-конфигурации. | Разграничить output, export и report delivery. |
| `docs/01 sources/06.3.7_configuration_platform_report_output_birt.md` | `03_platform/02_configuration`, `03_platform/11_reporting_output` | Разделить | Связка Report/Output и BIRT Report Service, контракты и pipeline. | Платформенный design и внешний сервисный контракт описаны у владельцев; старые машинные файлы остаются источниками сверки. | Какие части являются контрактом Java Report Service, а какие — платформенным design? |
| `docs/01 sources/07_tenant_and_security_platform.md` | `01_concept`; `02_architecture`; `03_platform/01_tenant_and_security`; backlog `PCDOC-12` | Разделить без копии в Tenant/Security | Tenant, IAM, роли, права, авторизация, изоляция данных, Event Model и security. | Исходник остаётся в старом `docs`; подтверждённое содержание переносится в документы-владельцы, целевые идеи — в backlog и открытые решения. | Общий вопрос: как определять tenant, предприятие, площадку и связанные tenant-формы. |
| `docs/01 sources/10_valuesets.md` | `03_platform/05_value_set_data/01_overview.md`; `02_value_set_model.md`; `03_value_items.md`; `04_tenant_override.md` | Разделить через offer-копию | ValueSetData, наборы значений, элементы, overrides и границы с НСИ. | Единая source-копия хранится в `03_platform/05_value_set_data/offers/`; содержательные блоки будут разнесены по целевым документам после анализа. | Проверить границу ValueSetData и `general_master_data`. |
| `docs/01 sources/11_1_integration_capability_architecture.md` | `10_backlog/platform_capabilities/integration_capability_backlog.md`; после утверждения capability — будущий пакет `03_platform/<integration_capability>/` | Future backlog | Архитектура интеграционной capability, connectors, runtime, external IDs, events и security. | Полная Integration Capability пока не имеет подтверждённого исполняемого владельца, поэтому будущие работы ведутся в backlog; `10_integration_events` остаётся текущим владельцем outbox/envelope. | После решения о владельце, номере каталога, коде и контрактах создать нормативный пакет и убрать backlog-статус. |
| `docs/01 sources/11_integration_event_platform.md` | `03_platform/10_integration_events/00_platform_overview.md`, `01_scope.md`, `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `07_quality.md`, `08_operations.md`, `90_traceability.md` | Разнесено по владельцу | Event Foundation, integration events, envelope, outbox/inbox и правила публикации. | Подтверждённая событийная основа оформлена как область `10_integration_events`; domain events остаются в Foundation/модулях, внешние flows — в будущей Integration Capability. | Переходная копия закрыта; открытые решения ведутся в `IE-DEC-*` и backlog. |
| `docs/01 sources/12_audit_history_platform.md` | `03_platform/09_audit_history/00_platform_overview.md`; `01_scope.md`; `02_architecture.md`; `03_contracts.md`; `04_runtime.md`; `05_security_and_audit.md`; `07_quality.md`; `08_operations.md`; `90_traceability.md` | Разделить; переходная копия удалена | Audit History, границы с business history, Workflow History и technical trace, а также трассировка ограничений MVP. | Подтверждённое содержание перенесено в канонический пакет `09_audit_history`; неподтверждённые ожидания сохранены в `90_traceability.md`; исходник в `docs` не удаляется. | Будущие решения AUD-05--07 перечислены в канонической трассировке |
| `docs/01 sources/13_platform_api_and_contracts.md` | `10_backlog/contract_governance/platform_api_contracts_backlog.md`; `03_platform/00_foundation/03_contracts.md`; `03_platform/03_object_runtime/03_contracts.md` | Разнесено по владельцам и backlog | Общие API, контракты, stable codes, error model и context. | Source смешивал application contracts, Object Runtime API, машинные контракты и будущие API governance rules. | Подтверждённое содержание ведут владельцы; нерешённые правила error model, versioning, endpoint naming, namespaces и `ExecutionProfileCode` ведутся в backlog. |
| `docs/01 sources/14_platform_runtime_conventions.md` | Foundation, Tenant/Security, Object Runtime, документы владельцев `03_platform/*/04_runtime.md` | Не переносить как отдельный документ | Общие runtime-соглашения, request pipeline, контекст, ошибки и правила выполнения. | Подтверждённые сведения уже покрыты владельцами; отдельный `06_runtime` создаёт дублирование проектной документации ядра. | Не создавать source-копию в `docs-new`; старый источник остаётся в `docs`. |
| `docs/01 sources/14.1_platform_runtime_reference_patterns.md` | `04_domain_modules/00_module_documentation_template.md` при отдельной нормализации шаблона | Не переносить как нормативный документ | Reference patterns слоев, entity logic, domain/application services и событий. | Это инженерные примеры, а не самостоятельное проектное решение. | Использовать только как справочный источник при нормализации module template, без source-копии в `docs-new`. |
| `docs/01 sources/15_mvp_foundation_plan.md` | `10_backlog/roadmap/preparation/09_two_week_platform_architecture_outcome_requirements.md`; `10_backlog/roadmap/preparation/platform_core_documentation_backlog.md` | Разнесено по актуальным плановым документам | План MVP Foundation, последовательность работ и scope. | Source-копия не является архитектурным стандартом; актуальные задачи и вход в архитектурный этап ведутся в соответствующих backlog-документах. | При необходимости связать задачи с issue tracker и актуальными статусами. |
| `docs/01 sources/16_module_architecture_standard.md` | `04_domain_modules/00_module_documentation_template.md` | Однозначно | Стандарт архитектуры прикладного модуля. | Соответствует шаблону документации модулей из раздела 10 `00`. | Проверить, нет ли правил, которые должны быть в `00_governance`, а не в шаблоне модуля. |
| `docs/01 sources/17_product_definition_reference_map.md` | Исходник в `docs/01 sources`; предметные решения — документы `04_domain_modules/02_product_process_definition/` | Не переносить как нормативный документ | Reference map demo/reference-кода, а не проектный документ модуля. | Краткие предметные решения уже находятся в документах Product & Process Definition; техническая инструкция остаётся историческим источником и не дублируется в `docs-new`. | Нет |
| `docs/01 sources/18_system_module_settings_decision.md` | `03_platform/07_settings`; `03_platform/02_configuration`; `10_backlog/requirements/` | Спорно | Решение по настройкам времени выполнения модулей и пользовательским предпочтениям. | Файл затрагивает Settings, runtime preferences и Configuration Platform; перенос в закрытую `09_common_application_contracts` не используется. | Являются ли эти настройки платформенной конфигурацией, runtime preferences или отдельным settings-сервисом? |

</details>

<details>
<summary><strong>Карточка переноса:</strong> <code>docs/01 sources/01_concept.md</code> → <code>docs-new/01_concept/01_product_concept.md</code> · однозначно</summary>

Документ: [docs-new/01_concept/01_product_concept.md](../01_concept/01_product_concept.md)

#### Паспорт переноса

| Поле | Значение |
| --- | --- |
| Исходный файл | `docs/01 sources/01_concept.md` |
| Целевой файл | `docs-new/01_concept/01_product_concept.md/01_concept.md (01 sources).md` |
| Тип документа по `00` | Concept / продуктовая концепция |
| Решение | Однозначно |
| Статус этапов | Копия исходника — готово; структурирование по `00` — следующий отдельный шаг; термины — следующий отдельный шаг; связи — следующий отдельный шаг. |
| Почему сюда | Исходник описывает верхнеуровневую концепцию DMP: контекст, проблемы, цель системы, ожидаемые эффекты и роль платформы в ИТ-ландшафте. Это полностью соответствует разделу `01_concept`. |
| Что важно не потерять | Контекст корпорации, типы производства, проблемы текущего ландшафта, цель DMP, ожидаемые эффекты, разграничение роли DMP и ERP. |

#### Краткое описание

Исходник фиксирует продуктовый смысл DMP: зачем нужна платформа, какие проблемы она решает, какой эффект ожидается и как DMP соотносится с ERP.

#### Разделы исходного документа

| Исходный раздел | Краткое пояснение |
| --- | --- |
| `Контекст` | Описывает исходную ситуацию корпорации: около 20 предприятий, разные типы производства, разрозненные ERP и локальные решения. |
| `Проблемы` | Фиксирует основные боли текущего ландшафта: прозрачность заказов, координация, аналитика, неравномерная автоматизация и WIP. |
| `Цель системы` | Формулирует назначение DMP как единой корпоративной платформы для жизненного цикла заказов и централизованного управления производством. |
| `Ожидаемые эффекты` | Перечисляет целевые бизнес-эффекты: прозрачность, сокращение цикла, загрузка оборудования, снижение WIP, унификация и единое пространство данных. |
| `Роль в ИТ-ландшафте` | Разделяет роль DMP и ERP: DMP отвечает за производственный контур, ERP остается в финансах, закупках и учете. |

#### Структура из `00`

Сначала показан короткий маршрут переноса, ниже — полная карта `00` с раскрываемыми крупными разделами. В дереве жирным выделен весь путь до файла.

##### Маршрут переноса

| Уровень | Узел |
| --- | --- |
| Корень | **`docs-new/`** |
| Раздел | **`01_concept/`** |
| Целевой каталог | **`01_product_concept.md/`** |
| Файл-источник | **`01_concept.md (01 sources).md`** |

##### Полная карта структуры `00`

<details>
<summary><code>00_governance/</code></summary>

- **`docs-new/`**
  - `00_governance/`
    - `00_documentation_strategy.md`
    - `01_document_types.md`
    - `02_document_lifecycle.md`
    - `03_review_and_approval.md`
    - `04_ai_usage_policy.md`
    - `05_naming_conventions.md`
    - `06_traceability_rules.md`
    - `requirements/`
      - `00_index.md`
      - `001_dmp_functional_requirements_and_constraints.md`

</details>

<details open>
<summary><strong>Целевой раздел: <code>01_concept/</code></strong></summary>

- **`docs-new/`**
  - **`01_concept/`**
    - **`01_product_concept.md/`**
      - **`01_concept.md (01 sources).md` ← перенесен этот файл**
    - `02_system_scope.md/`
      - `02_system_scope.md (01 sources).md`
    - `03_business_goals.md`
    - `04_stakeholders.md`
    - `05_success_metrics.md`

</details>

<details>
<summary><code>02_architecture/</code></summary>

- **`docs-new/`**
  - `02_architecture/`
    - `01_architecture_overview.md/`
      - `05_architecture.md (01 sources).md`
    - `02_architecture_principles.md`
    - `03_logical_architecture.md`
    - `04_service_architecture.md`
    - `05_data_architecture.md`
    - `06_deployment_architecture.md`
    - `07_integration_architecture.md`
    - `08_security_architecture.md`
    - `diagrams/`
      - `context.mmd`
      - `containers.mmd`
      - `deployment.mmd`

</details>

<details>
<summary><code>03_platform/</code></summary>

- **`docs-new/`**
  - `03_platform/`
    - `00_platform_core.md/`
      - `03_platform_core.md (01 sources).md`
    - `01_tenant_and_security/`
      - `01_overview.md`
      - `02_tenant_model.md`
      - `03_iam_model.md`
      - `04_authorization_contracts.md`
    - `02_configuration_platform/`
      - `01_overview.md`
      - `02_domain_model.md`
      - `03_data_configuration.md`
      - **`03_data_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `04_workflow_configuration.md`
      - `05_rule_configuration.md`
      - `06_condition_model.md`
      - **`06_condition_model/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `07_ui_configuration.md`
      - `08_reporting_configuration.md`
      - `09_output_configuration.md`
      - **`09_output_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `10_versioning_and_publish.md`
      - **`10_versioning_and_publish/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `11_import_export.md`
    - `03_workflow_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_action_execution.md`
      - `04_history.md`
    - `04_rule_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_result_model.md`
    - `05_value_set_data/`
      - `01_overview.md`
      - `02_value_set_model.md`
      - `03_value_items.md`
      - `04_tenant_override.md`
    - `06_integration/`
      - `01_overview.md`
      - `02_external_systems.md`
      - `03_connectors.md`
      - `04_integration_flows.md`
      - `05_external_id_mapping.md`
    - `07_event_foundation/`
      - `01_overview.md`
      - `02_event_envelope.md`
      - `03_outbox_inbox.md`
      - `04_idempotency.md`
    - `08_audit_history/`
      - `01_overview.md`
      - `02_audit_model.md`
      - `03_history_model.md`
      - `04_traceability.md`

    - `10_object_runtime/`
      - `01_overview.md`
      - `02_object_contract.md`
      - `03_change_execution.md`
      - `04_logic_and_validation_control.md`
      - `05_module_integration.md`
      - `06_current_implementation_gap.md`
      - `07_create_from_existing.md`
    - `11_numbering/`
      - `01_overview.md`
      - `02_design_decisions.md`
      - `03_model.md`
      - `04_runtime.md`
      - `05_object_runtime_integration.md`
      - `06_security.md`

</details>

<details>
<summary><code>04_domain_modules/</code></summary>

- **`docs-new/`**
  - `04_domain_modules/`
    - `01_module_index.md`
    - `00_module_documentation_template.md`
    - `00_common/`
      - `00_module_overview.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `05_rules.md`
      - `90_traceability_pr00.md`
    - `01_general_master_data/`
      - `00_module_overview.md`
      - `01_scope.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `04_workflows.md`
      - `05_rules.md`
      - `06_ui_views.md`
      - `07_reports_outputs.md`
      - `08_api_contracts.md`
      - `09_events.md`
      - `10_value_set_data_usage.md`
      - `11_permissions.md`
      - `12_audit_history.md`
      - `13_operations.md`
      - `14_test_strategy.md`
      - `backlog.md`
    - `02_product_process_definition/`
    - `03_plant_structure/`
    - `04_resource_management/`
    - `05_document_management/`
    - `06_project_management/`
    - `07_order_management/`
    - `08_planning_scheduling/`
    - `09_production_logistics/`
    - `10_shopfloor_execution/`
    - `11_quality_management/`
    - `12_machine_data_collection/`
    - `13_manufacturing_analytics/`

</details>

<details>
<summary><code>Исключённый раздел контрактов</code></summary>

Раздел `05_contracts/` исключён из целевой структуры. Старые файлы из `docs/05 contracts` остаются только исходниками сверки; в `docs-new` для них не создаётся ни отдельный раздел, ни новые дочерние папки у владельцев.

</details>

<details>
<summary><code>07_operations/</code></summary>

`07_operations/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной эксплуатационной документации: окружения, deployment, backup/restore, monitoring, incident response или platform-wide runbooks. Эксплуатация отдельных platform areas остаётся в `03_platform/*/08_operations.md`.

</details>

<details>
<summary><code>08_testing/</code></summary>

`08_testing/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной QA/test strategy, общих acceptance criteria, contract/integration/E2E/security/performance подходов или test data strategy. Проверки отдельных platform areas остаются в `03_platform/*/07_quality.md`, `90_traceability.md` и документах модулей.

</details>

<details>
<summary><code>09_decisions/</code></summary>

`09_decisions/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только как будущий механизм для отдельного утверждённого ADR; draft-копии из `docs/adr/drafts` остаются вне `docs-new`.

</details>

<details>
<summary><code>10_backlog/</code></summary>

- **`docs-new/`**
  - `10_backlog/`
    - `epics/`
      - `epic-configuration-platform-mvp.md`
      - `epic-workflow-foundation.md`
      - `epic-value-set-data-mvp.md`
    - `requirements/`
      - `req-configuration-publish.md`
      - `req-effective-ui-resolution.md`
    - `roadmap/`
      - `roadmap-mvp.md`
      - `roadmap-platform-foundation.md`

</details>

<details>
<summary><code>11_glossary/</code></summary>

- **`docs-new/`**
  - `11_glossary/`
    - `glossary.md`
    - `platform_terms.md`
    - `manufacturing_terms.md`
    - `ui_terms.md`
    - `security_terms.md`

</details>

<details>
<summary><code>12_appendices/</code></summary>

- **`docs-new/`**
  - `12_appendices/`
    - `imported_materials/`
    - `legacy_docs/`
    - `meeting_notes/`
    - `research/`
    - `vendor_references/`

</details>


</details>

<details>
<summary><strong>Карточка переноса:</strong> <code>docs/01 sources/02_system_scope.md</code> → <code>docs-new/01_concept/02_system_scope.md/02_system_scope.md (01 sources).md</code> · однозначно</summary>

Документ: [docs-new/01_concept/02_system_scope.md](../01_concept/02_system_scope.md)

#### Паспорт переноса

| Поле | Значение |
| --- | --- |
| Исходный файл | `docs/01 sources/02_system_scope.md` |
| Целевой файл | `docs-new/01_concept/02_system_scope.md/02_system_scope.md (01 sources).md` |
| Тип документа по `00` | Scope / границы системы |
| Решение | Однозначно |
| Статус этапов | Копия исходника — готово; структурирование по `00` — следующий отдельный шаг; термины — следующий отдельный шаг; связи — следующий отдельный шаг. |
| Почему сюда | Исходник описывает границы DMP: что входит в систему, какие интеграции предусмотрены, что явно не входит, и как различаются группы данных. |
| Что важно не потерять | Входящий scope, исключения из scope, ERP/CAD/PDM/MDC-интеграции, Data Scope Clarification и открытые вопросы по типам данных. |

#### Краткое описание

Исходник задает границы системы: функциональные области, интеграции, явные исключения и первичное разделение типов данных.

#### Разделы исходного документа

| Исходный раздел | Краткое пояснение |
| --- | --- |
| `Входит в систему` | Задает функциональный scope DMP: управление производством, логистика, операции, качество, оборудование и аналитика. |
| `Управление производством` | Раскрывает производственные заказы, планирование, диспетчеризацию и управление цехами. |
| `Производственная логистика` | Описывает движение материалов, WIP и комплектацию как часть системной границы. |
| `Управление операциями` | Фиксирует выполнение операций и терминалы рабочих мест. |
| `Управление качеством` | Включает контроль операций и несоответствия. |
| `Оборудование` | Описывает мониторинг, телеметрию и OEE как часть системного scope. |
| `Аналитика` | Фиксирует KPI, загрузку оборудования и сроки заказов. |
| `Интеграции` | Описывает обмен с ERP, CAD/PDM и MDC. |
| `ERP -> Платформа` | Перечисляет входящие данные из ERP: заказы, материалы и остатки. |
| `Платформа -> ERP` | Перечисляет обратную передачу статусов выполнения и фактических данных. |
| `CAD / PDM -> Платформа` | Фиксирует передачу спецификаций и версий изделий. |
| `MDC -> Платформа` | Фиксирует телеметрию и статусы оборудования. |
| `Не входит (явно не реализуется)` | Явно исключает финансовый учет, бухгалтерию и закупки как зоны ERP. |
| `Data Scope Clarification` | Разводит ValueSets, domain master data и operational/execution data как исходную постановку вопроса по данным. |

#### Структура из `00`

Сначала показан короткий маршрут переноса, ниже — полная карта `00` с раскрываемыми крупными разделами. В дереве жирным выделен весь путь до файла.

##### Маршрут переноса

| Уровень | Узел |
| --- | --- |
| Корень | **`docs-new/`** |
| Раздел | **`01_concept/`** |
| Целевой каталог | **`02_system_scope.md/`** |
| Файл-источник | **`02_system_scope.md (01 sources).md`** |

##### Полная карта структуры `00`

<details>
<summary><code>00_governance/</code></summary>

- **`docs-new/`**
  - `00_governance/`
    - `00_documentation_strategy.md`
    - `01_document_types.md`
    - `02_document_lifecycle.md`
    - `03_review_and_approval.md`
    - `04_ai_usage_policy.md`
    - `05_naming_conventions.md`
    - `06_traceability_rules.md`
    - `requirements/`
      - `00_index.md`
      - `001_dmp_functional_requirements_and_constraints.md`

</details>

<details open>
<summary><strong>Целевой раздел: <code>01_concept/</code></strong></summary>

- **`docs-new/`**
  - **`01_concept/`**
    - `01_product_concept.md/`
      - `01_concept.md (01 sources).md`
    - **`02_system_scope.md/`**
      - **`02_system_scope.md (01 sources).md` ← перенесен этот файл**
    - `03_business_goals.md`
    - `04_stakeholders.md`
    - `05_success_metrics.md`

</details>

<details>
<summary><code>02_architecture/</code></summary>

- **`docs-new/`**
  - `02_architecture/`
    - `01_architecture_overview.md/`
      - `05_architecture.md (01 sources).md`
    - `02_architecture_principles.md`
    - `03_logical_architecture.md`
    - `04_service_architecture.md`
    - `05_data_architecture.md`
    - `06_deployment_architecture.md`
    - `07_integration_architecture.md`
    - `08_security_architecture.md`
    - `diagrams/`
      - `context.mmd`
      - `containers.mmd`
      - `deployment.mmd`

</details>

<details>
<summary><code>03_platform/</code></summary>

- **`docs-new/`**
  - `03_platform/`
    - `00_platform_core.md/`
      - `03_platform_core.md (01 sources).md`
    - `01_tenant_and_security/`
      - `01_overview.md`
      - `02_tenant_model.md`
      - `03_iam_model.md`
      - `04_authorization_contracts.md`
    - `02_configuration_platform/`
      - `01_overview.md`
      - `02_domain_model.md`
      - `03_data_configuration.md`
      - **`03_data_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `04_workflow_configuration.md`
      - `05_rule_configuration.md`
      - `06_condition_model.md`
      - **`06_condition_model/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `07_ui_configuration.md`
      - `08_reporting_configuration.md`
      - `09_output_configuration.md`
      - **`09_output_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `10_versioning_and_publish.md`
      - **`10_versioning_and_publish/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `11_import_export.md`
    - `03_workflow_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_action_execution.md`
      - `04_history.md`
    - `04_rule_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_result_model.md`
    - `05_value_set_data/`
      - `01_overview.md`
      - `02_value_set_model.md`
      - `03_value_items.md`
      - `04_tenant_override.md`
    - `06_integration/`
      - `01_overview.md`
      - `02_external_systems.md`
      - `03_connectors.md`
      - `04_integration_flows.md`
      - `05_external_id_mapping.md`
    - `07_event_foundation/`
      - `01_overview.md`
      - `02_event_envelope.md`
      - `03_outbox_inbox.md`
      - `04_idempotency.md`
    - `08_audit_history/`
      - `01_overview.md`
      - `02_audit_model.md`
      - `03_history_model.md`
      - `04_traceability.md`

    - `10_object_runtime/`
      - `01_overview.md`
      - `02_object_contract.md`
      - `03_change_execution.md`
      - `04_logic_and_validation_control.md`
      - `05_module_integration.md`
      - `06_current_implementation_gap.md`
      - `07_create_from_existing.md`
    - `11_numbering/`
      - `01_overview.md`
      - `02_design_decisions.md`
      - `03_model.md`
      - `04_runtime.md`
      - `05_object_runtime_integration.md`
      - `06_security.md`

</details>

<details>
<summary><code>04_domain_modules/</code></summary>

- **`docs-new/`**
  - `04_domain_modules/`
    - `01_module_index.md`
    - `00_module_documentation_template.md`
    - `00_common/`
      - `00_module_overview.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `05_rules.md`
      - `90_traceability_pr00.md`
    - `01_general_master_data/`
      - `00_module_overview.md`
      - `01_scope.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `04_workflows.md`
      - `05_rules.md`
      - `06_ui_views.md`
      - `07_reports_outputs.md`
      - `08_api_contracts.md`
      - `09_events.md`
      - `10_value_set_data_usage.md`
      - `11_permissions.md`
      - `12_audit_history.md`
      - `13_operations.md`
      - `14_test_strategy.md`
      - `backlog.md`
    - `02_product_process_definition/`
    - `03_plant_structure/`
    - `04_resource_management/`
    - `05_document_management/`
    - `06_project_management/`
    - `07_order_management/`
    - `08_planning_scheduling/`
    - `09_production_logistics/`
    - `10_shopfloor_execution/`
    - `11_quality_management/`
    - `12_machine_data_collection/`
    - `13_manufacturing_analytics/`

</details>

<details>
<summary><code>Исключённый раздел контрактов</code></summary>

Раздел `05_contracts/` исключён из целевой структуры. Старые файлы из `docs/05 contracts` остаются только исходниками сверки; в `docs-new` для них не создаётся ни отдельный раздел, ни новые дочерние папки у владельцев.

</details>

<details>
<summary><code>07_operations/</code></summary>

`07_operations/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной эксплуатационной документации: окружения, deployment, backup/restore, monitoring, incident response или platform-wide runbooks. Эксплуатация отдельных platform areas остаётся в `03_platform/*/08_operations.md`.

</details>

<details>
<summary><code>08_testing/</code></summary>

`08_testing/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной QA/test strategy, общих acceptance criteria, contract/integration/E2E/security/performance подходов или test data strategy. Проверки отдельных platform areas остаются в `03_platform/*/07_quality.md`, `90_traceability.md` и документах модулей.

</details>

<details>
<summary><code>09_decisions/</code></summary>

`09_decisions/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только как будущий механизм для отдельного утверждённого ADR; draft-копии из `docs/adr/drafts` остаются вне `docs-new`.

</details>

<details>
<summary><code>10_backlog/</code></summary>

- **`docs-new/`**
  - `10_backlog/`
    - `epics/`
      - `epic-configuration-platform-mvp.md`
      - `epic-workflow-foundation.md`
      - `epic-value-set-data-mvp.md`
    - `requirements/`
      - `req-configuration-publish.md`
      - `req-effective-ui-resolution.md`
    - `roadmap/`
      - `roadmap-mvp.md`
      - `roadmap-platform-foundation.md`

</details>

<details>
<summary><code>11_glossary/</code></summary>

- **`docs-new/`**
  - `11_glossary/`
    - `glossary.md`
    - `platform_terms.md`
    - `manufacturing_terms.md`
    - `ui_terms.md`
    - `security_terms.md`

</details>

<details>
<summary><code>12_appendices/</code></summary>

- **`docs-new/`**
  - `12_appendices/`
    - `imported_materials/`
    - `legacy_docs/`
    - `meeting_notes/`
    - `research/`
    - `vendor_references/`

</details>


</details>

<details>
<summary><strong>Карточка переноса:</strong> <code>docs/01 sources/03_platform_core.md</code> → целевые документы Platform Core · разнесено по владельцам</summary>

Документы-владельцы: [обзор архитектуры DMP](../02_architecture/01_architecture_overview.md), [граница системы DMP](../01_concept/02_system_scope.md), [трассировка Foundation](../03_platform/00_foundation/90_traceability.md) и документы соответствующих `03_platform/*` areas.

#### Паспорт переноса

| Поле | Значение |
| --- | --- |
| Исходный файл | `docs/01 sources/03_platform_core.md` |
| Целевые документы | `01_concept/02_system_scope.md`, `02_architecture/01_architecture_overview.md`, `03_platform/*/`, `03_platform/00_foundation/90_traceability.md` |
| Тип документа по `00` | Platform design / обзор платформенного ядра |
| Решение | Разнесено по владельцам |
| Статус этапов | Копия исходника использована как переходный материал; структурирование завершено для верхнего концептуального и архитектурного слоя; staging-папка закрыта. |
| Почему так | Исходник фиксирует состав, границы, принципы и MVP-scope всего Platform Core. Это не один документ Foundation: общая рамка принадлежит `01_concept` и `02_architecture`, а детали capability — их владельцам в `03_platform`. |
| Что важно не потерять | Границы Platform Core, состав foundation-блоков, принципы данных и владения, MVP scope, common application contracts и ограничения для прикладных модулей. |

#### Краткое описание

Исходник описывает Platform Core как набор общих платформенных возможностей, правил и минимального MVP-фундамента для прикладных модулей. После нормализации он больше не используется как отдельная рабочая точка входа.

#### Разделы исходного документа

| Исходный раздел | Краткое пояснение |
| --- | --- |
| `1. Назначение документа` | Определяет Platform Core как общие уровни, сервисы и механизмы, на которых строятся прикладные модули. |
| `2. Почему Platform Core определяется раньше прикладной части` | Обосновывает, почему сначала фиксируются общие платформенные механизмы, а затем прикладные модули. |
| `3. Место Platform Core в общей архитектуре` | Показывает связь Platform Core с Platform Layer, Platform Services, data foundations и integration foundations. |
| `4. Границы Platform Core` | Разделяет, что входит в платформенное ядро, а что остается прикладной областью модулей. |
| `5. Архитектурные принципы Platform Core` | Фиксирует принципы единого корпоративного ядра, конфигурируемости, слабой связанности, поэтапной цифровизации и гибридного развертывания. |
| `6. Состав Platform Core` | Раскрывает foundation-блоки: tenant/IAM, configuration, workflow, rules, ValueSets, integration, events, audit/history и common contracts. |
| `7. Принципы данных и владения данными` | Разводит данные наборов значений, domain master data и operational data; фиксирует владение данными сервисами. |
| `8. MVP scope Platform Core` | Делит возможности Platform Core на обязательные для MVP и отложенные. |
| `9. Нефункциональные требования к Platform Core` | Фиксирует масштабирование, tenant isolation, гибридное deployment, расширяемость и стек. |
| `10. Что Platform Core задает для дальнейшей прикладной разработки` | Описывает, какие платформенные контракты должны учитывать прикладные модули. |
| `11. Следующие документы после фиксации Platform Core` | Задает дальнейшую последовательность: platform contracts, business slice, прикладная детализация. |

#### Структура из `00`

Сначала показан короткий маршрут переноса, ниже — полная карта `00` с раскрываемыми крупными разделами. В дереве жирным выделен весь путь до файла.

##### Маршрут переноса

| Уровень | Узел |
| --- | --- |
| Корень | **`docs-new/`** |
| Верхняя концепция | **`01_concept/02_system_scope.md`** |
| Верхняя архитектура | **`02_architecture/01_architecture_overview.md`** |
| Platform areas | **`03_platform/*/`** |
| Трассировка закрытия | **`03_platform/00_foundation/90_traceability.md`** |

##### Полная карта структуры `00`

<details>
<summary><code>00_governance/</code></summary>

- **`docs-new/`**
  - `00_governance/`
    - `00_documentation_strategy.md`
    - `01_document_types.md`
    - `02_document_lifecycle.md`
    - `03_review_and_approval.md`
    - `04_ai_usage_policy.md`
    - `05_naming_conventions.md`
    - `06_traceability_rules.md`
    - `requirements/`
      - `00_index.md`
      - `001_dmp_functional_requirements_and_constraints.md`

</details>

<details>
<summary><code>01_concept/</code></summary>

- **`docs-new/`**
  - `01_concept/`
    - `01_product_concept.md/`
      - `01_concept.md (01 sources).md`
    - `02_system_scope.md/`
      - `02_system_scope.md (01 sources).md`
    - `03_business_goals.md`
    - `04_stakeholders.md`
    - `05_success_metrics.md`

</details>

<details>
<summary><code>02_architecture/</code></summary>

- **`docs-new/`**
  - `02_architecture/`
    - `01_architecture_overview.md/`
      - `05_architecture.md (01 sources).md`
    - `02_architecture_principles.md`
    - `03_logical_architecture.md`
    - `04_service_architecture.md`
    - `05_data_architecture.md`
    - `06_deployment_architecture.md`
    - `07_integration_architecture.md`
    - `08_security_architecture.md`
    - `diagrams/`
      - `context.mmd`
      - `containers.mmd`
      - `deployment.mmd`

</details>

<details open>
<summary><strong>Целевой раздел: <code>03_platform/</code></strong></summary>

- **`docs-new/`**
  - **`03_platform/`**
    - **`00_platform_core.md/`**
      - **`03_platform_core.md (01 sources).md` ← перенесен этот файл**
    - `01_tenant_and_security/`
      - `01_overview.md`
      - `02_tenant_model.md`
      - `03_iam_model.md`
      - `04_authorization_contracts.md`
    - `02_configuration_platform/`
      - `01_overview.md`
      - `02_domain_model.md`
      - `03_data_configuration.md`
      - **`03_data_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `04_workflow_configuration.md`
      - `05_rule_configuration.md`
      - `06_condition_model.md`
      - **`06_condition_model/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `07_ui_configuration.md`
      - `08_reporting_configuration.md`
      - `09_output_configuration.md`
      - **`09_output_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `10_versioning_and_publish.md`
      - **`10_versioning_and_publish/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `11_import_export.md`
    - `03_workflow_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_action_execution.md`
      - `04_history.md`
    - `04_rule_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_result_model.md`
    - `05_value_set_data/`
      - `01_overview.md`
      - `02_value_set_model.md`
      - `03_value_items.md`
      - `04_tenant_override.md`
    - `06_integration/`
      - `01_overview.md`
      - `02_external_systems.md`
      - `03_connectors.md`
      - `04_integration_flows.md`
      - `05_external_id_mapping.md`
    - `07_event_foundation/`
      - `01_overview.md`
      - `02_event_envelope.md`
      - `03_outbox_inbox.md`
      - `04_idempotency.md`
    - `08_audit_history/`
      - `01_overview.md`
      - `02_audit_model.md`
      - `03_history_model.md`
      - `04_traceability.md`

    - `10_object_runtime/`
      - `01_overview.md`
      - `02_object_contract.md`
      - `03_change_execution.md`
      - `04_logic_and_validation_control.md`
      - `05_module_integration.md`
      - `06_current_implementation_gap.md`
      - `07_create_from_existing.md`
    - `11_numbering/`
      - `01_overview.md`
      - `02_design_decisions.md`
      - `03_model.md`
      - `04_runtime.md`
      - `05_object_runtime_integration.md`
      - `06_security.md`

</details>

<details>
<summary><code>04_domain_modules/</code></summary>

- **`docs-new/`**
  - `04_domain_modules/`
    - `01_module_index.md`
    - `00_module_documentation_template.md`
    - `00_common/`
      - `00_module_overview.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `05_rules.md`
      - `90_traceability_pr00.md`
    - `01_general_master_data/`
      - `00_module_overview.md`
      - `01_scope.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `04_workflows.md`
      - `05_rules.md`
      - `06_ui_views.md`
      - `07_reports_outputs.md`
      - `08_api_contracts.md`
      - `09_events.md`
      - `10_value_set_data_usage.md`
      - `11_permissions.md`
      - `12_audit_history.md`
      - `13_operations.md`
      - `14_test_strategy.md`
      - `backlog.md`
    - `02_product_process_definition/`
    - `03_plant_structure/`
    - `04_resource_management/`
    - `05_document_management/`
    - `06_project_management/`
    - `07_order_management/`
    - `08_planning_scheduling/`
    - `09_production_logistics/`
    - `10_shopfloor_execution/`
    - `11_quality_management/`
    - `12_machine_data_collection/`
    - `13_manufacturing_analytics/`

</details>

<details>
<summary><code>Исключённый раздел контрактов</code></summary>

Раздел `05_contracts/` исключён из целевой структуры. Старые файлы из `docs/05 contracts` остаются только исходниками сверки; в `docs-new` для них не создаётся ни отдельный раздел, ни новые дочерние папки у владельцев.

</details>

<details>
<summary><code>07_operations/</code></summary>

`07_operations/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной эксплуатационной документации: окружения, deployment, backup/restore, monitoring, incident response или platform-wide runbooks. Эксплуатация отдельных platform areas остаётся в `03_platform/*/08_operations.md`.

</details>

<details>
<summary><code>08_testing/</code></summary>

`08_testing/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной QA/test strategy, общих acceptance criteria, contract/integration/E2E/security/performance подходов или test data strategy. Проверки отдельных platform areas остаются в `03_platform/*/07_quality.md`, `90_traceability.md` и документах модулей.

</details>

<details>
<summary><code>09_decisions/</code></summary>

`09_decisions/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только как будущий механизм для отдельного утверждённого ADR; draft-копии из `docs/adr/drafts` остаются вне `docs-new`.

</details>

<details>
<summary><code>10_backlog/</code></summary>

- **`docs-new/`**
  - `10_backlog/`
    - `epics/`
      - `epic-configuration-platform-mvp.md`
      - `epic-workflow-foundation.md`
      - `epic-value-set-data-mvp.md`
    - `requirements/`
      - `req-configuration-publish.md`
      - `req-effective-ui-resolution.md`
    - `roadmap/`
      - `roadmap-mvp.md`
      - `roadmap-platform-foundation.md`

</details>

<details>
<summary><code>11_glossary/</code></summary>

- **`docs-new/`**
  - `11_glossary/`
    - `glossary.md`
    - `platform_terms.md`
    - `manufacturing_terms.md`
    - `ui_terms.md`
    - `security_terms.md`

</details>

<details>
<summary><code>12_appendices/</code></summary>

- **`docs-new/`**
  - `12_appendices/`
    - `imported_materials/`
    - `legacy_docs/`
    - `meeting_notes/`
    - `research/`
    - `vendor_references/`

</details>


</details>

<details>
<summary><strong>Карточка переноса:</strong> <code>docs/01 sources/05_architecture.md</code> → <code>docs-new/02_architecture/01_architecture_overview.md/05_architecture.md (01 sources).md</code> · однозначно</summary>

Документ: [docs-new/02_architecture/01_architecture_overview.md](../02_architecture/01_architecture_overview.md)

#### Паспорт переноса

| Поле | Значение |
| --- | --- |
| Исходный файл | `docs/01 sources/05_architecture.md` |
| Целевой файл | `docs-new/02_architecture/01_architecture_overview.md/05_architecture.md (01 sources).md` |
| Тип документа по `00` | Architecture overview |
| Решение | Однозначно |
| Статус этапов | Копия исходника — готово; структурирование по `00` — следующий отдельный шаг; термины — следующий отдельный шаг; связи — следующий отдельный шаг. |
| Почему сюда | Исходник описывает архитектурные принципы, уровни, сервисную архитектуру, архитектуру данных, взаимодействие сервисов и deployment. |
| Что важно не потерять | Архитектурные принципы, слои, platform/domain/integration services, Hybrid Multi-Tenant, типы данных, правила владения данными и deployment-модель. |

#### Краткое описание

Исходник является общим архитектурным обзором DMP: принципы, слои, сервисы, данные, взаимодействие сервисов и deployment-модель.

#### Разделы исходного документа

| Исходный раздел | Краткое пояснение |
| --- | --- |
| `Архитектурные принципы` | Фиксирует базовые принципы архитектуры: единая платформа, импортонезависимость, конфигурируемость, стандартизация, поэтапная цифровизация и интеграции. |
| `Архитектурные уровни` | Описывает слои системы: User Layer, Application Layer, Platform Layer и Data Layer. |
| `Сервисная архитектура` | Делит сервисы на Platform Services, Domain Services и Integration & External Services. |
| `Архитектура данных` | Описывает Hybrid Multi-Tenant, общие данные, данные предприятий, ValueSets, Domain Master Data и операционные данные. |
| `Принципы управления данными и взаимодействия сервисов` | Фиксирует владение данными сервисами, запрет прямого доступа к чужим таблицам и взаимодействие через API/события. |
| `Deployment` | Описывает cloud/local-гибридную модель развертывания. |

#### Структура из `00`

Сначала показан короткий маршрут переноса, ниже — полная карта `00` с раскрываемыми крупными разделами. В дереве жирным выделен весь путь до файла.

##### Маршрут переноса

| Уровень | Узел |
| --- | --- |
| Корень | **`docs-new/`** |
| Раздел | **`02_architecture/`** |
| Целевой каталог | **`01_architecture_overview.md/`** |
| Файл-источник | **`05_architecture.md (01 sources).md`** |

##### Полная карта структуры `00`

<details>
<summary><code>00_governance/</code></summary>

- **`docs-new/`**
  - `00_governance/`
    - `00_documentation_strategy.md`
    - `01_document_types.md`
    - `02_document_lifecycle.md`
    - `03_review_and_approval.md`
    - `04_ai_usage_policy.md`
    - `05_naming_conventions.md`
    - `06_traceability_rules.md`
    - `requirements/`
      - `00_index.md`
      - `001_dmp_functional_requirements_and_constraints.md`

</details>

<details>
<summary><code>01_concept/</code></summary>

- **`docs-new/`**
  - `01_concept/`
    - `01_product_concept.md/`
      - `01_concept.md (01 sources).md`
    - `02_system_scope.md/`
      - `02_system_scope.md (01 sources).md`
    - `03_business_goals.md`
    - `04_stakeholders.md`
    - `05_success_metrics.md`

</details>

<details open>
<summary><strong>Целевой раздел: <code>02_architecture/</code></strong></summary>

- **`docs-new/`**
  - **`02_architecture/`**
    - **`01_architecture_overview.md/`**
      - **`05_architecture.md (01 sources).md` ← перенесен этот файл**
    - `02_architecture_principles.md`
    - `03_logical_architecture.md`
    - `04_service_architecture.md`
    - `05_data_architecture.md`
    - `06_deployment_architecture.md`
    - `07_integration_architecture.md`
    - `08_security_architecture.md`
    - `diagrams/`
      - `context.mmd`
      - `containers.mmd`
      - `deployment.mmd`

</details>

<details>
<summary><code>03_platform/</code></summary>

- **`docs-new/`**
  - `03_platform/`
    - `00_platform_core.md/`
      - `03_platform_core.md (01 sources).md`
    - `01_tenant_and_security/`
      - `01_overview.md`
      - `02_tenant_model.md`
      - `03_iam_model.md`
      - `04_authorization_contracts.md`
    - `02_configuration_platform/`
      - `01_overview.md`
      - `02_domain_model.md`
      - `03_data_configuration.md`
      - **`03_data_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `04_workflow_configuration.md`
      - `05_rule_configuration.md`
      - `06_condition_model.md`
      - **`06_condition_model/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `07_ui_configuration.md`
      - `08_reporting_configuration.md`
      - `09_output_configuration.md`
      - **`09_output_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `10_versioning_and_publish.md`
      - **`10_versioning_and_publish/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `11_import_export.md`
    - `03_workflow_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_action_execution.md`
      - `04_history.md`
    - `04_rule_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_result_model.md`
    - `05_value_set_data/`
      - `01_overview.md`
      - `02_value_set_model.md`
      - `03_value_items.md`
      - `04_tenant_override.md`
    - `06_integration/`
      - `01_overview.md`
      - `02_external_systems.md`
      - `03_connectors.md`
      - `04_integration_flows.md`
      - `05_external_id_mapping.md`
    - `07_event_foundation/`
      - `01_overview.md`
      - `02_event_envelope.md`
      - `03_outbox_inbox.md`
      - `04_idempotency.md`
    - `08_audit_history/`
      - `01_overview.md`
      - `02_audit_model.md`
      - `03_history_model.md`
      - `04_traceability.md`

    - `10_object_runtime/`
      - `01_overview.md`
      - `02_object_contract.md`
      - `03_change_execution.md`
      - `04_logic_and_validation_control.md`
      - `05_module_integration.md`
      - `06_current_implementation_gap.md`
      - `07_create_from_existing.md`
    - `11_numbering/`
      - `01_overview.md`
      - `02_design_decisions.md`
      - `03_model.md`
      - `04_runtime.md`
      - `05_object_runtime_integration.md`
      - `06_security.md`

</details>

<details>
<summary><code>04_domain_modules/</code></summary>

- **`docs-new/`**
  - `04_domain_modules/`
    - `01_module_index.md`
    - `00_module_documentation_template.md`
    - `00_common/`
      - `00_module_overview.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `05_rules.md`
      - `90_traceability_pr00.md`
    - `01_general_master_data/`
      - `00_module_overview.md`
      - `01_scope.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `04_workflows.md`
      - `05_rules.md`
      - `06_ui_views.md`
      - `07_reports_outputs.md`
      - `08_api_contracts.md`
      - `09_events.md`
      - `10_value_set_data_usage.md`
      - `11_permissions.md`
      - `12_audit_history.md`
      - `13_operations.md`
      - `14_test_strategy.md`
      - `backlog.md`
    - `02_product_process_definition/`
    - `03_plant_structure/`
    - `04_resource_management/`
    - `05_document_management/`
    - `06_project_management/`
    - `07_order_management/`
    - `08_planning_scheduling/`
    - `09_production_logistics/`
    - `10_shopfloor_execution/`
    - `11_quality_management/`
    - `12_machine_data_collection/`
    - `13_manufacturing_analytics/`

</details>

<details>
<summary><code>Исключённый раздел контрактов</code></summary>

Раздел `05_contracts/` исключён из целевой структуры. Старые файлы из `docs/05 contracts` остаются только исходниками сверки; в `docs-new` для них не создаётся ни отдельный раздел, ни новые дочерние папки у владельцев.

</details>

<details>
<summary><code>07_operations/</code></summary>

`07_operations/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной эксплуатационной документации: окружения, deployment, backup/restore, monitoring, incident response или platform-wide runbooks. Эксплуатация отдельных platform areas остаётся в `03_platform/*/08_operations.md`.

</details>

<details>
<summary><code>08_testing/</code></summary>

`08_testing/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной QA/test strategy, общих acceptance criteria, contract/integration/E2E/security/performance подходов или test data strategy. Проверки отдельных platform areas остаются в `03_platform/*/07_quality.md`, `90_traceability.md` и документах модулей.

</details>

<details>
<summary><code>09_decisions/</code></summary>

`09_decisions/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только как будущий механизм для отдельного утверждённого ADR; draft-копии из `docs/adr/drafts` остаются вне `docs-new`.

</details>

<details>
<summary><code>10_backlog/</code></summary>

- **`docs-new/`**
  - `10_backlog/`
    - `epics/`
      - `epic-configuration-platform-mvp.md`
      - `epic-workflow-foundation.md`
      - `epic-value-set-data-mvp.md`
    - `requirements/`
      - `req-configuration-publish.md`
      - `req-effective-ui-resolution.md`
    - `roadmap/`
      - `roadmap-mvp.md`
      - `roadmap-platform-foundation.md`

</details>

<details>
<summary><code>11_glossary/</code></summary>

- **`docs-new/`**
  - `11_glossary/`
    - `glossary.md`
    - `platform_terms.md`
    - `manufacturing_terms.md`
    - `ui_terms.md`
    - `security_terms.md`

</details>

<details>
<summary><code>12_appendices/</code></summary>

- **`docs-new/`**
  - `12_appendices/`
    - `imported_materials/`
    - `legacy_docs/`
    - `meeting_notes/`
    - `research/`
    - `vendor_references/`

</details>


</details>

<details>
<summary><strong>Карточка переноса:</strong> <code>docs/01 sources/06_configuration_platform.md</code> → <code>docs-new/03_platform/02_configuration_platform/01_overview.md/06_configuration_platform.md (01 sources).md</code> · однозначно</summary>

Актуальный документ: [обзор Configuration](../03_platform/02_configuration/00_platform_overview.md).

#### Паспорт переноса

| Поле | Значение |
| --- | --- |
| Исходный файл | `docs/01 sources/06_configuration_platform.md` |
| Целевой файл | `docs-new/03_platform/02_configuration_platform/01_overview.md/06_configuration_platform.md (01 sources).md` |
| Тип документа по `00` | Platform design / Configuration Platform overview |
| Решение | Однозначно |
| Статус этапов | Копия исходника — готово; структурирование по `00` — следующий отдельный шаг; термины — следующий отдельный шаг; связи — следующий отдельный шаг. |
| Почему сюда | Исходник является обзором Configuration Platform: назначение, границы, связь с Object Runtime, виды конфигурации, уровни применения, версии, публикация и runtime-применение. |
| Что важно не потерять | Граница ответственности Configuration Platform, связь с Object Runtime, baseline package, виды конфигурации, уровни scope, публикация, effective configuration и запрет обхода runtime-механизмов. |

#### Краткое описание

Документ задает обзор Configuration Platform как механизма адаптации системы без изменения кода и фиксирует, что платформа конфигурирования хранит конфигурацию, но не исполняет workflow, правила, IAM и бизнес-операции.

#### Разделы исходного документа

| Исходный раздел | Краткое пояснение |
| --- | --- |
| `1. Назначение` | Определяет Configuration Platform как механизм адаптации системы без изменения кода. |
| `2. Граница ответственности` | Разводит, что относится к Configuration Platform, а что исполняют другие платформенные или прикладные сервисы. |
| `3. Связь с Object Runtime` | Фиксирует, что Configuration Platform не является источником исполняемой модели стандартного объекта и работает поверх Object Runtime. |
| `3.1 Реестр модулей и baseline package` | Разделяет registry contracts, baseline package, SystemBaseline и пользовательские переопределения. |
| `4. Виды конфигурации` | Описывает основные типы конфигурации: data, workflow, rules, UI, reporting, output/export. |
| `5. Границы конфигурируемости` | Фиксирует, какие изменения допустимы через конфигурацию, а какие остаются в коде и runtime-механизмах. |
| `6. Уровни применения конфигурации` | Описывает уровни применения и наследования конфигурации. |
| `7. Политика жизненного цикла конфигурации` | Фиксирует стадии жизни конфигурации и правила перехода между ними. |
| `8. Наследование и переопределение` | Описывает правила применения базовых настроек и переопределений. |
| `9. Версии и публикация` | Фиксирует версионирование и публикацию конфигурации. |
| `10. Разрешение конфигурации во время работы системы` | Описывает получение итоговой effective configuration в runtime. |
| `11. Модель данных конфигурации` | Дает обзор ключевых сущностей модели данных Configuration Platform. |
| `12. Валидация конфигурации` | Описывает проверки перед публикацией и применением конфигурации. |
| `13. Аудит и контроль изменений` | Фиксирует требования к журналированию изменений конфигурации. |
| `14. Import / Export` | Описывает переносимость и обмен конфигурацией. |
| `15. Модель администрирования` | Описывает административную модель работы с конфигурацией. |
| `16. Интеграция с другими сервисами платформы` | Показывает связи Configuration Platform с Workflow Engine, Rule Engine, IAM, ValueSets, Domain Modules и Object Runtime. |
| `17. Объем MVP` | Делит возможности Configuration Platform на обязательные для MVP и отложенные. |
| `18. Открытые проектные решения` | Фиксирует вопросы, которые нельзя закрывать без отдельного решения. |

#### Структура из `00`

Сначала показан короткий маршрут переноса, ниже — полная карта `00` с раскрываемыми крупными разделами. В дереве жирным выделен весь путь до файла.

##### Маршрут переноса

| Уровень | Узел |
| --- | --- |
| Корень | **`docs-new/`** |
| Раздел | **`03_platform/`** |
| Подраздел | **`02_configuration_platform/`** |
| Файл | **`01_overview.md`** |

##### Полная карта структуры `00`

<details>
<summary><code>00_governance/</code></summary>

- **`docs-new/`**
  - `00_governance/`
    - `00_documentation_strategy.md`
    - `01_document_types.md`
    - `02_document_lifecycle.md`
    - `03_review_and_approval.md`
    - `04_ai_usage_policy.md`
    - `05_naming_conventions.md`
    - `06_traceability_rules.md`
    - `requirements/`
      - `00_index.md`
      - `001_dmp_functional_requirements_and_constraints.md`

</details>

<details>
<summary><code>01_concept/</code></summary>

- **`docs-new/`**
  - `01_concept/`
    - `01_product_concept.md/`
      - `01_concept.md (01 sources).md`
    - `02_system_scope.md/`
      - `02_system_scope.md (01 sources).md`
    - `03_business_goals.md`
    - `04_stakeholders.md`
    - `05_success_metrics.md`

</details>

<details>
<summary><code>02_architecture/</code></summary>

- **`docs-new/`**
  - `02_architecture/`
    - `01_architecture_overview.md/`
      - `05_architecture.md (01 sources).md`
    - `02_architecture_principles.md`
    - `03_logical_architecture.md`
    - `04_service_architecture.md`
    - `05_data_architecture.md`
    - `06_deployment_architecture.md`
    - `07_integration_architecture.md`
    - `08_security_architecture.md`
    - `diagrams/`
      - `context.mmd`
      - `containers.mmd`
      - `deployment.mmd`

</details>

<details open>
<summary><strong>Целевой раздел: <code>03_platform/</code></strong></summary>

- **`docs-new/`**
  - **`03_platform/`**
    - `00_platform_core.md/`
      - `03_platform_core.md (01 sources).md`
    - `01_tenant_and_security/`
      - `01_overview.md`
      - `02_tenant_model.md`
      - `03_iam_model.md`
      - `04_authorization_contracts.md`
    - **`02_configuration_platform/`**
      - **`01_overview.md` ← перенесен этот файл**
      - `02_domain_model.md`
      - `03_data_configuration.md`
      - **`03_data_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `04_workflow_configuration.md`
      - `05_rule_configuration.md`
      - `06_condition_model.md`
      - **`06_condition_model/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `07_ui_configuration.md`
      - `08_reporting_configuration.md`
      - `09_output_configuration.md`
      - **`09_output_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `10_versioning_and_publish.md`
      - **`10_versioning_and_publish/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `11_import_export.md`
    - `03_workflow_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_action_execution.md`
      - `04_history.md`
    - `04_rule_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_result_model.md`
    - `05_value_set_data/`
      - `01_overview.md`
      - `02_value_set_model.md`
      - `03_value_items.md`
      - `04_tenant_override.md`
    - `06_integration/`
      - `01_overview.md`
      - `02_external_systems.md`
      - `03_connectors.md`
      - `04_integration_flows.md`
      - `05_external_id_mapping.md`
    - `07_event_foundation/`
      - `01_overview.md`
      - `02_event_envelope.md`
      - `03_outbox_inbox.md`
      - `04_idempotency.md`
    - `08_audit_history/`
      - `01_overview.md`
      - `02_audit_model.md`
      - `03_history_model.md`
      - `04_traceability.md`

    - `10_object_runtime/`
      - `01_overview.md`
      - `02_object_contract.md`
      - `03_change_execution.md`
      - `04_logic_and_validation_control.md`
      - `05_module_integration.md`
      - `06_current_implementation_gap.md`
      - `07_create_from_existing.md`
    - `11_numbering/`
      - `01_overview.md`
      - `02_design_decisions.md`
      - `03_model.md`
      - `04_runtime.md`
      - `05_object_runtime_integration.md`
      - `06_security.md`

</details>

<details>
<summary><code>04_domain_modules/</code></summary>

- **`docs-new/`**
  - `04_domain_modules/`
    - `01_module_index.md`
    - `00_module_documentation_template.md`
    - `00_common/`
      - `00_module_overview.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `05_rules.md`
      - `90_traceability_pr00.md`
    - `01_general_master_data/`
      - `00_module_overview.md`
      - `01_scope.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `04_workflows.md`
      - `05_rules.md`
      - `06_ui_views.md`
      - `07_reports_outputs.md`
      - `08_api_contracts.md`
      - `09_events.md`
      - `10_value_set_data_usage.md`
      - `11_permissions.md`
      - `12_audit_history.md`
      - `13_operations.md`
      - `14_test_strategy.md`
      - `backlog.md`
    - `02_product_process_definition/`
    - `03_plant_structure/`
    - `04_resource_management/`
    - `05_document_management/`
    - `06_project_management/`
    - `07_order_management/`
    - `08_planning_scheduling/`
    - `09_production_logistics/`
    - `10_shopfloor_execution/`
    - `11_quality_management/`
    - `12_machine_data_collection/`
    - `13_manufacturing_analytics/`

</details>

<details>
<summary><code>Исключённый раздел контрактов</code></summary>

Раздел `05_contracts/` исключён из целевой структуры. Старые файлы из `docs/05 contracts` остаются только исходниками сверки; в `docs-new` для них не создаётся ни отдельный раздел, ни новые дочерние папки у владельцев.

</details>

<details>
<summary><code>07_operations/</code></summary>

`07_operations/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной эксплуатационной документации: окружения, deployment, backup/restore, monitoring, incident response или platform-wide runbooks. Эксплуатация отдельных platform areas остаётся в `03_platform/*/08_operations.md`.

</details>

<details>
<summary><code>08_testing/</code></summary>

`08_testing/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной QA/test strategy, общих acceptance criteria, contract/integration/E2E/security/performance подходов или test data strategy. Проверки отдельных platform areas остаются в `03_platform/*/07_quality.md`, `90_traceability.md` и документах модулей.

</details>

<details>
<summary><code>09_decisions/</code></summary>

`09_decisions/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только как будущий механизм для отдельного утверждённого ADR; draft-копии из `docs/adr/drafts` остаются вне `docs-new`.

</details>

<details>
<summary><code>10_backlog/</code></summary>

- **`docs-new/`**
  - `10_backlog/`
    - `epics/`
      - `epic-configuration-platform-mvp.md`
      - `epic-workflow-foundation.md`
      - `epic-value-set-data-mvp.md`
    - `requirements/`
      - `req-configuration-publish.md`
      - `req-effective-ui-resolution.md`
    - `roadmap/`
      - `roadmap-mvp.md`
      - `roadmap-platform-foundation.md`

</details>

<details>
<summary><code>11_glossary/</code></summary>

- **`docs-new/`**
  - `11_glossary/`
    - `glossary.md`
    - `platform_terms.md`
    - `manufacturing_terms.md`
    - `ui_terms.md`
    - `security_terms.md`

</details>

<details>
<summary><code>12_appendices/</code></summary>

- **`docs-new/`**
  - `12_appendices/`
    - `imported_materials/`
    - `legacy_docs/`
    - `meeting_notes/`
    - `research/`
    - `vendor_references/`

</details>


</details>

<details>
<summary><strong>Карточка переноса:</strong> <code>docs/01 sources/06.3.0_configuration_platform_data_model.md</code> → <code>docs-new/03_platform/02_configuration_platform/offers/06.3.0_configuration_platform_data_model.md (01 sources).md</code> · разделить по блокам</summary>

Актуальный маршрут содержания: [трассировка Configuration](../03_platform/02_configuration/90_traceability.md).

#### Паспорт переноса

| Поле | Значение |
| --- | --- |
| Исходный файл | `docs/01 sources/06.3.0_configuration_platform_data_model.md` |
| Целевой каталог | `03_platform/02_configuration_platform/offers/` |
| Тип документа по `00` | Platform design / cross-cutting source for data configuration, conditions, output, versioning and import/export |
| Решение | Source-копия для разборки с последующим разделением по целевым документам |
| Статус этапов | Копия исходника в offer-папку Configuration Platform — готово; раскладка разделов — подготовлена; финальное структурирование целевых документов — следующий отдельный шаг; термины — следующий отдельный шаг; связи — следующий отдельный шаг. |
| Почему так | Исходник не является только domain/data model: внутри есть binding источников значений, localization, runtime resolution, validation before publish, publication records, OriginKey и import identity. Эти части должны лечь в разные целевые блоки Configuration Platform. |
| Что важно не потерять | ConfigurationScope, ConfigurationVersion, ConfigurationKind, ConfigurationEntry, ConfigurationProperty, localization, publication record, change log, binding к источнику значений, runtime resolution, validation before publish, OriginKey и import identity. |

#### Краткое описание

Документ описывает логическую модель хранения Configuration Platform и одновременно содержит фрагменты будущих документов по data configuration, condition model, output, versioning/publish и import/export. Поэтому файл сохранен как source-источник в offer-папке Configuration Platform, а финальное содержание нужно переносить частями.

#### Разделы исходного документа

| Исходный раздел | Краткое пояснение |
| --- | --- |
| `1. Назначение` | Определяет документ как логическую модель хранения Configuration Platform и отделяет ее от исполняемой модели стандартного объекта. |
| `2. Базовые сущности` | Описывает основные сущности модели: scope, version, kind, entry, property, localization, publication record и change log. |
| `3. Специализированные таблицы` | Показывает, когда поверх базовой entry/property-модели допустимы типизированные таблицы. |
| `4. Привязка поля к источнику значений` | Описывает binding поля к ValueSet, external reference или enum-like источнику. |
| `5. Локализация` | Фиксирует модель локализованных значений и правила получения invariant/localized value. |
| `6. Получение итоговой конфигурации во время работы системы` | Описывает runtime-алгоритм ResolveEffective и применение опубликованных версий. |
| `7. Валидация перед публикацией` | Перечисляет проверки схемы, идентичности, связей, совместимости, binding и запрета обхода Object Mutation Pipeline. |
| `8. Связанные документы` | Указывает документы, с которыми должна синхронизироваться модель данных Configuration Platform. |

#### Раскладка разделов по целевым блокам

| Раздел исходника | Целевой блок `00` | Почему переносится туда | Что проверить при финальном переносе |
| --- | --- | --- | --- |
| `1. Назначение` | `03_data_configuration.md`; вводный контекст для всех offer-копий | Раздел задает границу: Configuration Platform хранит опубликованные оболочки, настройки и переопределения поверх Object Runtime, а не исполняемую модель бизнес-объекта. | Не превращать этот вводный блок в повтор общего обзора `01_overview.md`; оставить только то, что нужно для конкретного целевого документа. |
| `2. Базовые сущности` / `ConfigurationScope`, `ConfigurationEntry`, `ConfigurationProperty`, `ConfigurationPropertyLocalization`, `ConfigurationChange` | `03_data_configuration.md` | Это базовая модель данных конфигурации: scope, entry/property, локализация и журнал изменений конфигуратора. | Отделить логическую модель конфигурации от требований физического хранения и индексации. |
| `2. Базовые сущности` / `ConfigurationVersion`, `PublicationRecord`, `ParentBaseVersionId`, `PreviousVersionId` | `10_versioning_and_publish.md` | Эти сущности описывают версию, публикацию, историю внутри scope и связь с родительской опубликованной базой. | Не смешивать историю версии внутри scope и воспроизводимую runtime-цепочку parent base. |
| `2. Базовые сущности` / `ConfigurationKind.Category = Output` и `OverrideMode` | `09_output_configuration.md` | Output указан как отдельная категория конфигурационного артефакта, для которой могут применяться свои правила override. | Переносить только общую модель артефакта output; delivery/report-specific правила брать из профильных output/report документов. |
| `2. Базовые сущности` / `OriginKey`, `ParentOriginKey`, `OverridesEntryId` | `11_import_export.md`; также связь с `03_data_configuration.md` | `OriginKey` используется для импорта, сравнения, переноса на новую родительскую базу и отслеживания источника. | Зафиксировать стабильность identity и не строить публичную идентичность из CLR `FullName`. |
| `3. Специализированные таблицы` | `03_data_configuration.md`; `06_condition_model.md`; `09_output_configuration.md` | Раздел показывает, когда базовая entry/property-модель расширяется типизированными таблицами для data, workflow/rule/UI/report/output. | Для каждого целевого блока оставить только релевантные типизированные таблицы; не переносить весь список во все документы. |
| `4. Привязка поля к источнику значений` | `03_data_configuration.md`; `06_condition_model.md` | Binding источника значений является metadata поля; `SelectionFilter` должен быть metadata/expression/contract, а не произвольным кодом. | Развести правила data field binding и правила conditions/filters; спорные expression-формы вынести в вопросы. |
| `5. Локализация` | `03_data_configuration.md`; при необходимости `09_output_configuration.md` | Локализованные значения свойства относятся к модели данных конфигурации; output может использовать localization для человекочитаемых форм. | Не делать язык отдельным scope; сохранить правило invariant value. |
| `6. Получение итоговой конфигурации во время работы системы` | `10_versioning_and_publish.md`; связь с `03_data_configuration.md` | Runtime использует только опубликованные версии и точную parent base chain. | Проверить, что runtime resolution не дублирует отдельные runtime-документы и не допускает draft configuration. |
| `7. Валидация перед публикацией` | `10_versioning_and_publish.md`; `03_data_configuration.md`; `06_condition_model.md`; `11_import_export.md` | Проверки относятся к публикации, но затрагивают schema, identity, binding, localization, registry contract codes и запрет обхода Object Mutation Pipeline. | Разнести проверки по владельцам: publish gate, data schema, condition/filter validation, import identity. |
| `8. Связанные документы` | Миграционный реестр и блок связей целевых документов | Раздел нужен как источник связей: Configuration Platform overview, scope/version pipeline, runtime data flow, ADR-017. | После финального переноса обновить связанные документы в каждом целевом `.md`. |

#### Структура из `00`

Сначала показан короткий маршрут переноса, ниже — полная карта `00` с раскрываемыми крупными разделами. В дереве жирным выделен весь путь до файла.

##### Маршрут переноса

| Уровень | Узел |
| --- | --- |
| Корень | **`docs-new/`** |
| Раздел | **`03_platform/`** |
| Подраздел | **`02_configuration_platform/`** |
| Каталог source-копии | **`03_platform/02_configuration_platform/offers/`** |
| Целевые разделы для будущей разборки | **`03_data_configuration.md`**; **`06_condition_model.md`**; **`09_output_configuration.md`**; **`10_versioning_and_publish.md`**; **`11_import_export.md`** |
| Файл-источник | **`06.3.0_configuration_platform_data_model.md`** |

##### Полная карта структуры `00`

<details>
<summary><code>00_governance/</code></summary>

- **`docs-new/`**
  - `00_governance/`
    - `00_documentation_strategy.md`
    - `01_document_types.md`
    - `02_document_lifecycle.md`
    - `03_review_and_approval.md`
    - `04_ai_usage_policy.md`
    - `05_naming_conventions.md`
    - `06_traceability_rules.md`
    - `requirements/`
      - `00_index.md`
      - `001_dmp_functional_requirements_and_constraints.md`

</details>

<details>
<summary><code>01_concept/</code></summary>

- **`docs-new/`**
  - `01_concept/`
    - `01_product_concept.md/`
      - `01_concept.md (01 sources).md`
    - `02_system_scope.md/`
      - `02_system_scope.md (01 sources).md`
    - `03_business_goals.md`
    - `04_stakeholders.md`
    - `05_success_metrics.md`

</details>

<details>
<summary><code>02_architecture/</code></summary>

- **`docs-new/`**
  - `02_architecture/`
    - `01_architecture_overview.md/`
      - `05_architecture.md (01 sources).md`
    - `02_architecture_principles.md`
    - `03_logical_architecture.md`
    - `04_service_architecture.md`
    - `05_data_architecture.md`
    - `06_deployment_architecture.md`
    - `07_integration_architecture.md`
    - `08_security_architecture.md`
    - `diagrams/`
      - `context.mmd`
      - `containers.mmd`
      - `deployment.mmd`

</details>

<details open>
<summary><strong>Целевой раздел: <code>03_platform/</code></strong></summary>

- **`docs-new/`**
  - **`03_platform/`**
    - `00_platform_core.md/`
      - `03_platform_core.md (01 sources).md`
    - `01_tenant_and_security/`
      - `01_overview.md`
      - `02_tenant_model.md`
      - `03_iam_model.md`
      - `04_authorization_contracts.md`
    - **`02_configuration_platform/`**
      - `01_overview.md`
      - `02_domain_model.md`
      - `03_data_configuration.md`
      - **`03_data_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `04_workflow_configuration.md`
      - `05_rule_configuration.md`
      - `06_condition_model.md`
      - **`06_condition_model/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `07_ui_configuration.md`
      - `08_reporting_configuration.md`
      - `09_output_configuration.md`
      - **`09_output_configuration/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `10_versioning_and_publish.md`
      - **`10_versioning_and_publish/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
      - `11_import_export.md`
      - **`11_import_export/`**
        - **`offers/`**
          - **`06.3.0_configuration_platform_data_model.md`**
    - `03_workflow_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_action_execution.md`
      - `04_history.md`
    - `04_rule_engine/`
      - `01_overview.md`
      - `02_runtime_contract.md`
      - `03_result_model.md`
    - `05_value_set_data/`
      - `01_overview.md`
      - `02_value_set_model.md`
      - `03_value_items.md`
      - `04_tenant_override.md`
    - `06_integration/`
      - `01_overview.md`
      - `02_external_systems.md`
      - `03_connectors.md`
      - `04_integration_flows.md`
      - `05_external_id_mapping.md`
    - `07_event_foundation/`
      - `01_overview.md`
      - `02_event_envelope.md`
      - `03_outbox_inbox.md`
      - `04_idempotency.md`
    - `08_audit_history/`
      - `01_overview.md`
      - `02_audit_model.md`
      - `03_history_model.md`
      - `04_traceability.md`

    - `10_object_runtime/`
      - `01_overview.md`
      - `02_object_contract.md`
      - `03_change_execution.md`
      - `04_logic_and_validation_control.md`
      - `05_module_integration.md`
      - `06_current_implementation_gap.md`
      - `07_create_from_existing.md`
    - `11_numbering/`
      - `01_overview.md`
      - `02_design_decisions.md`
      - `03_model.md`
      - `04_runtime.md`
      - `05_object_runtime_integration.md`
      - `06_security.md`

</details>

<details>
<summary><code>04_domain_modules/</code></summary>

- **`docs-new/`**
  - `04_domain_modules/`
    - `01_module_index.md`
    - `00_module_documentation_template.md`
    - `00_common/`
      - `00_module_overview.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `05_rules.md`
      - `90_traceability_pr00.md`
    - `01_general_master_data/`
      - `00_module_overview.md`
      - `01_scope.md`
      - `02_domain_model.md`
      - `03_object_runtime_model.md`
      - `04_workflows.md`
      - `05_rules.md`
      - `06_ui_views.md`
      - `07_reports_outputs.md`
      - `08_api_contracts.md`
      - `09_events.md`
      - `10_value_set_data_usage.md`
      - `11_permissions.md`
      - `12_audit_history.md`
      - `13_operations.md`
      - `14_test_strategy.md`
      - `backlog.md`
    - `02_product_process_definition/`
    - `03_plant_structure/`
    - `04_resource_management/`
    - `05_document_management/`
    - `06_project_management/`
    - `07_order_management/`
    - `08_planning_scheduling/`
    - `09_production_logistics/`
    - `10_shopfloor_execution/`
    - `11_quality_management/`
    - `12_machine_data_collection/`
    - `13_manufacturing_analytics/`

</details>

<details>
<summary><code>Исключённый раздел контрактов</code></summary>

Раздел `05_contracts/` исключён из целевой структуры. Старые файлы из `docs/05 contracts` остаются только исходниками сверки; в `docs-new` для них не создаётся ни отдельный раздел, ни новые дочерние папки у владельцев.

</details>

<details>
<summary><code>07_operations/</code></summary>

`07_operations/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной эксплуатационной документации: окружения, deployment, backup/restore, monitoring, incident response или platform-wide runbooks. Эксплуатация отдельных platform areas остаётся в `03_platform/*/08_operations.md`.

</details>

<details>
<summary><code>08_testing/</code></summary>

`08_testing/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только при появлении сквозной QA/test strategy, общих acceptance criteria, contract/integration/E2E/security/performance подходов или test data strategy. Проверки отдельных platform areas остаются в `03_platform/*/07_quality.md`, `90_traceability.md` и документах модулей.

</details>

<details>
<summary><code>09_decisions/</code></summary>

`09_decisions/` не создаётся в текущем комплекте `docs-new`. Раздел допустим только как будущий механизм для отдельного утверждённого ADR; draft-копии из `docs/adr/drafts` остаются вне `docs-new`.

</details>

<details>
<summary><code>10_backlog/</code></summary>

- **`docs-new/`**
  - `10_backlog/`
    - `epics/`
      - `epic-configuration-platform-mvp.md`
      - `epic-workflow-foundation.md`
      - `epic-value-set-data-mvp.md`
    - `requirements/`
      - `req-configuration-publish.md`
      - `req-effective-ui-resolution.md`
    - `roadmap/`
      - `roadmap-mvp.md`
      - `roadmap-platform-foundation.md`

</details>

<details>
<summary><code>11_glossary/</code></summary>

- **`docs-new/`**
  - `11_glossary/`
    - `glossary.md`
    - `platform_terms.md`
    - `manufacturing_terms.md`
    - `ui_terms.md`
    - `security_terms.md`

</details>

<details>
<summary><code>12_appendices/</code></summary>

- **`docs-new/`**
  - `12_appendices/`
    - `imported_materials/`
    - `legacy_docs/`
    - `meeting_notes/`
    - `research/`
    - `vendor_references/`

</details>


</details>

### 5.3 Предложения по сопоставлению: `docs/02 requrements`

<details>
<summary><strong>Предложения по сопоставлению: `docs/02 requrements`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/02 requrements/!role_permition_refactor.md` | `03_platform/01_tenant_and_security/90_traceability.md`; backlog `PCDOC-13` | Разобрать без копии в Tenant/Security | Итоговое решение по рефакторингу ролей и разрешений. | Требование относится к Tenant and Security: Role, Permission, RolePermission, UserRoleAssignment и authorization scope. | Что уже реализовано, а что остается backlog? |
| `docs/02 requrements/01_admin_list_newfilters_config.md` | `03_platform/02_configuration_platform/requirements/01_admin_list_newfilters_config.md (02 requrements).md` | Requirement-копия для разборки | Требования к спискам Admin и новым фильтрам. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Какая часть относится к общей UI Configuration, а какая к конкретному Admin-приложению? |
| `docs/02 requrements/01_roles.md` | `03_platform/01_tenant_and_security/90_traceability.md`; backlog `PCDOC-13` | Разобрать без копии в Tenant/Security | Требования к ролям. | Требование относится к IAM-модели Tenant and Security; финальная сверка с контрактами выполняется отдельным шагом. | Есть ли утвержденная матрица ролей? |
| `docs/02 requrements/01_roles_localization.md` | `03_platform/01_tenant_and_security/06_user_experience.md`; `11_glossary`; backlog `PCDOC-13` | Разобрать без копии в Tenant/Security | Локализация ролей. | Требование относится к IAM/security, а связи с UI и глоссарием будут отражены при финальном переносе. | Где хранится источник локализованных названий ролей? |
| `docs/02 requrements/01_roles_rules.md` | `03_platform/01_tenant_and_security/06_user_experience.md`; `12_frontend_platform`; backlog `PCDOC-13` | Разобрать без копии в Tenant/Security | Правила принятия решений для UI по ролям. | Требование относится к authorization behavior для ролей; UI-поведение и контракт доступа нужно разделить при финальном переносе. | UI сам решает доступность или получает готовые effective permissions? |
| `docs/02 requrements/01_tenants_rules.md` | `03_platform/01_tenant_and_security/06_user_experience.md`; `12_frontend_platform`; backlog `PCDOC-13` | Разобрать без копии в Tenant/Security | Правила tenant UI. | Требование относится к Admin-сценариям Tenant/Security; общие UI-правила должны перейти к Frontend Platform. | Используем термин tenant или предприятие с техническим alias? |
| `docs/02 requrements/01_users_rules.md` | `03_platform/01_tenant_and_security/06_user_experience.md`; `12_frontend_platform`; backlog `PCDOC-13` | Разобрать без копии в Tenant/Security | Правила User UI. | Требование относится к Admin-сценариям Tenant/Security; membership-часть зависит от открытого решения `TS-DEC-02`. | Нужна ли отдельная страница `users` в IAM-документации? |
| `docs/02 requrements/02_ui.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/02_ui.md (02 requrements).md` | Requirement-копия для разборки | Общие UI-требования. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Что является нормативным UI-контрактом, а что макетом/черновиком? |
| `docs/02 requrements/03_01_runtime_engine_for_admin_config.md` | `03_platform/12_frontend_platform/90_traceability.md` | Разобрать без копии в `06_runtime` | Общий runtime engine для Admin и Configurator. | Подтверждённые сведения уже отражены во Frontend Platform; будущие шаблоны и компоненты ведутся как `FE-DEC-*`. | Где граница frontend runtime и платформенного Object Runtime? |
| `docs/02 requrements/03_02_shell_configurator_vs_admin.md` | `03_platform/02_configuration_platform/requirements/03_02_shell_configurator_vs_admin.md (02 requrements).md` | Requirement-копия для разборки | Shell и навигация Configurator vs Admin. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Один shell или две разные модели приложений? |
| `docs/02 requrements/03_03_page_configuration_explorer_ui.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_03_page_configuration_explorer_ui.md (02 requrements).md` | Requirement-копия для разборки | UI Configuration Explorer, сценарии и макеты. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Что уже нормативно, а что является макетом? |
| `docs/02 requrements/03_04_page_data_inspector_ui.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_04_page_data_inspector_ui.md (02 requrements).md` | Requirement-копия для разборки | Data Inspector UI. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Нужен ли Data Inspector как отдельный артефакт? |
| `docs/02 requrements/03_05_property_editor_ui.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_05_property_editor_ui.md (02 requrements).md` | Requirement-копия для разборки | Универсальный Property Editor. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Какие свойства редактируются через схему, а какие через custom editor? |
| `docs/02 requrements/03_06_page_workflow_editor_ui_v2.md` | `03_platform/02_configuration_platform/04_workflow_configuration/requirements/03_06_page_workflow_editor_ui_v2.md (02 requrements).md` | Requirement-копия для разборки | Workflow Editor UI. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Что является workflow contract, а что экраном редактора? |
| `docs/02 requrements/03_07_page_view_editor_ui.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_07_page_view_editor_ui.md (02 requrements).md` | Requirement-копия для разборки | View Editor UI. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Какие элементы view являются stable contract? |
| `docs/02 requrements/03_08_page_action_model_and_action_ux_v2.md` | `03_platform/02_configuration_platform/04_workflow_configuration/requirements/03_08_page_action_model_and_action_ux_v2.md (02 requrements).md` | Requirement-копия для разборки | Action model и UX действий. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Как разделить `ActionCode`, команду lifecycle и UI action? |
| `docs/02 requrements/03_09_page_rule_editor_ui.md` | `03_platform/02_configuration_platform/05_rule_configuration/requirements/03_09_page_rule_editor_ui.md (02 requrements).md` | Requirement-копия для разборки | Rule Editor UI и модель правил. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Какие правила конфигурируемые, а какие доменные инварианты? |
| `docs/02 requrements/03_10_page_navigation_editor_ui.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_10_page_navigation_editor_ui.md (02 requrements).md` | Requirement-копия для разборки | Navigation/Menu Editor UI. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Навигация является отдельным artifact type или частью view/menu configuration? |
| `docs/02 requrements/03_11_page_workplace_editor_ui.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_11_page_workplace_editor_ui.md (02 requrements).md` | Requirement-копия для разборки | Workplace Editor UI. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Нужно ли заводить отдельный artifact `Workplace`? |
| `docs/02 requrements/03_12_page_value_set_editor_ui.md` | `03_platform/05_value_set_data/requirements/03_12_page_value_set_editor_ui.md (02 requrements).md` | Requirement-копия для разборки | Value Set Editor UI. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Где граница ValueSet и ValueSetData? |
| `docs/02 requrements/03_12_1_value_set_data_editor_design.md` | `03_platform/05_value_set_data/requirements/03_12_1_value_set_data_editor_design.md (02 requrements).md` | Requirement-копия для разборки | Value Set Data Editor. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Должен ли редактор быть частью платформы ValueSetData или UI Configuration? |
| `docs/02 requrements/03_12_admin_runtime_refactor_plan.md` | `03_platform/12_frontend_platform/90_traceability.md` | Разобрать без копии в `06_runtime` | План frontend-refactor для общего runtime Admin и Configurator. | Подтверждённые сведения уже отражены во Frontend Platform; оставшиеся пункты являются frontend backlog/traceability, а не отдельным runtime-пакетом. | Какие пункты уже выполнены? |
| `docs/02 requrements/03_13_page_context_version_management_ui_v4.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_13_page_context_version_management_ui_v4.md (02 requrements).md` | Requirement-копия для разборки | UI управления контекстом и версиями. | Исходник помещен в UI configuration requirements; правила версий/публикации позже нужно сверить с lifecycle configuration version. | Что является частью lifecycle configuration version? |
| `docs/02 requrements/03_14_objectlist_bounded_runtime_layout_model.md` | `03_platform/10_object_runtime/requirements/03_14_objectlist_bounded_runtime_layout_model.md (02 requrements).md` | Requirement-копия для разборки | Bounded runtime layout model для object list. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Runtime list — часть Object Runtime или UI Configuration? |
| `docs/02 requrements/03_15_runtime_list_query_filters_requirements.md` | `03_platform/10_object_runtime/requirements/03_15_runtime_list_query_filters_requirements.md (02 requrements).md` | Requirement-копия для разборки | Runtime list query, filters and presets. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Где хранить filter operators: runtime contract или UI preset? |
| `docs/02 requrements/03_config_app_ui_req_02_v4.md` | `03_platform/02_configuration_platform/07_ui_configuration.md/requirements/03_config_app_ui_req_02_v4.md (02 requrements).md` | Requirement-копия для разборки | UI Studio Model для конфигуратора. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Какие части стали текущим целевым решением? |
| `docs/02 requrements/04_artifact_editor_exploer_diff_promts.md` | `03_platform/02_configuration_platform/requirements/04_artifact_editor_exploer_diff_promts.md (02 requrements).md` | Requirement-копия для разборки | Prompts/rules для artifact editor explorer diff. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Нужно ли хранить AI prompts в governance или убрать в приложения? |
| `docs/02 requrements/04_reg_and_scenario_config_product_definition.md` | `03_platform/02_configuration_platform/requirements/04_reg_and_scenario_config_product_definition.md (02 requrements).md` | Requirement-копия для разборки | Сценарий конфигурирования ProductDefinition. | Файл перенесен в requirements Configuration Platform как материал для разбора рядом с `02_domain_model.md`; модульные примеры и платформенные правила будут разделяться при финальной структуризации. | Что является модульным требованием, а что платформенным примером? |
| `docs/02 requrements/05_configuration_studio_end_to_end_scenarios_corporate_tenant_lifecycle.md` | `03_platform/02_configuration_platform/requirements/05_configuration_studio_end_to_end_scenarios_corporate_tenant_lifecycle.md (02 requrements).md` | Requirement-копия для разборки | Сквозные сценарии Configuration Studio по corporate/tenant lifecycle. | Файл помещен в целевую папку requirements по основному смысловому владельцу; связи с другими разделами будут уточняться при финальной структуризации. | Перевести в testing или оставить как backlog требований? |
| `docs/02 requrements/06_wf_runtime_ui_requirements_and_flows_revised_list_ui.md` | `03_platform/04_workflow/03_contracts.md`, `04_runtime.md`, `01_scope.md`, `90_traceability.md`; UI renderer, layout и пользовательские flow остаются исходными требованиями для будущего владельца Frontend Platform | Содержательные серверные части разобраны в каноническом Workflow-пакете; переходная копия удалена | WF Runtime UI, требования и пользовательские flow | Исходник сохранён в `docs/02 requrements`; backend-сведения отражены в Workflow, UI-часть не выдана за текущую гарантию и ожидает владельца Frontend Platform | Отдельный runtime UI contract в Workflow не создаётся; серверные DTO и границы описываются в Workflow, renderer и layout — у Frontend Platform |

</details>

### 5.4 Предложения по сопоставлению: `docs/03 conf_artefacts`

<details>
<summary><strong>Предложения по сопоставлению: `docs/03 conf_artefacts`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/03 conf_artefacts/Action.md` | `03_platform/02_configuration_platform/offers/Action.md (03 conf_artefacts).md` | Source-копия для разборки | Артефакт Action. | Исходник хранится в Configuration Platform; при финальном переносе нужно выделить workflow/action configuration и schema/contract часть. | Связь `Action` с `CommandCode` и `ActionCode` требует решения. |
| `docs/03 conf_artefacts/Artefact_classification.md` | `03_platform/02_configuration_platform/offers/Artefact_classification.md (03 conf_artefacts).md` | Source-копия для разборки | Классификация конфигурационных моделей. | Исходник хранится в Configuration Platform как основа для обзора классификации артефактов. | Нужно ли закрепить типы как stable codes? |
| `docs/03 conf_artefacts/Configurator_ui_structure.md` | `03_platform/02_configuration_platform/offers/Configurator_ui_structure.md (03 conf_artefacts).md` | Source-копия для разборки | Целевая структура UI конфигуратора. | Исходник хранится в Configuration Platform; позже нужно отделить UI configuration от shell/navigation. | Это нормативная структура или макет приложения? |
| `docs/03 conf_artefacts/EnumPresentation.md` | `03_platform/02_configuration_platform/offers/EnumPresentation.md (03 conf_artefacts).md` | Source-копия для разборки | Представление enum. | Исходник хранится в Configuration Platform; машинную модель и текстовое описание нужно будет разделить. | Связь с SystemEnum и ValueSet. |
| `docs/03 conf_artefacts/GridUi.md` | `03_platform/02_configuration_platform/offers/GridUi.md (03 conf_artefacts).md` | Source-копия для разборки | Grid UI artifact. | Исходник хранится в Configuration Platform как материал для UI configuration и schema contract. | Где граница `GridUi` и `View`? |
| `docs/03 conf_artefacts/Lookup.md` | `03_platform/02_configuration_platform/offers/Lookup.md (03 conf_artefacts).md` | Source-копия для разборки | Lookup artifact. | Исходник хранится в Configuration Platform; позже нужно решить границу runtime-ссылок и UI-выбора. | Lookup является UI artifact или runtime contract? |
| `docs/03 conf_artefacts/Navigation.md` | `03_platform/02_configuration_platform/offers/Navigation.md (03 conf_artefacts).md` | Source-копия для разборки | Navigation artifact. | Исходник хранится в Configuration Platform как материал для UI/navigation configuration. | Нужен ли отдельный menu/navigation contract? |
| `docs/03 conf_artefacts/NumberingRule.md` | `03_platform/02_configuration_platform/offers/NumberingRule.md (03 conf_artefacts).md` | Source-копия для разборки | NumberingRule artifact. | Исходник хранится в Configuration Platform, но финальный владелец может быть платформенный блок Numbering. | Синхронизировать с текущими docs-new Numbering. |
| `docs/03 conf_artefacts/ObjectNavigationPolicy.md` | `03_platform/02_configuration_platform/offers/ObjectNavigationPolicy.md (03 conf_artefacts).md` | Source-копия для разборки | Политика навигации объекта. | Исходник хранится в Configuration Platform; позже нужно разделить Object Runtime и UI navigation. | Где граница object-level navigation и menu navigation? |
| `docs/03 conf_artefacts/ObjectType.md` | `03_platform/02_configuration_platform/offers/ObjectType.md (03 conf_artefacts).md` | Source-копия для разборки | ObjectType artifact. | Исходник хранится в Configuration Platform, но финальная связь идет с Object Runtime и кодами типов объектов. | Проверить связь с module docs `03_object_runtime_model.md`. |
| `docs/03 conf_artefacts/Output.md` | `03_platform/02_configuration_platform/offers/Output.md (03 conf_artefacts).md` | Source-копия для разборки | Output artifact. | Исходник хранится в Configuration Platform как материал для output configuration и schema contract. | Разделить Output и Report. |
| `docs/03 conf_artefacts/Report.md` | `03_platform/02_configuration_platform/offers/Report.md (03 conf_artefacts).md` | Source-копия для разборки | Report artifact. | Исходник хранится в Configuration Platform как материал для reporting configuration и schema contract. | Связь Report с dataset и BIRT. |
| `docs/03 conf_artefacts/Rules.md` | `03_platform/02_configuration_platform/offers/Rules.md (03 conf_artefacts).md` | Source-копия для разборки | Rules artifact. | Исходник хранится в Configuration Platform; позже нужно разделить конфигурацию правил и исполнение Rule Engine. | Таксономия правил требует утверждения. |
| `docs/03 conf_artefacts/SystemEnum.md` | `03_platform/02_configuration_platform/offers/SystemEnum.md (03 conf_artefacts).md` | Source-копия для разборки | SystemEnum artifact. | Исходник хранится в Configuration Platform; позже нужно связать с ValueSetData и статическими кодами. | SystemEnum остается отдельным механизмом или видом ValueSet? |
| `docs/03 conf_artefacts/ValueSet.md` | `03_platform/02_configuration_platform/offers/ValueSet.md (03 conf_artefacts).md` | Source-копия для разборки | ValueSet artifact. | Исходник хранится в Configuration Platform; финально должен быть связан с ValueSetData. | Нет. |
| `docs/03 conf_artefacts/View.md` | `03_platform/02_configuration_platform/offers/View.md (03 conf_artefacts).md` | Source-копия для разборки | View artifact. | Исходник хранится в Configuration Platform как материал для UI configuration и schema contract. | Разделить view, grid и form sections. |
| `docs/03 conf_artefacts/Workflow.md` | `03_platform/02_configuration_platform/offers/Workflow.md (03 conf_artefacts).md` | Source-копия для разборки | Workflow artifact. | Исходник хранится в Configuration Platform; финально должен быть связан с workflow configuration и Workflow Engine runtime. | Связать с Workflow Engine runtime. |
| `docs/03 conf_artefacts/artifact_property_regrouping.md` | `03_platform/02_configuration_platform/offers/artifact_property_regrouping.md (03 conf_artefacts).md` | Source-копия для разборки | Перегруппировка свойств артефактов. | Исходник хранится в Configuration Platform как рабочий материал для анализа модели артефактов. | Что уже принято, а что осталось экспериментом? |

</details>

### 5.5 Предложения по сопоставлению: `docs/04 runtime`

<details>
<summary><strong>Предложения по сопоставлению: `docs/04 runtime`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/04 runtime/configuration_scope_version_pipeline.md` | `03_platform/02_configuration/04_runtime.md`, `07_quality.md`, `08_operations.md`, `90_traceability.md` | Разобрать без копии в `06_runtime` | Pipeline scope/version для конфигурации. | Подтверждённая часть принадлежит Configuration; production-гарантии кэша, rebase/merge и observability остаются у Configuration quality/operations или backlog. | Где хранить rebase/merge pipeline? |
| `docs/04 runtime/runtime_data_flow.md` | `03_platform/03_object_runtime`, `03_platform/02_configuration`, `03_platform/12_frontend_platform` | Разобрать без копии в `06_runtime` | Runtime data flow. | Содержание разделяется между владельцами: effective configuration — Configuration, object operations — Object Runtime, frontend rendering — Frontend Platform. | Что является общим pipeline, а что Object Runtime? |
| `docs/04 runtime/runtime_settings_authoring_guide.md` | `03_platform/07_settings`, `03_platform/07_settings/90_traceability.md` | Разобрать без копии в `06_runtime` | Authoring guide runtime settings. | Подтверждённая часть покрыта Settings; неподтверждённые варианты остаются в `SET-DEC-*`. | Есть ли отдельный Settings Platform или это часть Configuration Platform? |

</details>

### 5.6 Предложения по сопоставлению: `docs/05 contracts`

<details>
<summary><strong>Предложения по сопоставлению: `docs/05 contracts`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/05 contracts/report_service/*` | Не переносить в `docs-new` как отдельный раздел | Source для сверки | Report Service OpenAPI/JSON Schema/XSD/XML/examples и API contract. | Нормативный смысл покрыт Reporting/Output; отдельный contract catalog не создаётся. | При изменении реализации сверять документы владельца и старые source-файлы. |
| `docs/05 contracts/rule_and_condition_model_platform_contract.md` | Не переносить в `docs-new` как отдельный документ | Source для сверки | Контракт rule and condition model. | Подтверждённые части уже у Rules и Configuration; неподтверждённые возможности не становятся MVP. | Где канонический источник condition model — решается владельцами Rules/Configuration. |
| `docs/05 contracts/tenant_security/*` | Не переносить в `docs-new` как отдельный раздел | Source для сверки | Security catalog, inventory и SQL snapshot. | Нормативный смысл покрыт Tenant/Security; отдельный contract catalog не создаётся. | Snapshot или источник истины — решает Tenant/Security. |
</details>

### 5.7 Предложения по сопоставлению: `docs/10 modules_req`

<details>
<summary><strong>Предложения по сопоставлению: `docs/10 modules_req`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/10 modules_req/01 ProductDefinition/nomenclature_architecture_decision_v2.md` | `04_domain_modules/02_product_process_definition/90_traceability_pr02.md`, `backlog.md` | Offer-копия удалена | Текущий пакет фиксирует `Nomenclature` как внешний объект `01_general_master_data`; оставшиеся будущие части маршрутизированы в backlog модуля 02. Исходник в `docs` сохранён. | Вопрос о владельце больше не используется как незакрытая граница текущего пакета. | Будущие части проверяются по `PPD-BL-009`. |
| `docs/10 modules_req/08 PlanningScheduling/planning_scheduling_architecture_revised.md` | `04_domain_modules/08_planning_scheduling/offers/planning_scheduling_architecture_revised.md (10 modules_req).md` | Module-копия для разборки | Архитектура модуля планирования и расписаний. | Исходник помещен в offer-зону Planning & Scheduling; финальные разделы модуля будут заполняться после анализа. | Нужна связь с заказами, ресурсами и производственной логистикой. |
| `docs/10 modules_req/08 PlanningScheduling/planning_scheduling_scenarios_outline.md` | `04_domain_modules/08_planning_scheduling/offers/planning_scheduling_scenarios_outline.md (10 modules_req).md` | Module-копия для разборки | Сценарии планирования и расписаний. | Исходник помещен в offer-зону Planning & Scheduling; сценарии нужно разложить по scope, workflow и operations. | Какие сценарии входят в MVP? |

</details>

### 5.8 Предложения по сопоставлению: `docs/10_backlog`

<details>
<summary><strong>Предложения по сопоставлению: `docs/10_backlog`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/10_backlog/report_birt_pipeline_backlog.md` | `03_platform/02_configuration_platform/08_reporting_configuration.md/report_birt_pipeline_backlog.md (10_backlog).md` | Backlog-копия для разборки | Бэклог Report/Output и BIRT Report Service. | Исходник помещен в узел reporting configuration как материал для последующей сверки backlog с design-документом. | Синхронизировать с reporting/output documents. |
| `docs/10_backlog/roadmap/preparation/00_platform_completion_estimate.md` | `10_backlog/roadmap/preparation/00_platform_completion_estimate.md` | Backlog | Оценка готовности платформы. | Подготовительный материал roadmap. | Обновить после свежей оценки состояния кода. |
| `docs/10_backlog/roadmap/preparation/01_roadmap_preparation_summary.md` | `10_backlog/roadmap/preparation/01_roadmap_preparation_summary.md` | Backlog | Итог предварительной подготовки roadmap. | Roadmap preparation. | Проверить актуальность после реструктуризации docs-new. |
| `docs/10_backlog/roadmap/preparation/02_platform_assessment.md` | `10_backlog/roadmap/preparation/02_platform_assessment.md` | Backlog | Оценка состояния платформенного ядра. | Поддерживает planning/backlog. | Нужна новая оценка после plant→site изменений. |
| `docs/10_backlog/roadmap/preparation/03_roadmap_draft.md` | `10_backlog/roadmap/roadmap-platform-foundation.md` | Backlog | Черновой roadmap. | Может стать основой целевого roadmap. | Что утверждено, а что черновик? |
| `docs/10_backlog/roadmap/preparation/04_candidate_module_map.md` | `04_domain_modules/01_module_index.md`; `10_backlog/roadmap/preparation/` | Спорно | Предварительная карта прикладных модулей. | Может конфликтовать с утвержденным списком 13 модулей из `00`. | Какие модули утверждены? |
| `docs/10_backlog/roadmap/preparation/05_estimation_model.md` | `10_backlog/roadmap/preparation/05_estimation_model.md` | Backlog | Методика оценки трудозатрат. | Это planning-документ. | Нужна ли методика в normative docs? |
| `docs/10_backlog/roadmap/preparation/06_open_decisions_and_risks.md` | `10_backlog/roadmap/preparation/06_open_decisions_and_risks.md` | Разделить | Открытые решения и риски. | Открытые решения вести в backlog и `90_traceability`; ADR создавать только по отдельному решению. Риски — backlog. | Какие open decisions уже закрыты? |
| `docs/10_backlog/roadmap/preparation/07_platform_gap_backlog.md` | `10_backlog/epics/platform-gap-backlog.md` | Backlog | Бэклог разрывов платформы. | План устранения gaps. | Сверить с текущим кодом и тестами. |
| `docs/10_backlog/roadmap/preparation/08_object_runtime_technology_alignment.md` | `03_platform/10_object_runtime/06_current_implementation_gap.md`; `10_backlog/epics/object-runtime-alignment.md` | Разделить | План-факт приведения документации к Object Runtime. | Содержит gap analysis и backlog. | Что уже перенесено в Object Runtime docs? |

</details>

<details>
<summary><strong>Карточка разборки:</strong> <code>docs/01 sources/07_tenant_and_security_platform.md</code> → будущая концепция и Tenant/Security decisions</summary>

Исходник остаётся в старом `docs`. Внутри Tenant/Security не создаётся папка с source-копиями; подтверждённое содержание переносится в документы-владельцы, а целевые идеи и расхождения ведутся через backlog `PCDOC-12`.

#### Паспорт разборки

| Поле | Значение |
| --- | --- |
| Исходный файл | `docs/01 sources/07_tenant_and_security_platform.md` |
| Source handling | Старый исходник используется как материал анализа; копия внутри Tenant/Security не создаётся |
| Решение | Разделить через concept, architecture, Tenant/Security decisions и backlog |
| Почему так | Исходник одновременно описывает общий security-контекст, tenant model, identity/IAM, authorization, isolation, связи с configuration, UI, audit и integration security. Это нельзя переносить одним финальным документом без потери границ ответственности. |
| Что важно не потерять | Hybrid Multi-Tenant, Tenant Context, IAM-сущности, permissions, runtime authorization contract, tenant isolation, связь с Configuration Platform, security/audit/integration rules и MVP scope. |

#### Раскладка разделов по целевым блокам

| Раздел исходника | Целевой блок `00` | Почему переносится туда | Что проверить при финальном переносе |
| --- | --- | --- | --- |
| `1. Назначение документа`; `2. Связь с другими документами`; `3. Основные принципы`; `15. Ключевая мысль`; `16. Что дальше` | `01_overview.md` | Эти разделы задают общий контекст Tenant and Security Platform и связи с платформой. | Не превращать overview в полный дубль tenant/IAM/authorization-моделей; оставить обзор и навигацию. |
| `4. Tenant Model`; `7. Tenant Isolation Model`; `9. Связь с Configuration Platform` / `9.1 Scope Model` | `02_tenant_model.md` | Здесь описаны tenant, site, tenant context, правила scope и изоляция данных. | Сохранить открытый вопрос по tenant/site/enterprise-иерархии и не переводить tenant-формы по частям. |
| `5. Identity Model`; `8. Security в Platform Services` / `8.1 Identity Service (IAM)`; `9.2 Roles для конфигурации` | `03_iam_model.md` | Эти разделы описывают пользователей, роли, binding пользователя к tenant, IAM и роли для Configuration Platform. | Проверить унификацию `Identity Service` → `IAM` и не смешивать роль IAM с UI visibility. |
| `6. Authorization Model`; `14. Контракты для разработчиков (critical)` | `04_authorization_contracts.md` | Здесь находятся уровни проверки, типы проверок, runtime contract и обязательные правила авторизации. | Кодовые блоки и сигнатуры контрактов переносить без русификации технических имен. |
| `10. UI и Security` | `01_overview.md`; связь с будущим `07_ui_configuration.md` | Раздел показывает влияние security на UI, но не является самостоятельной IAM-моделью. | Не переносить UI-правила в authorization contracts без контекста frontend/runtime. |
| `11. Audit и Security`; `12. Integration Security` | `01_overview.md`; связи с `08_audit_history` и `06_integration` | Это связанные требования к audit и integration security. | В целевом документе оставить ссылки, а детальные правила переносить в профильные блоки позже. |
| `13. MVP Scope` | `01_overview.md`; дополнительно backlog/roadmap при необходимости | Раздел задает обязательное и отложенное для MVP. | Не превращать отложенные механизмы в один термин; ABAC, row-level security, SSO/SAML/OIDC и fine-grained policy engine анализировать отдельно. |

</details>

<details>
<summary><strong>Карточка разборки:</strong> <code>docs/01 sources/10_valuesets.md</code> → единая копия в <code>offers/</code> ValueSetData</summary>

Единая копия исходника для разборки:
`docs-new/03_platform/05_value_set_data/offers/10_valuesets.md (01 sources).md`.

Старые локальные дубли внутри отдельных целевых блоков удалены; используется одна source-копия.

#### Паспорт разборки

| Поле | Значение |
| --- | --- |
| Исходный файл | `docs/01 sources/10_valuesets.md` |
| Единая offer-копия | `03_platform/05_value_set_data/offers/10_valuesets.md (01 sources).md` |
| Решение | Разделить через offer-копии |
| Почему так | Исходник описывает одновременно назначение ValueSetData, термины, scope, core concepts, ownership, access principles, связь с Configuration Platform, consumption и MVP patterns. Эти части должны лечь в разные документы блока ValueSetData. |
| Что важно не потерять | Разделение ValueSet, ValueSetData, ValueSetItem и SystemEnum; ownership definition/data; tenant/site overrides; связь с Configuration Platform; consumption by modules; MVP patterns и open decisions. |

#### Раскладка разделов по целевым блокам

| Раздел исходника | Целевой блок `00` | Почему переносится туда | Что проверить при финальном переносе |
| --- | --- | --- | --- |
| `1. Назначение`; `2. Термины`; `3. Scope`; `5. Роль ValueSets в платформе`; `9. Consumption by Other Modules`; `10. MVP Patterns`; `Ключевая мысль документа` | `01_overview.md` | Эти разделы задают обзор, границы, роль ValueSetData и потребителей. | Не расширять ValueSetData до всей Reference Data Platform; сохранить границу с НСИ и SystemEnum. |
| `4. Core Concepts` / `ValueSet`; `ValueSetData`; `SystemEnum`; `6. Ownership Model` / `Definition ownership` | `02_value_set_model.md` | Здесь описана модель набора значений, отличие definition и data, а также SystemEnum. | Развести ValueSet, SystemEnum и reference data; спорные синонимы выносить в глоссарий/вопросы. |
| `4. Core Concepts` / `ValueSetItem`; `6. Ownership Model` / `Data ownership`; `7. Data Access Principles` | `03_value_items.md` | Эти разделы описывают элементы набора, владение данными и правила доступа. | Проверить, какие правила относятся к данным элементов, а какие к общей модели набора. |
| `6. Ownership Model` / `Scope данных`; `8. Связь с Configuration Platform` | `04_tenant_override.md` | Здесь описываются уровни данных, переопределения и связь с configuration scope. | Сверить `Site`/`Tenant` terminology с ADR-018 и текущим правилом по platform scope. |
| `11. Open Decisions` | Миграционный реестр; затем профильные целевые документы после решения | Открытые вопросы нельзя прятать внутри одного финального design-документа. | После решения обновить glossary, ключевые понятия и связанные документы. |

</details>

### 5.9 Предложения по сопоставлению: `docs/adr`

<details>
<summary><strong>Предложения по сопоставлению: `docs/adr`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/adr/drafts/*.md` | Не копировать в `docs-new` как `* (adr drafts).md` | Source остаётся вне `docs-new`; целевые ADR в текущем пакете не создаются | ADR drafts содержат полезные решения, но текущие копии были точными дублями старых источников и не имели целевого frontmatter/status. | Нужны только как источники сверки; нормализация статусов и номеров не блокирует текущий комплект. |
</details>

#### 5.9.1 Что полезного дают старые ADR drafts

Старые ADR drafts полезны не как отдельная целевая папка текущего `docs-new`, а как источники архитектурных ограничений. При переносе документов из `docs/` в `docs-new/` они используются для проверки смысла: не потеряна ли граница, не подменен ли термин, не превратился ли черновик в утвержденное решение без статуса.

| Группа ADR | Что фиксирует | Куда применимо в текущей структуре | Как меняет уже имеющиеся и будущие файлы |
| --- | --- | --- | --- |
| `ADR-001`, `ADR-003`, `ADR-013` | MVP как modular monolith, преемственность от C#/React reference implementation и критерии MVP business slice. | `02_architecture/`, `03_platform/*/`, `04_domain_modules/`, `10_backlog/roadmap/`. | В архитектурных и платформенных документах нельзя описывать микросервисы как обязательное MVP-решение. `Platform Services` и `Domain Services` нужно читать как логические границы, пока отдельный ADR не утвердит физическое выделение сервисов. |
| `ADR-004`, `ADR-005`, `ADR-011` про workflow revision, `ADR-0014` | Разделение workflow command, transition, business action; backend-граница Workflow Runtime; pinning workflow definition revision; инициализация workflow через platform outbox. | `03_platform/04_workflow/`, `03_platform/02_configuration/`, `03_platform/03_object_runtime/`. | В документах нужно развести `CommandCode`, `ActionCode`, `Transition` и runtime-команды. Workflow Configuration не должна становиться местом бизнес-логики, а исправления workflow-документов должны ссылаться на backend runtime и outbox-границу. |
| `ADR-006` | Категории правил и граница Rule Runtime. | `03_platform/05_rules/`, `03_platform/02_configuration/`, `03_platform/04_workflow/`. | Правила отображения, проверки, workflow guards и доменные инварианты нельзя смешивать в одной таблице. В документах нужно явно отделять frontend display behavior от backend authority. |
| `ADR-007`, `ADR-011` про UI, `ADR-014`, `ADR-015` | Граница frontend business logic, гибридная UI Configuration, единая граница artifact/runtime/frontend contract, display values для ссылочных и scope-полей. | `03_platform/02_configuration/`, `03_platform/03_object_runtime/`, `03_platform/12_frontend_platform/`. | UI-документы должны описывать runtime-проекцию и серверные контракты, а не превращать frontend в источник правды. Для ссылочных полей нужно сохранять разделение machine value и display value. |
| `ADR-008`, `ADR-009` | Границы Admin API, Runtime API, Integration API, authentication и authorization. | `02_architecture/`, `03_platform/01_tenant_and_security/`. | В API и security-документах нужно отделять административные API, runtime API и integration API. IAM, permissions и authorization contracts должны быть связаны с Tenant and Security Platform, а не размазаны по UI-документам. |
| `ADR-010`, `ADR-0014` | Постоянные AuditHistory, WorkflowHistory, Outbox и обработка событий. | `03_platform/10_integration_events/`, `03_platform/09_audit_history/`, `03_platform/04_workflow/`; будущий `07_operations` только при появлении сквозного runbook. | Audit/history и integration events нужно разводить по ответственности: история действий и workflow не равна интеграционному событийному outbox, хотя они связаны процессом исполнения. |
| `ADR-012` | Граница ValueSet и SystemEnum. | `03_platform/06_value_sets/`, `03_platform/02_configuration/`, `11_glossary/`. | Value Sets нельзя автоматически расширять до всей Reference Data Platform. При переносе документов нужно проверять, где речь о наборе значений, где о системном enum, а где о платформенных справочных данных. |
| `ADR-016`, `ADR-017` | Object Runtime для стандартных бизнес-объектов; baseline lifecycle, module registry и stable codes. | `03_platform/03_object_runtime/`, `03_platform/02_configuration/`. | Configuration не должна дублировать исполняемую модель стандартного объекта. Документы по baseline, stable codes и module registry нужно проверять на согласованность с `BusinessObjectContract<T>`, `ObjectRuntimeDescriptor`, `SystemBaseline` и published configuration. |
| `ADR-018` | Переименование platform scope `Plant` в `Site`. | `03_platform/01_tenant_and_security/`, `03_platform/02_configuration/`, `03_platform/06_value_sets/`, `03_platform/03_object_runtime/`, `11_glossary/`. | Активные документы должны использовать `Site` для platform scope. `Plant Structure` остается названием прикладной области, а исторические упоминания `Plant` допустимы только как source/reference или при описании старого состояния. |

#### 5.9.2 Как ADR должны влиять на перенос документов

| Правило применения ADR | Что делать при переносе |
| --- | --- |
| ADR со статусом `Draft` | Использовать как сильный архитектурный источник, но не превращать спорные формулировки в утвержденный текст без открытого вопроса. |
| ADR со статусом accepted/implementation branch | Проверять активные документы на соответствие и выносить расхождения в открытые вопросы или отдельные PR. |
| ADR конфликтует с исходным документом | Не переписывать исходный смысл молча. В целевом документе сохранять исходную позицию, а конфликт фиксировать в разделе открытых вопросов и ссылке на ADR. |
| ADR вводит термин или границу | Обновлять `11_glossary`, ключевые понятия целевого документа и связанные документы. |
| ADR описывает runtime/API/contract | Проверять narrative-документ, `03_platform/<area>/03_contracts.md` и `03_platform/<area>/04_runtime.md`. |
| ADR с дублирующейся нумерацией | Не переносить в `docs-new` как целевой ADR до отдельного решения по статусу и номеру. |

#### 5.9.3 Что это меняет в уже имеющихся целевых файлах

| Целевой файл или раздел | Что проверить по ADR | Возможное изменение |
| --- | --- | --- |
| `01_concept/02_system_scope.md` | `ADR-001`, `ADR-003`, `ADR-018`. | Уточнить, где system scope описывает логические границы MVP, а где физическое развёртывание. Проверить plant/site-формулировки. |
| `02_architecture/01_architecture_overview.md` | `ADR-001`, `ADR-003`, `ADR-008`, `ADR-010`. | Архитектурные слои и сервисы описывать как логические границы MVP. API boundaries и outbox/audit/history не смешивать. |
| `02_architecture/01_architecture_overview.md`, `03_platform/*/00_platform_overview.md`, `03_platform/00_foundation/90_traceability.md` | `ADR-001`, `ADR-016`, `ADR-017`, `ADR-018`. | Platform Core должен показывать состав платформенных capability без превращения их в отдельные deployable services. Baseline, stable codes и Object Runtime сверять с ADR. |
| `03_platform/01_tenant_and_security/` | `ADR-008`, `ADR-009`, `ADR-018`. | Уточнить IAM, permissions, authorization contracts и scope-термины. Проверить, что `Site` используется как platform scope, а не смешивается с доменным Plant Structure. |
| `03_platform/02_configuration/` | `ADR-011`, `ADR-014`, `ADR-015`, `ADR-017`. | Развести configuration artifacts, runtime projection, frontend contracts, baseline package/snapshot и published configuration. |
| `03_platform/04_workflow/` | `ADR-004`, `ADR-005`, `ADR-011`, `ADR-0014`. | Вынести workflow runtime, command execution, revision pinning и outbox initialization в согласованную модель. |
| `03_platform/05_rules/` | `ADR-006`. | Проверить, что категории правил не описаны как один общий механизм без различия frontend/backend/domain authority. |
| `03_platform/06_value_sets/` | `ADR-012`, `ADR-018`. | Развести ValueSet, SystemEnum, Reference Data и scope overrides; проверить Site/Plant consistency. |
| `03_platform/10_integration_events/` и `03_platform/09_audit_history/` | `ADR-010`, `ADR-0014`. | Синхронизировать event outbox, workflow history и audit history без смешивания их владельцев. |
| `03_platform/03_object_runtime/` | `ADR-014`, `ADR-015`, `ADR-016`, `ADR-017`. | Проверить Object Mutation Pipeline, runtime display, reference fields, standard object contracts и связь с Configuration. |

| `11_glossary/` | Все ADR, вводящие термины. | После переноса ADR добавить термины, запрещенные синонимы, связанные коды и документы; спорные формы оставить в вопросах. |

### 5.10 Предложения по сопоставлению: остальные разделы `docs`

<details>
<summary><strong>Предложения по сопоставлению: остальные разделы `docs`</strong></summary>

| Документ из `docs` | Место в структуре `00` | Решение | Краткое содержание | Объяснение переноса | Вопрос |
| --- | --- | --- | --- | --- | --- |
| `docs/architecture/module-integration-rules.md` | `04_domain_modules/00_module_documentation_template.md` и `02_architecture/07_integration_architecture.md` | Разделить по владельцам | Правила API/events, запрет прямого доступа к чужому storage, запрет module-local Workflow infrastructure и scale-check. | Общие правила прикладных модулей закреплены в шаблоне; карта конкретных связей и контрактов остаётся в общей интеграционной архитектуре. | Нет |
| `docs/archive/2026-05-31_superseded/04_go_angular_continuity_strategy.md` | `12_appendices/legacy_docs/2026-05-31_superseded/` | Архив | Go + Angular continuity strategy. | Исторический материал, не нормативный. | Нет. |
| `docs/archive/2026-05-31_superseded/ADR-002-target-stack-go-angular-postgresql.md` | `12_appendices/legacy_docs/2026-05-31_superseded/` | Архив | Старый ADR по Go + Angular + PostgreSQL. | Устаревший ADR. | Нет. |
| `docs/archive/2026-05-31_superseded/README.md` | `12_appendices/legacy_docs/2026-05-31_superseded/README.md` | Архив | Индекс superseded-документов. | Сохранять как архивный индекс. | Нет. |
| `docs/archive/2026-05-31_superseded/architecture_drift_decisions_go_angular_mvp.md` | `12_appendices/legacy_docs/2026-05-31_superseded/` | Архив | Drift decisions для перехода на Go + Angular MVP. | Исторический drift. | Нет. |
| `docs/archive/2026-05-31_superseded/architecture_drift_matrix_2026-05-05.md` | `12_appendices/legacy_docs/2026-05-31_superseded/` | Архив | Матрица architecture drift. | Исторический контроль расхождений. | Нет. |
| `docs/archive/2026-05-31_superseded/implementation_inventory_2026-05-05.md` | `12_appendices/legacy_docs/2026-05-31_superseded/` | Архив | Inventory реализации на дату. | Исторический снимок реализации. | Нет. |
| `docs/current/01_platform_implementation_baseline.md` | `12_appendices/research/current_state/`; выборочно `03_platform` и `10_backlog` | Разделить | Текущий baseline реализации платформы. | Использовать для сверки текущего состояния, не как целевую архитектуру. | Какие выводы уже перенесены в target docs? |
| `docs/current/02_platform_capability_map.md` | `12_appendices/research/current_state/`; `10_backlog/roadmap/` | Разделить | Карта текущих platform capabilities. | Источник gap/backlog. | Обновить после свежего master. |
| `docs/current/03_current_platform_architecture.md` | `12_appendices/research/current_state/`; выборочно `02_architecture` | Разделить | Текущая архитектура платформы. | Фиксирует current state, не обязательно target state. | Какие элементы должны стать целевыми? |
| `docs/current/04_object_runtime_reference_module.md` | `03_platform/10_object_runtime/07_create_from_existing.md`; `12_appendices/research/current_state/` | Разделить | Эталонный прикладной модуль Object Runtime. | Пример реализации; целевые правила нужно перенести в Object Runtime docs. | Где пример, а где нормативный паттерн? |
| `docs/current/05_object_runtime_reference_relations_design.md` | `03_platform/10_object_runtime/02_object_contract.md`; `03_platform/10_object_runtime/07_create_from_existing.md` | Разделить | Ссылки и коллекции в эталонном Object Runtime. | Нормативные правила связей должны перейти в Object Runtime. | Сверить с текущими contracts и тестами. |
| `docs/other_tech/dmp-mes-composite-source-feature-audit.md` | `12_appendices/research/mes_composite/` | Архив | Аудит возможностей DMP vs MES Composite. | Исследование, не нормативный target. | Какие выводы уже перенесены в Object Runtime? |
| `docs/other_tech/dmp-module-object-runtime.md` | `03_platform/10_object_runtime/offers/dmp-module-object-runtime.md (other_tech).md` | Offer-копия для разборки | Object Runtime и прикладная бизнес-логика объектов. | Исходник помещен в offer-зону Object Runtime; нормативные части и исследовательские сравнения будут разделяться отдельным шагом. | Отделить целевое решение от анализа MES. |
| `docs/other_tech/dmp-vs-mes-composite-critical-analysis.md` | `12_appendices/research/mes_composite/` | Архив | Критический анализ DMP vs MES Composite. | Research/reference. | Нет. |
| `docs/other_tech/mes-composite-service.md` | `12_appendices/research/mes_composite/` | Архив | Описание MesCompositeService. | Внешний/исторический источник для сравнения. | Нет. |
| `docs/reconciliation/README.md` | `12_appendices/imported_materials/reconciliation/README.md` или `00_governance/06_traceability_rules.md` | Спорно | Reconciliation documents index. | Может быть служебным appendix или правилом сверки. | Нужен ли отдельный процесс reconciliation в governance? |

</details>

## 6. Оставшиеся документы

В этом разделе перечислены файлы из `docs/`, которые пока не отображаются в фактической таблице как отдельный целевой документ, source-, offer- или requirement-копия в `docs-new/`.

Это не означает, что документ потерян: часть материалов является архивом или research, часть требует отдельного решения перед переносом, а часть должна остаться только рабочим источником для backlog или issue tracker.

| Группа | Количество | Что это значит |
| --- | ---: | --- |
| Рабочие/служебные документы | 3 | Не надо переносить автоматически в нормативную структуру; нужен отдельный выбор: backlog, issue tracker или только учет в реестре. |
| Roadmap и backlog preparation | 9 | Нужна отдельная разборка: что остается roadmap, что становится backlog, а что устарело после актуализации master. |
| Архив | 6 | Переносить только как legacy/reference, не как нормативные документы. |
| Current state и research | 8 | Использовать как материал сверки и источники решений, но не переносить один-в-один в target design. |

| Исходный файл | Текущий статус | Что сделать дальше | Комментарий для проверки |
| --- | --- | --- | --- |
| `docs/!bugs_fix.md` | Рабочий список замечаний | Решить: переносить в backlog или вести через issue tracker | Не является нормативным документом. |
| `docs/01 sources/00_release_plan_manifest.md` | Рабочий план очередности | Учитывать в roadmap/миграционном плане, но не переносить как финальный документ | Содержит порядок выпуска материалов, а не целевую архитектуру. |
| `docs/reconciliation/README.md` | Служебная сверка | Решить: appendix для imported materials или governance-процесс reconciliation | Нужен отдельный выбор владельца процесса. |
| `docs/10_backlog/roadmap/preparation/00_platform_completion_estimate.md` | Backlog preparation | Разбирать отдельным блоком roadmap | Проверить актуальность после обновления fork от master. |
| `docs/10_backlog/roadmap/preparation/01_roadmap_preparation_summary.md` | Backlog preparation | Разбирать отдельным блоком roadmap | Содержит сводку подготовки, не финальный roadmap. |
| `docs/10_backlog/roadmap/preparation/02_platform_assessment.md` | Backlog preparation | Разбирать отдельным блоком roadmap | Нужна повторная оценка на актуальном master. |
| `docs/10_backlog/roadmap/preparation/03_roadmap_draft.md` | Backlog preparation | Сравнить с целевым roadmap и перенести только актуальное | Черновик не должен становиться нормативным без проверки. |
| `docs/10_backlog/roadmap/preparation/04_candidate_module_map.md` | Backlog preparation | Сверить с `04_domain_modules/00_module_index` | Возможен конфликт с утвержденной картой модулей. |
| `docs/10_backlog/roadmap/preparation/05_estimation_model.md` | Backlog preparation | Решить, нужна ли методика оценки в документации | Может остаться только как planning/reference. |
| `docs/10_backlog/roadmap/preparation/06_open_decisions_and_risks.md` | Backlog preparation | Разделить на open decisions и risks | Открытые решения вести в backlog и `90_traceability`; риски — в backlog. |
| `docs/10_backlog/roadmap/preparation/07_platform_gap_backlog.md` | Backlog preparation | Обновить после сверки с текущим кодом | Использовать как источник gap backlog, не как design. |
| `docs/10_backlog/roadmap/preparation/08_object_runtime_technology_alignment.md` | Backlog preparation | Сверить с `03_platform/10_object_runtime` | Разделить target rules и backlog alignment. |
| `docs/archive/2026-05-31_superseded/04_go_angular_continuity_strategy.md` | Архив | Переносить только в legacy/reference при необходимости | Устаревший исторический материал. |
| `docs/archive/2026-05-31_superseded/ADR-002-target-stack-go-angular-postgresql.md` | Архив | Переносить только в legacy/reference при необходимости | Устаревший ADR по прежнему стеку. |
| `docs/archive/2026-05-31_superseded/README.md` | Архив | Переносить только как индекс legacy-папки | Не нормативный источник. |
| `docs/archive/2026-05-31_superseded/architecture_drift_decisions_go_angular_mvp.md` | Архив | Переносить только в legacy/reference при необходимости | Исторический drift. |
| `docs/archive/2026-05-31_superseded/architecture_drift_matrix_2026-05-05.md` | Архив | Переносить только в legacy/reference при необходимости | Историческая матрица расхождений. |
| `docs/archive/2026-05-31_superseded/implementation_inventory_2026-05-05.md` | Архив | Переносить только в legacy/reference при необходимости | Исторический снимок реализации. |
| `docs/current/01_platform_implementation_baseline.md` | Current state | Использовать для сверки текущего состояния | Не переносить один-в-один в target design. |
| `docs/current/02_platform_capability_map.md` | Current state | Использовать для gap analysis и roadmap | Сверить с актуальным master. |
| `docs/current/03_current_platform_architecture.md` | Current state | Использовать для проверки архитектурных расхождений | Отличать current state от целевой архитектуры. |
| `docs/current/04_object_runtime_reference_module.md` | Current state | Вытащить нормативные паттерны в Object Runtime после анализа | Отделить пример реализации от правила. |
| `docs/current/05_object_runtime_reference_relations_design.md` | Current state | Сверить с Object Runtime contracts | Отделить актуальные правила ссылок и коллекций от research. |
| `docs/other_tech/dmp-mes-composite-source-feature-audit.md` | Research | Использовать как источник анализа для Object Runtime | Не переносить как нормативный документ. |
| `docs/other_tech/dmp-vs-mes-composite-critical-analysis.md` | Research | Использовать как appendix/research при необходимости | Не переносить как target design. |
| `docs/other_tech/mes-composite-service.md` | Research | Использовать только как внешний исторический источник | Не переносить в нормативную структуру без выделения решений. |

## 7. Следующий порядок работы

| Шаг | Что делать | Критерий готовности |
| --- | --- | --- |
| 1 | Начать с `docs/01 sources`, потому что это основной архитектурный массив. | Для каждого файла есть отдельная ветка или отдельный PR. |
| 2 | Не переносить спорные файлы без решения. | Вопрос зафиксирован в этом реестре и не скрыт в тексте целевого документа. |
| 3 | Для файлов со статусом `Разделить` сначала сделать карту разбиения, затем переносить частями. | В одном PR не смешиваются разные целевые документы без необходимости. |
| 4 | Для `docs/03 conf_artefacts` сначала определить stable artifact vocabulary. | Каждый артефакт связан с design-документом и schema/contract. |
| 5 | Для `docs/02 requrements` отделять требования, UX-макеты и проверочные сценарии. | Нормативный design не содержит непроверенных UI-черновиков. |
| 6 | ADR drafts не переносить в текущий `docs-new`; использовать только как источники сверки. | В `docs-new` нет `* (adr drafts).md` и нет физического `09_decisions` без утверждённого ADR. |
| 7 | Архив и research переносить после основных документов. | Они не влияют на целевую структуру и не блокируют архитектурный перенос. |

## 8. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 17:29 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: добавить автоматическую историю реестра миграции | [113d258d](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/113d258dcd3c9199e3d30704ed8b5908b1ee47e6) |
