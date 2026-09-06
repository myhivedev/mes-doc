---
id: DOC-00-00-00
title: '00 Documentation Strategy — Digital Manufacturing Platform'
type: architecture
status: in-review
version: '1.5'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: governance
module: governance
holder: '@axelprosoft'
created_at: 2026-08-07 10:40
created_by: '@axelprosoft'
updated_at: 2026-08-27 15:50
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: upstream
---

# 00 Documentation Strategy — Digital Manufacturing Platform

## 1. Назначение

Этот документ фиксирует стартовую стратегию ведения проектной, архитектурной, эксплуатационной, контрактной и прикладной документации для Digital Manufacturing Platform.

Цель — дать команде достаточно конкретные правила, чтобы можно было сразу создавать, переносить и сопровождать документацию в большом проекте, где одновременно работают аналитики, архитекторы, разработчики, QA, DevOps/SRE, интеграторы, заказчик и AI-ассистенты.

Документ задает:

- основной формат документации;
- среду ведения;
- структуру папок;
- типы документов;
- шаблон внутренних документов;
- правила для прикладных модулей из утвержденных функциональных требований;
- процесс изменения документов;
- CI-проверки;
- правила использования AI.

Аудитория документа:

- аналитики и владельцы требований;
- архитекторы и технические лиды;
- разработчики backend/frontend;
- QA;
- DevOps/SRE;
- интеграторы;
- представители заказчика, участвующие в согласовании документации.

Документ должен быть понятен без знания внутренней истории обсуждений. Если правило зависит от архитектурного решения, рядом должна быть ссылка на документ, ADR или open decision.

---

## 2. Базовое решение

Для проекта принимается подход **docs-as-code**.

```text
Markdown-файлы в Git
  → Проверка pull request
  → CI-проверки документации
  → Опубликованный документационный портал
```

Базовые решения:

1. **Git является источником истины** для утвержденной документации.
2. **Markdown является основным форматом**.
3. **Документация меняется через ветку, pull request и проверку**.
4. **Опубликованная документация собирается в документационный портал**.
5. **AI может помогать писать, проверять и синхронизировать документы**, но не обходит проверку.
6. **Контракты хранятся в машинно-читаемом виде**, где это возможно: OpenAPI, AsyncAPI, JSON Schema, YAML-каталоги кодов.

### 2.1 Язык и терминология

Основной язык нормативной документации проекта — русский.

В обычном тексте и заголовках используются русские формулировки: `источник истины`, `область действия`, `контракт`, `поведение во время выполнения`, `стабильный код`, `расхождение документации и кода`, `готовность к разработке`, `готовность к утверждению`.

Английские названия используются только там, где это имена стандартов, API-форматов, кодовых артефактов, каталогов, файлов, YAML-полей или официальных названий платформенных блоков. Если английский термин важен для кода или контракта, он указывается как технический алиас при первом употреблении.

---

## 3. Источник истины

### 3.1 Основной источник истины

Основным источником истины является Git-репозиторий.

Стартовый вариант:

```text
repo/
  docs-new/
```

`docs-new/` является корнем нормативной документации.

Позже возможна гибридная модель:

```text
dmp-docs/                         # общая нормативная документация
service-configuration/docs/        # service-local docs
service-iam/docs/                  # service-local docs
service-order-management/docs/     # service-local docs
```

Даже если часть документации будет жить рядом с сервисами, опубликованный документационный портал должен собирать ее в единую структуру.

### 3.2 Что не является источником истины

Не являются основным источником истины:

- Confluence;
- Notion;
- SharePoint;
- Word;
- Excel;
- PowerPoint;
- Slack / Teams;
- письма;
- локальные файлы;
- неутвержденные AI-черновики.

Эти инструменты допустимы для обсуждений, черновиков, протоколов и внешнего обмена, но утвержденное решение должно быть перенесено в Git-документацию.

---

## 4. Форматы

### 4.1 Основной формат

Основной формат:

```text
*.md
```

Markdown используется для:

- концепции;
- описание области действия;
- архитектуры;
- platform design;
- module specs;
- требований;
- ADR;
- runbooks;
- эксплуатационных инструкций;
- тестовой стратегии;
- глоссария.

### 4.2 Дополнительные форматы

```text
OpenAPI YAML/JSON       # REST API-контракты
AsyncAPI YAML/JSON      # контракты событий
JSON Schema             # схемы payload/config
YAML                    # каталоги, стабильные коды, матрицы
Mermaid                 # простые диаграммы
PlantUML                # сложные диаграммы
```

Правило:

> Текстовые version-control friendly форматы предпочтительнее бинарных файлов.

### 4.3 Нежелательные основные форматы

DOCX/XLSX/PDF/PPTX не используются как основной формат нормативной документации.

Исключения допустимы для:

- официальной передачи заказчику;
- презентаций;
- подписываемых документов;
- legacy-приложений;
- внешних материалов.

Но источник решения все равно должен быть отражен в Markdown.

---

## 5. Рекомендуемая среда

Стартовая среда:

```text
Git + Markdown + MkDocs Material + CI-проверки
```

| Компонент | Назначение |
|---|---|
| Git | versioning, branches, history |
| Markdown | основной формат |
| MkDocs Material | документационный портал |
| CI | проверки сборки, ссылок, структуры |
| VS Code / JetBrains | редактирование |
| GitHub / GitLab / Azure DevOps | PR, проверка, согласования |

### 5.1 Почему не Confluence как главное хранилище

Confluence удобен для обсуждений, но хуже подходит для:

- branch-based development;
- проверки кода;
- синхронизации с кодом;
- массовых AI-правок;
- автоматических проверок;
- контроля дрейфа между документацией и кодом.

Поэтому Confluence может быть вспомогательным инструментом, но не источником истины.

### 5.2 Возможные будущие расширения

| Инструмент | Когда рассматривать |
|---|---|
| Docusaurus | если нужен React/MDX developer portal |
| Backstage TechDocs | если появится internal developer portal и service catalog |
| Antora | если документация станет multi-repo/multi-version enterprise-системой |
| Vale | если нужен строгий language/style linting |
| Structurizr | если нужно формализовать C4-модель |

Для старта они не обязательны.

---

## 6. Базовая структура `docs-new/`

```text
repo/
  docs-new/
    00_governance/
    01_concept/
    02_architecture/
    03_platform/
    04_domain_modules/
    10_backlog/
    11_glossary/
    12_appendices/
```

---

## 7. Назначение разделов

### 7.1 `00_governance/`

Правила управления документацией и проектными решениями.

```text
00_governance/
  00_documentation_strategy.md
  01_document_template.md
  99_archive_migration_review.md
  02_platform_area_documentation_playbook.md
  03_contract_documentation_standard.md
  04_module_documentation_method.md
  requirements/
    00_index.md
    001_dmp_functional_requirements_and_constraints.md
```

### 7.2 `01_concept/`

Верхнеуровневая концепция продукта.

```text
01_concept/
  01_product_concept.md
  02_system_scope.md
  03_business_goals.md
  04_stakeholders.md
  05_success_metrics.md
```

### 7.3 `02_architecture/`

Общая архитектура системы.

```text
02_architecture/
  01_architecture_overview.md
  02_architecture_principles.md
  03_logical_architecture.md
  04_service_architecture.md
  05_data_architecture.md
  06_deployment_architecture.md
  07_integration_architecture.md
  08_security_architecture.md
  diagrams/
    context.mmd
    containers.mmd
    deployment.mmd
```

Документ `07_integration_architecture.md` является канонической картой
межмодульных взаимодействий DMP: он показывает основные HTTP API, application
ports, host adapters и integration events между областями. Платформенная область
описывает в своих документах только собственные контракты, сценарии и
ограничения, а не копирует общую карту.

### 7.4 `03_platform/`

Нормативные документы по платформенному ядру (Platform Core).

```text
03_platform/
  00_platform_documentation_template.md
  00_foundation/
    00_platform_overview.md
    01_scope.md
    ...
    90_traceability.md
  01_tenant_and_security/
    00_platform_overview.md
    01_scope.md
    ...
    90_traceability.md
  02_configuration/
    00_platform_overview.md
    01_scope.md
    ...
    90_traceability.md
    artifact_types/
  03_object_runtime/
  04_workflow/
  05_rules/
  06_value_sets/
  07_settings/
  08_numbering/
  09_audit_history/
  10_integration_events/
  11_reporting_output/
  12_frontend_platform/
```

Назначение: зафиксировать общие механизмы, обязательные для всех модулей.

Единый состав документов платформенной области задаётся в `03_platform/00_platform_documentation_template.md`: `00_platform_overview.md`, `01_scope.md`, `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `05_security_and_audit.md`, `06_user_experience.md`, `07_quality.md`, `08_operations.md`, `90_traceability.md`. `00_platform_overview.md` и `01_scope.md` обязательны; остальные документы создаются только при наличии содержания. Уровень детализации API, событий, extension points и других граничных контрактов задаётся [Стандартом описания контрактов](03_contract_documentation_standard.md).

Состояние конкретного факта, поля, требования или решения не смешивается со статусом файла во frontmatter. Для сведений используется контролируемая классификация из `03_platform/00_platform_documentation_template.md` и `02_platform_area_documentation_playbook.md`; расхождения и открытые решения с устойчивыми ID фиксируются в `90_traceability.md`.

Переходные папки старой структуры, например `02_configuration_platform/`, `03_workflow_engine/`, `05_value_set_data/`, `10_object_runtime/`, могут временно оставаться в репозитории как склад исходных материалов. Они не являются целевым нормативным пакетом после появления соответствующей новой области.

После завершения миграции переходная копия закрывается по процедуре
[«Закрыть переходную папку после миграции»](02_platform_area_documentation_playbook.md#закрыть-переходную-папку-после-миграции).
Исходные материалы в `docs/` сохраняются, а актуальный канонический маршрут и
состояние удалённой копии фиксируются в
[`99_archive_migration_review.md`](99_archive_migration_review.md). Переходная
папка не используется как архив требований.

Базовый набор модулей ядра платформы:

| Платформенный блок | Назначение | Граница ответственности |
|---|---|---|
| Tenant Security | Контекст предприятия и площадки, пользователи, роли, права и проверка доступа | Не описывает бизнес-структуру предприятия как прикладные основные данные |
| Configuration | Версионированная публикация конфигурационных артефактов, baseline, effective configuration и схемы артефактов | Не исполняет бизнес-сценарии и не хранит прикладные основные данные |
| Object Runtime | Единый механизм выполнения операций с бизнес-объектами: чтение, создание, изменение, удаление, выполнение действий, подключение правил, workflow, аудита и прав | Не владеет бизнес-смыслом объектов и не заменяет прикладные модули |
| Workflow | Исполнение состояний, команд, переходов и истории workflow | Не подменяет доменную логику прикладного модуля |
| Rules | Исполнение правил по точкам выполнения | Не хранит локальные инварианты доменной модели как единственный источник |
| Value Sets | Технический механизм наборов значений, элементов, системных enum и tenant/site override | Не является модулем Общая НСИ и не владеет номенклатурой, контрагентами, серийными номерами |
| Settings | Каталог настроек, значения, области действия, effective value и user preferences | Не является частью общей Configuration, если речь идёт о runtime-значениях настроек |
| Numbering | Правила нумерации, счётчики и интеграция с Object Runtime | Не хранит предметный жизненный цикл объекта |
| Audit History | Аудит, история объектов, трассируемость изменений | Не заменяет бизнес-журналы модулей, если они являются частью предметной области |
| Integration Events | Конверт события, outbox/inbox, идемпотентность, публикация событий | Не задаёт бизнес-смысл событий без модуля-владельца |
| Reporting Output | Определения report/output и runtime формирования результата | Не владеет источниками данных и не заменяет Dataset / Read Query Capability |
| Frontend Platform | Общие frontend contracts, runtime packages, shell и platform applications | Не владеет предметными сценариями прикладных модулей |

Правило разделения платформы и прикладной области:

> Платформенное ядро предоставляет общие механизмы и контракты. Прикладной модуль владеет бизнес-смыслом, основными данными, жизненным циклом бизнес-объектов и требованиями из своего раздела функциональных требований.

Правило для Object Runtime:

> Object Runtime является частью платформенного ядра. В документации на русском языке его следует пояснять как `единый механизм выполнения операций с бизнес-объектами`. Прикладные модули не должны описывать собственный альтернативный механизм выполнения операций с объектами как целевое решение. Если модулю нужен особый обработчик, обход части правил, пакетная операция или интеграционный сценарий, это описывается как расширение или исключение из общего механизма Object Runtime.

Нормативная документация по Object Runtime должна находиться в `03_platform/03_object_runtime/`. Если целевого документа еще нет в `docs-new/`, решение фиксируется как открытое проектное решение или backlog-задача и не заменяется ссылкой на материалы вне `docs-new/`.

Особенно важно:

- ValueSetData описывает технический механизм наборов значений и их tenant-переопределений.
- Общая НСИ (`general_master_data`) является прикладным модулем и описывает бизнес-данные: единицы измерения, номенклатуру, виды и группы номенклатуры, реквизиты номенклатуры, серийные номера, контрагентов.
- Machine Data Collection (`machine_data_collection`) является прикладным модулем мониторинга оборудования, а не заменой общей Integration Foundation.
- Product & Process Definition (`product_process_definition`) ведется как единый прикладной модуль, пока требования не будут официально разделены.

### 7.5 `04_domain_modules/`

Документация по прикладным модулям. Стартово модулей 13 по функциональным требованиям, но структура должна оставаться одинаковой при расширении состава модулей.

```text
04_domain_modules/
  01_module_index.md
  00_module_documentation_template.md

  00_common/
    00_module_overview.md
    02_domain_model.md
    03_object_runtime_model.md
    05_rules.md
    90_traceability_pr00.md

  01_general_master_data/
    00_module_overview.md
    01_scope.md
    02_domain_model.md
    03_object_runtime_model.md
    04_workflows.md
    05_rules.md
    06_ui_views.md
    07_reports_outputs.md
    08_api_contracts.md
    09_events.md
    10_value_set_data_usage.md
    11_permissions.md
    12_audit_history.md
    13_operations.md
    14_test_strategy.md
    backlog.md

  02_product_process_definition/
  03_plant_structure/
  04_resource_management/
  05_document_management/
  06_project_management/
  07_order_management/
  08_planning_scheduling/
  09_production_logistics/
  10_shopfloor_execution/
  11_quality_management/
  12_machine_data_collection/
  13_manufacturing_analytics/
```

Правило:

> Каждый прикладной модуль документируется по единому шаблону. Документы шаблона создаются при наличии проектного содержания.

Стартовый список прикладных модулей должен соответствовать утвержденным функциональным требованиям DMP:

| ModuleNo | ModuleCode | ModuleFolder | Русское название | English name |
|---|---|---|---|---|
| `01` | `general_master_data` | `01_general_master_data` | Общая НСИ и основные данные | General Master Data |
| `02` | `product_process_definition` | `02_product_process_definition` | Составы и технологии | Product & Process Definition |
| `03` | `plant_structure` | `03_plant_structure` | Производственная структура | Plant Structure |
| `04` | `resource_management` | `04_resource_management` | Управление ресурсами | Resource Management |
| `05` | `document_management` | `05_document_management` | Управление документацией | Document Management |
| `06` | `project_management` | `06_project_management` | Управление проектами | Project Management |
| `07` | `order_management` | `07_order_management` | Управление заказами | Order Management |
| `08` | `planning_scheduling` | `08_planning_scheduling` | Планирование и расписания | Advanced Planning & Scheduling |
| `09` | `production_logistics` | `09_production_logistics` | Производственная логистика | Production Logistics |
| `10` | `shopfloor_execution` | `10_shopfloor_execution` | Управление операциями | Shopfloor Operations Management |
| `11` | `quality_management` | `11_quality_management` | Управление качеством | Quality Management |
| `12` | `machine_data_collection` | `12_machine_data_collection` | Мониторинг оборудования (MDC) | Machine Data Collection |
| `13` | `manufacturing_analytics` | `13_manufacturing_analytics` | Производственная аналитика (BI) | Manufacturing Analytics |

Правила соответствия функциональным требованиям:

1. Нельзя добавлять новый прикладной модуль в документационную структуру без явного решения, к какому разделу требований он относится.
2. Нельзя разделять модуль из функциональных требований на несколько документационных модулей без отдельного архитектурного решения или ADR.
3. Нельзя объединять несколько модулей требований в один documentation module без явного описания границ и последствий для владения.
4. Каждый `ModuleCode` должен быть стабильным: переименование требует решения о миграции и обновления ссылок, владельца, контрактов и стабильных кодов.
5. Для каждого прикладного модуля должен быть указан источник требований: номера `REQ-xx-*` и `LIM-xx-*`, относящиеся к модулю.

### 7.6 Контракты

Контракты описываются в документах владельцев: для платформенных областей это
`03_platform/<area>/03_contracts.md`, для прикладных модулей —
`04_domain_modules/<module>/08_api_contracts.md` и `09_events.md`.

Верхний раздел `05_contracts/` не используется. Машинные файлы из старого
`docs/05 contracts` рассматриваются как исходные материалы: они сверяются с
кодом и документами владельцев, но не переносятся в `docs-new` как отдельный
тип проектной документации.

### 7.7 Условные будущие разделы

Следующие разделы не входят в текущую базовую структуру `docs-new/` и не
создаются как пустые папки.

| Раздел | Когда может появиться | Где вести содержание сейчас |
| --- | --- | --- |
| `07_operations/` | Только при появлении сквозной эксплуатационной документации всей платформы: окружения, deployment, backup/restore, мониторинг, incident response или platform-wide runbooks. | `03_platform/*/08_operations.md`, документы владельцев, `10_backlog` |
| `08_testing/` | Только при появлении сквозной QA/test strategy всей платформы: общие acceptance criteria, contract/integration/E2E/security/performance подходы или test data strategy. | `03_platform/*/07_quality.md`, `90_traceability.md`, документы модулей, `10_backlog` |
| `09_decisions/` | Только если отдельно принято решение вести самостоятельные ADR с владельцем, статусом, номером, последствиями и ссылками из документов-владельцев. | Документы-владельцы, `90_traceability.md`, `10_backlog`; черновики `docs/adr/drafts` остаются источниками сверки |

`05_contracts/` не является условным будущим разделом: верхний раздел
контрактов не используется. Контракты ведутся у владельцев.

### 7.8 `10_backlog/`

Крупные feature specs, roadmap и requirements, если они слишком большие для issue tracker.

```text
10_backlog/
  epics/
    epic-configuration-platform-mvp.md
    epic-workflow-foundation.md
    epic-value-set-data-mvp.md
  requirements/
    req-configuration-publish.md
    req-effective-ui-resolution.md
  roadmap/
    roadmap-mvp.md
    roadmap-platform-foundation.md
```

Issue tracker хранит задачи и статус исполнения. Markdown хранит крупное требование, контекст и архитектурные детали.

### 7.9 `11_glossary/`

Терминология.

```text
11_glossary/
  platform_terms.md
  manufacturing_terms.md
  configuration_terms.md
  security_terms.md
  object_runtime_terms.md
  frontend_platform_terms.md
  reporting_output_terms.md
```

### 7.10 `12_appendices/`

Вспомогательные материалы.

```text
12_appendices/
  imported_materials/
  legacy_docs/
  meeting_notes/
  research/
  vendor_references/
```

---

## 8. Frontmatter документа

Каждый нормативный документ должен начинаться с YAML frontmatter. Шапка хранит
идентичность документа, его принадлежность, ответственность и состояние
согласования. Содержательные связи между документами указываются в основном тексте,
а не отдельным полем шапки.

```yaml
---
id: DOC-<РАЗДЕЛ>-<ПАПКА>-<ДОКУМЕНТ>
title: '<Название, совпадающее с H1>'
type: <значение по расположению документа>
status: draft
version: '0.1'
owner: '@<github-username владельца>'
reviewers: []
scope: <область действия>
module: <имя папки или модуля>
holder: '@axelprosoft'
created_at: YYYY-MM-DD HH:MM
created_by: '@<github-username автора>'
updated_at: YYYY-MM-DD HH:MM
last_modified_by: '@<github-username автора>'
last_reviewed: null
review_status: not_started
supersedes: []
source: upstream
---
```

Если утверждение документа сверено с конкретной редакцией исходников, разрешено
добавить необязательное поле `source_revision` в формате
`<remote-or-branch>@<commit>`, например
`source_revision: origin/master@0123456789abcdef`.

### 8.1 Назначение ID по пути

Для новых документов используется одна основная схема: все три кода берутся из
структуры пути.

```text
docs-new/<NN_раздел>/<NN_папка>/<NN_документ>.md
                       ↓
                 DOC-NN-NN-NN
```

Например,
`docs-new/04_domain_modules/02_orders/05_rules.md` получает ID
`DOC-04-02-05`. Каждый числовой префикс нормализуется до двух цифр: префикс
`001_` файла соответствует коду документа `01`.

Документу непосредственно в `00_governance` назначается код модуля `00`, потому
что сам раздел является корнем управления документацией. Служебный документ,
расположенный непосредственно в другом разделе и вне нумерованного модуля,
получает резервный код модуля `99`. Поэтому
`03_platform/00_platform_documentation_template.md` получает ID
`DOC-03-99-00`, а документ платформенной области
`03_platform/00_foundation/00_platform_overview.md` — `DOC-03-00-00`. Эти ID не
конфликтуют.

Именованной папке `docs-new/00_governance/requirements` закреплён код `99`:

| Путь | ID |
|---|---|
| `00_governance/requirements/00_index.md` | `DOC-00-99-00` |
| `00_governance/requirements/001_dmp_functional_requirements_and_constraints.md` | `DOC-00-99-01` |

Новые документы должны размещаться в нумерованной структуре. Зарегистрированные
служебные файлы, созданные до введения этого правила, не являются образцом для
назначения новых ID. Соответствие ID пути и отсутствие дублей проверяет workflow.

### 8.2 Допустимые `type`

```text
concept
scope
architecture
design
governance
contract
runtime
module-spec
requirement
adr
runbook
operation
assurance
traceability
testing
glossary
appendix
```

Тип выбирается по расположению и назначению документа:

| Расположение | `type` |
|---|---|
| `00_governance/00_documentation_strategy.md` | `architecture` |
| `00_governance/requirements/` | `requirement` |
| `00_governance/01_document_template.md` | `appendix` |
| Прочие стандарты и playbook в `00_governance/` | `governance` |
| `00_governance/02_platform_area_documentation_playbook.md` | `governance` |
| `01_concept/` | `concept` |
| `02_architecture/` | `architecture` |
| `03_platform/00_platform_documentation_template.md` | `appendix` |
| Целевой пакет `03_platform/<NN_область>/` | По имени файла из таблицы ниже |
| Остальные документы `03_platform/` | `design` |
| `03_platform/*/02_architecture.md` | `architecture` |
| `03_platform/*/03_contracts.md` | `contract` |
| `03_platform/*/04_runtime.md` | `runtime` |
| `03_platform/*/05_security_and_audit.md`, `03_platform/*/07_quality.md` | `assurance` |
| `03_platform/*/08_operations.md` | `operation` |
| `03_platform/*/90_traceability.md` | `traceability` |
| `04_domain_modules/` | По назначению файла из таблицы 10.0 |
| `07_operations/runbooks/`, если создан условный раздел | `runbook` |
| Остальные документы `07_operations/`, если создан условный раздел | `operation` |
| `08_testing/`, если создан условный раздел | `testing` |
| `09_decisions/*.md`, если создан условный раздел ADR | `adr` |
| `10_backlog/` | `requirement` |
| `11_glossary/` | `glossary` |
| `12_appendices/` | `appendix` |

Допустимые значения: `concept`, `scope`, `architecture`, `design`,
`governance`, `contract`, `runtime`, `module-spec`, `requirement`, `adr`, `runbook`,
`operation`, `assurance`, `traceability`, `testing`, `glossary`,
`appendix`.

Для целевого пакета платформенной области действуют следующие значения:

| Файл | Допустимый `type` |
|---|---|
| `00_platform_overview.md` | `design` |
| `01_scope.md` | `scope` |
| `02_architecture.md` | `architecture` |
| `03_contracts.md` | `contract` |
| `04_runtime.md` | `runtime` |
| `05_security_and_audit.md` | `assurance` |
| `06_user_experience.md` | `design` |
| `07_quality.md` | `assurance` |
| `08_operations.md` | `operation` |
| `90_traceability.md` | `traceability` |

`runtime` описывает исполнение конкретной платформенной возможности, тогда как
`runtime-convention` задаёт общее соглашение для нескольких областей. Тип
`assurance` объединяет документы о проверяемых гарантиях безопасности, аудита и
качества. Предметные названия `security`, `ux` и `quality` не используются как
`type`: безопасность и качество относятся к `assurance`, пользовательское
взаимодействие — к `design`. Для эксплуатации используется единый тип
`operation`, без дублирующего варианта `operations`.

### 8.3 Допустимые `status`

```text
draft
in-review
approved
deprecated
superseded
archived
```

| `status` | Допустимый `review_status` | Значение |
|---|---|---|
| `draft` | `not_started` | Черновик, согласование не начато |
| `in-review` | `in_review`, `changes_requested`, `awaiting_reviewer` | На согласовании |
| `approved` | `approved` | Утверждённый источник истины |
| `deprecated` | `approved` | Устаревает, но ещё применим |
| `superseded` | `approved` | Заменён другим документом |
| `archived` | `approved` | Исторический документ |

### 8.4 Как выбирать значения полей

| Поле | Как заполнить при создании |
|---|---|
| `id` | Получить из пути по правилу 8.1 |
| `title` | Указать полное название; оно должно точно совпадать с единственным H1 |
| `type` | Выбрать по расположению документа согласно 8.2 |
| `status` | Для нового документа указать `draft`; дальнейшие переходы выполняются по разделу 22 |
| `version` | Для нового документа указать `'0.1'`; далее поле ведёт workflow |
| `owner` | Указать GitHub username владельца содержания с `@` |
| `reviewers` | Перечислить GitHub usernames проверяющих с `@`; если они не назначены, оставить `[]` |
| `scope` | Указать область верхнего уровня без числового префикса, например `governance`, `platform`, `domain`, `operations` |
| `module` | Указать имя папки модуля без числового префикса; для `requirements` — `requirements` |
| `holder` | Указать `@axelprosoft` |
| `created_at` | Указать дату и время создания в формате `YYYY-MM-DD HH:MM` |
| `created_by` | Указать GitHub username автора с `@` |
| `updated_at` | При создании повторить `created_at`; далее поле ведёт workflow |
| `last_modified_by` | При создании повторить `created_by`; далее поле ведёт workflow |
| `last_reviewed` | Для ещё не проверенного документа указать `null` |
| `review_status` | Выбрать пару для текущего `status` по таблице 8.3; при создании указать `not_started` |
| `supersedes` | Оставить `[]`, если документ ничего не заменяет; иначе перечислить ID заменяемых документов |
| `source` | Указать происхождение документа согласно таблице ниже; это поле не заменяет `source_revision`. |
| `source_revision` | Необязательное поле. Если документ сверялся с исходниками, указать `<remote-or-branch>@<commit>` |

Допустимые значения `source` имеют следующий смысл:

| Значение | Когда используется |
| --- | --- |
| `authored` | Документ написан и поддерживается как нормативный или рабочий материал текущего репозитория. Он может основываться на коде, требованиях и старых источниках. |
| `upstream` | Документ сохранён из ранее существовавшего утверждённого или внешнего источника без полной переработки в текущий нормативный пакет. |
| `imported` | Временная копия исходного материала для миграции; допускается только в `offers/`, `requirements/` или другом явно обозначенном импортированном узле. |

Файлы в каталогах `offers/` являются рабочими source-копиями для разбора и не
считаются версионируемыми нормативными документами `docs-new`. После разбора
содержание переносится в целевой документ с YAML-шапкой или в backlog, а
source-копия удаляется. Если материал в `offers/` должен стать нормативным
документом, его сначала переносят в канонический каталог и оформляют по
шаблону.

Для документов, написанных в `docs-new` по результатам анализа, используется
`authored`. Ревизия кода и ветка, по которым проверялись утверждения, указываются
отдельно в `source_revision`.

`review_status` является техническим отражением состояния PR. Для нового файла
автор задаёт только исходную пару `draft` / `not_started`; далее поле ведёт
workflow. При редактировании существующего документа пользователь и AI не должны
менять `review_status` вручную. Workflow восстанавливает его до допустимого
значения до строгой проверки и не позволяет устаревшей ветке откатить нормативный
`status` из основной ветки.

### 8.5 Версия и история изменений

- Поле хранится строкой в формате `MAJOR.MINOR`, например `version: '1.2'`.
- При создании указываются версия `0.1` и пустая таблица истории изменений.
  Строку 0.1 автоматически добавляет workflow после коммита файла; автор не
  добавляет её вручную.
- Линия `0.x` соответствует состояниям `draft` и `in-review`.
- Согласованный переход `in-review` → `approved` создаёт первую утверждённую
  базовую версию `1.0`.
- Переход `approved` → `deprecated` остаётся в линии `1.x` и повышает `MINOR`.
- Переход `deprecated` → `superseded` создаёт версию `2.0`.
- Переход из любого состояния в `archived` создаёт версию `2.0`. Если документ
  уже находится в линии `2.x` или выше, версия продолжает расти без отката.
- Обычное содержательное изменение внутри текущего состояния повышает `MINOR`.
  Существенная переработка с заменой нормативного документа оформляется через
  `deprecated` → `superseded` и поле `supersedes` нового документа.
- После создания нельзя вручную менять `version`, `updated_at`,
  `last_modified_by` и таблицу истории. Workflow обновляет их от имени автора
  исходного изменения.
- Раздел `История изменений` должен быть последним разделом документа.
- Пустая таблица истории допустима только до первой автоматической записи
  workflow. Для нового файла workflow создаёт строку 0.1 по его исходному коммиту. Для
  подключаемого существующего документа пустая таблица разрешается, только если в базовой
  ветке ещё нет валидной истории; строка 0.1 создаётся по исходному Git-созданию файла.
  Удаление или перезапись уже существующей валидной истории отклоняется.
- Файл `00_governance/01_document_template.md` является служебной заготовкой:
  он постоянно хранится как `draft / not_started`, версия `0.1`, с пустой
  таблицей истории. Автоматизация жизненного цикла не изменяет сам шаблон;
  обычные правила начинают действовать для созданного по нему нового документа.

---

## 9. Структура нормативного документа

Рекомендуемая структура:

```markdown
# Название документа

## 1. Назначение

## 2. Область действия

### 2.1 Входит

### 2.2 Не входит

## 3. Контекст

## 4. Ключевые понятия

## 5. Модель / проектное решение

## 6. Поведение во время выполнения

## 7. Контракты

## 8. Правила валидации

## 9. Безопасность и tenant-модель

## 10. Аудит, история и трассируемость

## 11. Импорт, экспорт и миграция

## 12. MVP-объем

## 13. Открытые решения

## 14. Связанные документы

## 15. История изменений
```

Не каждый документ обязан содержать все разделы, но для документов уровня платформы эта структура рекомендуется.

---

## 10. Шаблон прикладного модуля

Каждый прикладной модуль должен иметь одинаковую структуру.

```text
04_domain_modules/<module_number>_<module_code>/
  00_module_overview.md
  01_scope.md
  02_domain_model.md
  03_object_runtime_model.md
  04_workflows.md
  05_rules.md
  06_ui_views.md
  07_reports_outputs.md
  08_api_contracts.md
  09_events.md
  10_value_set_data_usage.md
  11_permissions.md
  12_audit_history.md
  13_operations.md
  14_test_strategy.md
  15_navigation_menu.md
  backlog.md
```

### 10.0 Назначение и типы документов прикладного модуля

| Файл | Рекомендуемый `type` | Зачем нужен |
|---|---|---|
| `00_module_overview.md` | `module-spec` | Дает краткую карту модуля: назначение, бизнес-возможность, основные сценарии, зависимости и состав документов. |
| `01_scope.md` | `scope` | Фиксирует границы модуля: что входит, что не входит, какие требования покрываются и где проходят границы с соседними модулями. |
| `02_domain_model.md` | `design` | Описывает бизнес-сущности модуля, поля, связи, владение данными, наследование от Common и границы агрегатов. |
| `03_object_runtime_model.md` | `design` | Описывает, как объекты модуля публикуются в Object Runtime: object types, поля, ссылки, коллекции, представление, lookup, поиск, действия и особые обработчики. |
| `04_workflows.md` | `design` | Описывает workflow только для тех object types, которым нужен жизненный цикл сверх стандартных Common-правил архивирования и удаления. |
| `05_rules.md` | `design` | Фиксирует инварианты, проверки, уникальности, ограничения изменения данных и настраиваемые правила модуля. |
| `06_ui_views.md` | `design` | Описывает пользовательские списки, карточки, вкладки, фильтры, формы выбора и состав UI-представлений. |
| `07_reports_outputs.md` | `design` | Описывает отчеты, выходные формы, экспорты и наборы данных, которые предоставляет модуль. |
| `08_api_contracts.md` | `contract` | Описывает публичные и внутренние API только там, где стандартного Object Runtime недостаточно или нужен внешний контракт. |
| `09_events.md` | `contract` | Описывает публикуемые и потребляемые события модуля и их payload-контракты. |
| `10_value_set_data_usage.md` | `design` | Описывает использование общих ValueSet, локальных списков значений и правил ведения классификационных данных. |
| `11_permissions.md` | `design` | Описывает права доступа к объектам, действиям, workflow-командам и UI-представлениям модуля. |
| `12_audit_history.md` | `design` | Описывает аудит изменений, историю объектов, историю workflow и требования к трассировке действий пользователя. |
| `13_operations.md` | `design` | Описывает прикладные операции, функции и алгоритмы модуля: создание, копирование, пересчет, генерацию, массовые операции, команды, внутренние функции и их последствия. Это не эксплуатационный runbook. |
| `14_test_strategy.md` | `testing` | Описывает стратегию проверки модуля: unit, integration, contract, E2E, тестовые данные и критерии приемки. |
| `15_navigation_menu.md` | `design` | Описывает пункты главного меню модуля и связь пунктов меню с UI-представлениями. |
| `backlog.md` | `appendix` | Хранит отложенные вопросы и задачи, которые не являются утвержденным проектным решением. |

### 10.1 `00_module_overview.md`

```markdown
# Обзор модуля — <Название модуля>

## 1. Назначение модуля
## 2. Бизнес-возможность
## 3. Основные пользователи
## 4. Основные сценарии
## 5. Внешние зависимости
## 6. Зависимости от платформенного ядра
## 7. Основные ObjectTypeCode
## 8. Использование Object Runtime
## 9. Основные риски и открытые вопросы
```

### 10.2 `01_scope.md`

```markdown
# Область действия — <Название модуля>

## 1. Входит
## 2. Не входит
## 3. Границы с соседними модулями
## 4. Владение данными
## 5. Интеграционные границы
```

### 10.3 `02_domain_model.md`

```markdown
# Доменная модель — <Название модуля>

## 1. Ключевые сущности
## 2. Корни агрегатов
## 3. Value objects (объекты-значения)
## 4. Доменные инварианты
## 5. Состояния жизненного цикла
## 6. Область tenant
## 7. Точки расширения
## 8. Владение данными
## 9. Связи с другими модулями
## 10. Связь с Object Runtime
```

Правило:

> Прикладной модуль владеет своими бизнес-сущностями. Платформенные сервисы не владеют прикладными основными данными.

### 10.4 `03_object_runtime_model.md`

```markdown
# Runtime-модель объектов — <Название модуля>

## 1. Объекты модуля

| ObjectTypeCode | Бизнес-смысл | Владелец данных | Стандартная обработка Object Runtime? | Примечание |
|---|---|---|---:|---|

## 2. Поля, ссылки и коллекции
## 3. Представление объектов
## 4. Lookup, поиск и фильтрация
## 5. Runtime-действия и обработчики

| ObjectTypeCode | Runtime-действие или операция | Кто вызывает | Правила и проверки | Workflow | Аудит |
|---|---|---|---|---|---|

## 6. Runtime-публикация пакетных операций

| Операция | Объем данных | Какие правила обязательны | Какие правила можно отключать | Контроль последствий |
|---|---:|---|---|---|

## 7. Особые обработчики и исключения

| Сценарий | Почему не хватает стандартной обработки | Как сохраняется общий контракт Object Runtime |
|---|---|---|

## 8. Текущие ограничения реализации
## 9. Открытые решения
```

Правило:

> Документ прикладного модуля описывает только то, как модуль использует Object Runtime. Общая архитектура Object Runtime документируется в `03_platform/03_object_runtime/`.

### 10.5 `04_workflows.md`

```markdown
# Workflow — <Название модуля>

## 1. Типы объектов с workflow

| ObjectTypeCode | WorkflowCode | InitialState | Notes |
|---|---|---|---|

## 2. Действия / команды

| ActionCode | Meaning | User-facing? | Domain Handler |
|---|---|---:|---|

## 3. Состояния
## 4. Правила переходов
## 5. Точки выполнения
## 6. Назначения и права
## 7. Требования к истории workflow
## 8. Открытые решения
```

### 10.6 `05_rules.md`

```markdown
# Правила — <Название модуля>

## 1. Жестко заданные инварианты
## 2. Настраиваемые правила

| RuleCode | RuleType | ExecutionPointCode | Parameters | Result |
|---|---|---|---|---|

## 3. Контекст правила
## 4. Поведение при ошибке
## 5. Требования к трассировке и аудиту
```

### 10.7 `06_ui_views.md`

```markdown
# UI-представления — <Название модуля>

## 1. Индекс представлений

| ViewCode | ViewType | ObjectTypeCode | Purpose |
|---|---|---|---|

## 2. Формы объектов
## 3. Списки объектов
## 4. Дашборды
## 5. Рабочие области
## 6. Представление действий
## 7. Фильтры
## 8. Видимость по ролям
## 9. Локализация
```

### 10.8 `07_reports_outputs.md`

```markdown
# Отчеты и выходные формы — <Название модуля>

## 1. Наборы данных

В этом разделе перечисляются используемые `DatasetCode` и источники данных.
Это не означает, что в платформе уже есть отдельный configuration artifact
`Dataset`. Для object datasets источник истины - `BusinessObjectContract<T>` и
Module Registration. Отдельные standalone/configurable datasets требуют
отдельного проектного решения и, вероятно, программного `DatasetContract<T>` или
аналогичного контракта.

| DatasetCode | Вид набора данных | Владелец | Источник истины | Используется в | Поля/результат |
|---|---|---|---|---|---|

Виды наборов данных:

- Object List;
- Object Lookup;
- Object Details;
- Report/Analytics;
- Integration;
- External;
- Planning/Optimization.

## 2. Метрики
## 3. Отчеты
## 4. Выходные формы и экспорт
## 5. Параметры и фильтры
## 6. Требования к валидации
```

### 10.9 `08_api_contracts.md`

```markdown
# API-контракты — <Название модуля>

## 1. Публичный API
## 2. Внутренний API
## 3. Команды
## 4. Запросы
## 5. Модель ошибок
## 6. Правила совместимости
## 7. Связанные OpenAPI-файлы
```

### 10.10 `09_events.md`

```markdown
# События — <Название модуля>

## 1. Публикуемые события

| EventTypeCode | Trigger | PayloadVersion | Consumers |
|---|---|---|---|

## 2. Потребляемые события

| EventTypeCode | Source | Handling | Idempotency Key |
|---|---|---|---|

## 3. Схемы payload
## 4. Повторы и поведение при ошибках
```

### 10.11 `10_value_set_data_usage.md`

```markdown
# Использование ValueSetData — <Название модуля>

## 1. Value sets

| ValueSetCode | Где используется | Политика corporate/tenant |
|---|---|---|

## 2. Классификационные наборы
## 3. Наборы значений типа enum
## 4. Привязки полей
## 5. Open Decisions
```

### 10.12 `11_permissions.md`

```markdown
# Права доступа — <Название модуля>

## 1. Коды прав доступа

| PermissionCode | Resource | Action | Область |
|---|---|---|---|

## 2. Рекомендации по ролям
## 3. Авторизация API
## 4. Видимость в UI
## 5. Авторизация workflow-действий
```

### 10.13 `12_audit_history.md`

```markdown
# Аудит и история — <Название модуля>

## 1. Требования к аудиту
## 2. История объектов
## 3. История workflow
## 4. Техническая трассировка
## 5. Сроки и правила хранения
```

### 10.14 `13_operations.md`

```markdown
# Прикладные операции и алгоритмы — <Название модуля>

## 1. Операции создания и изменения
## 2. Пользовательские и системные команды
## 3. Внутренние прикладные функции
## 4. Копирование, пересчет и генерация данных
## 5. Массовые операции
## 6. Предусловия и постусловия
## 7. Транзакционные границы и побочные эффекты
## 8. Связь с Object Runtime actions
## 9. Открытые решения
```

Правило:

> `13_operations.md` описывает прикладные операции, внутренние прикладные функции и алгоритмы модуля. Эксплуатационные инструкции, мониторинг, оповещения и runbooks ведутся у владельцев; `07_operations/` создаётся только для сквозных platform-wide runbooks.

### 10.15 `14_test_strategy.md`

```markdown
# Тестовая стратегия — <Название модуля>

## 1. Unit-тесты
## 2. Интеграционные тесты
## 3. Контрактные тесты
## 4. E2E-сценарии
## 5. Тестовые данные
## 6. Критерии приемки
```

### 10.16 `15_navigation_menu.md`

```markdown
# Меню навигации — <Название модуля>

## 1. Назначение документа
## 2. Источник меню
## 3. Правила публикации меню
## 4. Структура меню
## 5. Представления без пункта меню
## 6. Связь с правами
```

Документ описывает только навигацию главного меню: группы, пункты меню, целевые представления, порядок и основание.

Состав списков, карточек, вкладок, фильтров и списков выбора описывается в `06_ui_views.md`.

Права доступа описываются в `11_permissions.md`; документ меню только указывает, что пункт меню отображается при наличии права просмотра целевого объекта.

---

## 11. Правила для большого числа прикладных модулей

### 11.1 Индекс модулей обязателен

Файл:

```text
04_domain_modules/01_module_index.md
```

Содержит таблицу по всем прикладным модулям из утвержденных функциональных требований:

| ModuleNo | ModuleCode | ModuleFolder | Название | Владелец | Статус | Основные документы | Сервисы | Примечания |
|---|---|---|---|---|---|---|---|---|
| 01 | general_master_data | 01_general_master_data | Общая НСИ и основные данные / General Master Data | module:general_master_data | draft | link | уточняется | |
| 02 | product_process_definition | 02_product_process_definition | Составы и технологии / Product & Process Definition | module:product_process_definition | draft | link | уточняется | Единый модуль по требованиям |
| 03 | plant_structure | 03_plant_structure | Производственная структура / Plant Structure | module:plant_structure | draft | link | уточняется | |
| 04 | resource_management | 04_resource_management | Управление ресурсами / Resource Management | module:resource_management | draft | link | уточняется | |
| 05 | document_management | 05_document_management | Управление документацией / Document Management | module:document_management | draft | link | уточняется | |
| 06 | project_management | 06_project_management | Управление проектами / Project Management | module:project_management | draft | link | уточняется | |
| 07 | order_management | 07_order_management | Управление заказами / Order Management | module:order_management | draft | link | уточняется | |
| 08 | planning_scheduling | 08_planning_scheduling | Планирование и расписания / Advanced Planning & Scheduling | module:planning_scheduling | draft | link | уточняется | |
| 09 | production_logistics | 09_production_logistics | Производственная логистика / Production Logistics | module:production_logistics | draft | link | уточняется | |
| 10 | shopfloor_execution | 10_shopfloor_execution | Управление операциями / Shopfloor Operations Management | module:shopfloor_execution | draft | link | уточняется | |
| 11 | quality_management | 11_quality_management | Управление качеством / Quality Management | module:quality_management | draft | link | уточняется | |
| 12 | machine_data_collection | 12_machine_data_collection | Мониторинг оборудования (MDC) / Machine Data Collection | module:machine_data_collection | draft | link | уточняется | |
| 13 | manufacturing_analytics | 13_manufacturing_analytics | Производственная аналитика (BI) / Manufacturing Analytics | module:manufacturing_analytics | draft | link | уточняется | |

### 11.2 Минимальный набор документов на модуль

Обязательный минимум перед стартом разработки модуля:

```text
00_module_overview.md
01_scope.md
02_domain_model.md
03_object_runtime_model.md
08_api_contracts.md
09_events.md
11_permissions.md
14_test_strategy.md
```

Если применимо, добавляются:

```text
04_workflows.md
05_rules.md
06_ui_views.md
07_reports_outputs.md
10_value_set_data_usage.md
12_audit_history.md
13_operations.md
15_navigation_menu.md
```

### 11.3 Готовность модуля к разработке

Перед началом разработки модуля должно быть готово:

```text
- обзор модуля;
- область действия: что входит и что не входит;
- ссылка на раздел функциональных требований и список REQ/LIM;
- доменная модель;
- владение данными;
- список ObjectTypeCode;
- описание связи объектов модуля с Object Runtime;
- список операций изменения объектов и пакетных операций;
- требуемые возможности платформенного ядра;
- черновик API;
- черновик событий;
- черновик прав доступа;
- использование ValueSetData;
- тестовая стратегия.
```

### 11.4 Границы модуля

Каждый модуль обязан явно отвечать:

```text
- какими данными модуль владеет;
- какие данные читает через контракты;
- какие события публикует;
- какие события потребляет;
- какие платформенные сервисы использует;
- какие workflow подключает;
- какие правила вызывает;
- какие операции идут через Object Runtime;
- какие операции требуют особого обработчика или исключения;
- какие права доступа нужны;
- какие отчеты и выходные формы предоставляет.
```

### 11.5 Трассируемость к требованиям

Для каждого прикладного модуля в `00_module_overview.md` или `01_scope.md` должен быть раздел `Связь с требованиями`.

Минимальный формат:

```markdown
## Связь с требованиями

| ID требования | Тип | Краткий смысл | Покрытие документами |
|---|---|---|---|
| REQ-07-B-001 | базовое требование | Ведение внешних и внутренних заказов | 01_scope.md, 02_domain_model.md |
| LIM-07-O-001 | ограничение | Подбор допустимых опций требует отдельной проработки | 01_scope.md, backlog.md |
```

Правила:

1. Требование не переписывается свободным текстом без сохранения `Requirement ID`.
2. Если требование пока не проработано, оно остается в таблице со статусом `open` или ссылкой на backlog.
3. Ограничения `LIM-*` документируются вместе с требованиями, чтобы границы модуля не терялись при детализации.
4. Если модуль использует Platform Core, это не снимает с него обязанности описать бизнес-смысл требования и владение данными.

---

## 12. Именование файлов

Использовать:

```text
lowercase_snake_case.md
```

Примеры:

```text
platform_core.md
configuration_platform.md
tenant_and_security_platform.md
runtime_conventions.md
```

Нумерация используется для порядка чтения:

```text
00_overview.md
01_scope.md
02_domain_model.md
04_workflows.md
```

Смысловая идентичность задается не номером файла, а `id` во frontmatter.

---

## 13. Ссылки между документами

Использовать относительные ссылки:

```markdown
См. [Configuration](../03_platform/02_configuration/00_platform_overview.md)
```

Каждый нормативный документ должен иметь раздел:

```markdown
## Связанные документы

- [Platform Area Template](../03_platform/00_platform_documentation_template.md)
- [Object Runtime](../03_platform/03_object_runtime/00_platform_overview.md)
```

Нельзя писать неявно:

```text
как описано выше
как мы обсуждали
в предыдущем документе
```

Нужно писать конкретно:

```text
См. `03_platform/02_configuration/04_runtime.md`.
```

---

## 14. ADR

ADR не является обязательным типом документа для текущего комплекта
`docs-new`. Основной путь фиксации решений — документы-владельцы,
`90_traceability.md` и `10_backlog`. Отдельный ADR создаётся только после
явного решения вести самостоятельное архитектурное решение с владельцем,
статусом, последствиями и ссылками из документов-владельцев.

ADR уместен, если отдельного документа-владельца и трассировки недостаточно, а
решение:

- влияет на архитектуру;
- меняет контракт;
- меняет границы ответственности;
- задаёт долгосрочное ограничение;
- выбирает один вариант из нескольких;
- может вызвать повторные споры.

Шаблон:

```markdown
---
id: ADR-0001
title: Use docs-as-code for project documentation
type: adr
status: approved
owner: architecture
reviewers: []
scope: documentation
version: 1.0
last_reviewed: 2026-05-15
supersedes: []
---

# ADR-0001 — Use docs-as-code for project documentation

## Status
Approved

## Context
Describe the problem and constraints.

## Decision
Describe the chosen decision.

## Consequences

### Positive
### Negative
### Risks

## Alternatives Considered

## Связанные документы
```

---

## 15. Процесс изменения документации

```text
Запрос на изменение / задача
  → ветка
  → правка Markdown
  → локальный предпросмотр
  → pull request
  → проверка
  → CI-проверки
  → merge
  → публикация документационного портала
```

### 15.1 Кто утверждает

| Тип документа | Кто меняет | Кто утверждает |
|---|---|---|
| Concept | analyst / architect | product owner / architect |
| Architecture | architect / tech lead | architecture owner |
| Platform design | architect / senior engineer | platform owner |
| Module spec | analyst / module lead | module owner |
| API-контракт | backend lead | architecture / consumers |
| Контракт события | backend/integration lead | architecture / consumers |
| Соглашение о поведении во время выполнения | architect / tech lead | architecture board |
| Operations | DevOps / SRE | operations owner |
| Testing | QA lead | QA owner |
| ADR | architect / tech lead | architecture owner |

### 15.2 PR-шаблон для документации

```markdown
## Изменение документации

### Тип изменения

- [ ] Новый документ
- [ ] Обновление существующего документа
- [ ] Вывод из использования
- [ ] ADR
- [ ] Обновление контракта
- [ ] Обновление спецификации модуля

### Область действия

- [ ] Концепция
- [ ] Архитектура
- [ ] Платформа
- [ ] Object Runtime
- [ ] Прикладной модуль
- [ ] Контракт
- [ ] Поведение во время выполнения
- [ ] Эксплуатация
- [ ] Тестирование

### Влияние

- [ ] API-контракт изменен
- [ ] Контракт события изменен
- [ ] Стабильные коды изменены
- [ ] Модель конфигурации изменена
- [ ] Контракт Object Runtime изменен
- [ ] Поведение workflow изменено
- [ ] Поведение пакетных операций изменено
- [ ] Безопасность или права изменены
- [ ] Нет влияния на поведение во время выполнения

### Проверки

- [ ] Ссылки проверены
- [ ] Frontmatter заполнен
- [ ] Связанные документы обновлены
- [ ] ADR добавлен или обновлен, если нужен
- [ ] Глоссарий обновлен, если добавлены новые термины
- [ ] Изменения кода связаны с документом, если применимо
```

---

## 16. Синхронизация кода и документации

Если PR меняет поведение системы, он должен явно указать влияние на документацию.

В каждый code PR добавляется:

```markdown
## Влияние на документацию

- [ ] Архитектурная документация обновлена
- [ ] API-контракты обновлены
- [ ] Контракты событий обновлены
- [ ] Соглашения о поведении во время выполнения обновлены
- [ ] Документация модуля обновлена
- [ ] Нет влияния на документацию

Причина:
```

Документацию обязательно обновлять, если меняются:

- публичный API;
- payload события;
- стабильный код;
- модель прав;
- контракт Object Runtime;
- состав правил и проверок при изменении бизнес-объектов;
- возможность отключения части правил или проверок при пакетных операциях;
- семантика workflow-действия;
- вид конфигурации;
- поведение переопределений;
- поведение tenant/security;
- pipeline выполнения;
- модель аудита;
- жизненный цикл доменного объекта;
- поведение внешней интеграции;
- контракт UI-конфигурации;
- контракт набора данных, отчета или выходной формы.

---

## 17. Автоматические проверки

### 17.1 Минимальный CI

```text
1. Сборка документационного портала
2. Проверка битых ссылок
3. Проверка синтаксиса Markdown
4. Проверка обязательных полей frontmatter
5. Проверка дублирующихся ID документов
6. Проверка Mermaid-диаграмм
7. Проверка ссылок на удаленные файлы
8. Проверка запрещенных терминов и согласованности с глоссарием
```

### 17.2 Инструменты

| Проверка | Инструмент |
|---|---|
| Сборка документации | MkDocs |
| Битые ссылки | mkdocs build --strict / lychee |
| Стиль Markdown | markdownlint |
| Языковой стиль | Vale |
| Валидация YAML | yamllint |
| Валидация JSON Schema | ajv |
| Валидация OpenAPI | spectral |
| Проверка рендера Mermaid | mermaid-cli |
| Дублирующиеся ID | custom script |
| Проверка реестра стабильных кодов | custom script |

### 17.3 Проверки frontmatter

CI должен проверять наличие:

```text
id
title
type
status
owner
reviewers
scope
version
```

Для module docs дополнительно желательно проверять:

```text
module
```

### 17.4 Проверки дрейфа архитектуры и документации

Постепенно добавить проверки:

```text
- PermissionCode есть в коде, но отсутствует в permission-codes.yaml
- ActionCode используется в workflow, но не описан в module workflows
- EventTypeCode публикуется сервисом, но нет event schema
- DatasetCode используется UI/reporting/output, но не зарегистрирован в
  runtime/registry contract
- ObjectTypeCode есть в registry, но не описан в module domain model
- ObjectTypeCode есть в реестре, но не описана связь с Object Runtime
- Пакетная операция есть в коде, но не описана в `13_operations.md`; если она доступна через Object Runtime, также нет описания в `03_object_runtime_model.md`
- API endpoint есть в OpenAPI, но нет текстового описания
- ConfigurationKind используется в коде, но не описан в platform docs
```

---

## 18. Каталог стабильных кодов

Стабильные коды ведутся в документах владельцев. Владелец кода отвечает за
смысл, совместимость, миграцию и ссылки на реализацию. Общий каталог стабильных
кодов в `docs-new` не создаётся без отдельного решения о формате и владельце
такого каталога.

Пример:

```yaml
codes:
  - code: OrderManagement.ProductionOrder
    type: ObjectTypeCode
    owner: order_management
    status: approved
    description: Production order aggregate root

  - code: ProductionOrderWorkflow
    type: WorkflowCode
    owner: order_management
    status: approved
    description: Main workflow for production orders
```

Правило:

> Стабильный код нельзя переименовывать без решения о миграции и ADR.

---

## 19. Глоссарий

Глоссарий должен содержать:

```text
Term
Preferred English name
Preferred Russian name
Definition
Allowed synonyms
Forbidden synonyms
Related codes
Related documents
```

### 19.1. Единый формат тематических глоссариев

Новые и обновляемые тематические глоссарии используют единый frontmatter документа и общую базовую таблицу терминов:

| Русский термин | Preferred English | Технический алиас | Статус | Определение | Источник в коде | Нежелательные синонимы |
| --- | --- | --- | --- | --- | --- | --- |

Эта таблица отображает содержательный минимум раздела 19: русский и английский preferred names, технический алиас, определение, источник и ограничения употребления. Допустимые синонимы, связанные коды и связанные документы указываются в дополнительных колонках или специализированных разделах тематического глоссария, если они нужны для конкретной области.

Тематический глоссарий может содержать дополнительные разделы, если они отражают его предметную границу: например, исходные термины `TERM-*`, UI-подписи, термины, требующие владельца, или границы с соседними областями. Дополнительные разделы не изменяют базовую таблицу и не создают второго источника истины. Исторические таблицы, отличающиеся от нового формата, приводятся к нему при плановом обновлении соответствующего глоссария.

Локальная таблица терминов в `00_platform_overview.md` является краткой копией для чтения пакета. Источником истины для термина остаётся тематический глоссарий, а для утверждённого `TERM-*` — исходное требование.

Пример:

```markdown
## Workflow Action

Preferred English: Workflow Action  
Preferred Russian: Команда workflow  
Forbidden: Transition button, UI action without context  
Definition: Пользовательская или системная команда, инициирующая workflow transition через ActionCode.  
Related: ActionCode, WorkflowCode, Transition
```

Если вводится новый термин, нужно:

1. Добавить его в глоссарий.
2. Проверить, не дублирует ли он существующий термин.
3. Указать preferred naming.
4. Указать forbidden synonyms, если есть риск путаницы.

---

## 20. Политика использования AI в документации

### 20.1 AI может

- писать черновики;
- предлагать структуру;
- искать противоречия;
- синхронизировать формулировки;
- генерировать ADR draft;
- сравнивать docs и code;
- делать summary изменений;
- предлагать glossary updates.

### 20.2 AI не должен

- менять approved-документы без PR;
- принимать архитектурные решения без владельца;
- тихо переименовывать стабильные коды;
- удалять ограничения области действия;
- смешивать platform и domain responsibilities;
- превращать черновик в утвержденный документ без проверки;
- изменять контракты без анализа совместимости.

### 20.3 Prompt-шаблон для AI

```text
Ты работаешь с документацией Digital Manufacturing Platform.

Задача:
[описать задачу]

Ограничения:
- не менять стабильные коды без явного указания;
- не смешивать platform и domain responsibilities;
- сохранять текущую структуру документа, если не сказано иначе;
- все новые термины вынести в glossary candidates;
- все спорные решения вынести в Open Decisions;
- если изменение требует ADR, явно указать это.

Результат:
- summary изменений;
- список измененных файлов;
- список open questions;
- список документов, которые нужно синхронизировать.
```

---

## 21. Готовность документа к утверждению

Документ готов, если:

```text
- заполнен frontmatter;
- есть владелец;
- определен status;
- указана область действия;
- есть раздел “Входит / Не входит”, если применимо;
- есть related documents;
- ссылки работают;
- термины соответствуют glossary;
- стабильные коды вынесены в каталог, если применимо;
- open decisions явно перечислены;
- документ прошел проверку;
- CI-проверки пройдены;
- published portal успешно собран.
```

---

## 22. Жизненный цикл документа

```text
Draft
  → На проверке
  → Approved
  → Deprecated
  → Superseded / Archived
```

| From | To | Условие | Результат для `version` |
|---|---|---|---|
| `draft` | `in-review` | Автор считает документ готовым к проверке | Следующая версия `0.x` |
| `draft` | `approved` | PR слит до синхронизации рабочего статуса | `1.0` |
| `in-review` | `approved` | PR слит в `master` | `1.0` |
| `approved` | `deprecated` | Документ ещё применим, но планируется замена | Следующая версия `1.x` |
| `deprecated` | `superseded` | Появился заменяющий документ | `2.0` |
| Любой | `archived` | Документ сохранён только для истории | `2.0`, без понижения уже существующей версии |

Переходы, пропускающие состояния цепочки, отклоняются workflow. Исключения:
post-merge финализатор может выполнить `draft` → `approved`, поскольку сам merge
считается согласованием; архивировать документ разрешено из любого состояния.
Изменение содержания без смены `status` повышает только `MINOR` текущей линии
версии.

### 22.1 Автоматическая синхронизация с pull request

Для новых документов рабочая пара `status` / `review_status` синхронизируется с
состоянием PR без изменения версии и истории:

| Событие PR | `status` | `review_status` |
|---|---|---|
| Открыт Draft PR | `draft` | `not_started` |
| PR готов к проверке | `in-review` | `in_review` |
| Запрошен reviewer или отправлены исправления | `in-review` | `awaiting_reviewer` |
| Reviewer запросил изменения | `in-review` | `changes_requested` |

События PR не понижают нормативные статусы `approved`, `deprecated`,
`superseded` и `archived`. Approval review не создаёт отдельный commit в ветке
PR: рабочий статус меняется только по состоянию самого PR.

Факт merge в `master` считается достаточным подтверждением согласования, без
дополнительной проверки API approvals. После merge отдельный workflow
автоматически переводит изменённые документы из `draft` или `in-review` в
`approved` / `approved`, назначает первую утверждённую версию `1.0` и добавляет
строку истории со ссылкой на merged PR.

---

## 23. Ownership

Каждый документ должен иметь владельца содержания в поле `owner`. Значением
является GitHub username с `@`. Поле `holder` фиксирует держателя процесса
документирования, а `reviewers` — назначенных участников проверки. Список может
содержать любые GitHub usernames в формате `@username` либо оставаться пустым.
Наличие отдельных GitHub approvals не является техническим условием
post-merge workflow.

Пример:

```yaml
owner: '@axelprosoft'
reviewers: []
holder: '@axelprosoft'
```

---

## 24. Версионирование

Git хранит полную техническую историю изменений, а workflow автоматически ведёт
смысловую версию и таблицу истории документа.

Поле `version` во frontmatter отражает смысловую версию документа и хранится
строкой в формате `MAJOR.MINOR`.

```yaml
version: '1.2'
```

Версия связана с жизненным циклом:

```text
0.x — draft / in-review
1.x — approved / deprecated
2.x — superseded / archived
```

Переход к `2.0` означает прекращение действия прежнего нормативного контракта:
документ заменён, архивирован либо переработан через создание заменяющего
документа. Точные переходы и правила расчёта приведены в разделах 8.5 и 22.
Пользователь не меняет `version` вручную.

---

## 25. Публикация

После merge в main документация публикуется в documentation portal.

```text
main branch обновлена
  → CI build
  → checks
  → генерация статического сайта
  → публикация во внутренний портал
```

Стартово можно публиковать один портал для всей команды. Позже можно разделить:

```text
Internal engineering docs
Customer-facing docs
Operations docs
Developer portal
```

---

## 26. Порядок внедрения

### Шаг 1. Создать стратегию

```text
docs-new/00_governance/00_documentation_strategy.md
```

### Шаг 2. Создать структуру папок

```text
docs-new/
  00_governance/
  01_concept/
  02_architecture/
  03_platform/
  04_domain_modules/
  10_backlog/
  11_glossary/
  12_appendices/
```

### Шаг 3. Перенести существующие документы

Пример mapping:

```text
01_concept.md                         → 01_concept/01_product_concept.md/01_concept.md (01 sources).md
02_system_scope.md                    → 01_concept/02_system_scope.md/02_system_scope.md (01 sources).md
03_platform_core.md                   → staging-копия `03_platform/00_platform_core.md/` закрывается после переноса; общий состав Platform Core находится в `02_architecture/01_architecture_overview.md`, границы системы — в `01_concept/02_system_scope.md`, детализация capabilities — в `03_platform/*/`, Foundation — в `03_platform/00_foundation/`, правила оформления — в `03_platform/00_platform_documentation_template.md`
04_module_map.md                      → 04_domain_modules/01_module_index.md or 01_concept/module_map.md
05_architecture.md                    → 02_architecture/01_architecture_overview.md/05_architecture.md (01 sources).md
06_configuration_platform.md          → 03_platform/02_configuration/00_platform_overview.md, 01_scope.md, 90_traceability.md и последующие документы-владельцы
07_tenant_and_security_platform.md    → 01_concept / 02_architecture / 03_platform/01_tenant_and_security decisions, без source-копии внутри Tenant/Security
10_value_set_data.md                  → 03_platform/06_value_sets/ после вычитки и трассировки
11_integration_event_platform.md      → 03_platform/10_integration_events/ после вычитки и трассировки
12_audit_history_platform.md          → 03_platform/09_audit_history/ после вычитки и трассировки
13_platform_api_and_contracts.md      → 03_platform/00_foundation/03_contracts.md + 03_platform/03_object_runtime/03_contracts.md + 10_backlog/contract_governance/platform_api_contracts_backlog.md; staging-копия не сохраняется
14_platform_runtime_conventions.md    → не создавать отдельную source-копию; подтверждённые сведения ведут Foundation, Tenant/Security, Object Runtime и документы владельцев `03_platform/*/04_runtime.md`
14.1_platform_runtime_reference_patterns.md → не создавать отдельную source-копию; reference-примеры не являются нормативным проектным документом
15_mvp_foundation_plan.md             → 10_backlog/roadmap/preparation/09_two_week_platform_architecture_outcome_requirements.md + platform_core_documentation_backlog.md; source-копия в `docs-new` не сохраняется
```

Для каждого перенесённого файла сохраняются исходное имя и источник в скобках: `имя-файла (имя-папки-источника).расширение`. Папка `offers/` создаётся только в родительском целевом разделе, если один исходный файл нужно разделить по нескольким дочерним документам. Для однозначного конечного размещения source-копия сразу помещается в конечный узел без `offers/`.

### Шаг 4. Добавить frontmatter

Добавить YAML frontmatter во все нормативные документы.

### Шаг 5. Настроить MkDocs

Минимальный `mkdocs.yml`:

```yaml
site_name: Digital Manufacturing Platform Documentation
site_description: Architecture and project documentation
repo_url: https://example.com/repo

theme:
  name: material
  features:
    - navigation.sections
    - navigation.expand
    - navigation.indexes
    - search.highlight
    - content.code.copy

markdown_extensions:
  - admonition
  - toc:
      permalink: true
  - tables
  - fenced_code
  - pymdownx.superfences
  - pymdownx.details
  - pymdownx.tabbed
  - pymdownx.tasklist

plugins:
  - search
```

### Шаг 6. Настроить CI

Минимально:

```bash
mkdocs build --strict
markdownlint docs-new/**/*.md
yamllint docs-new/**/*.yaml docs-new/**/*.yml
```

Позже:

```bash
spectral lint docs-new/**/*.yaml
ajv validate -s schema.json -d docs-new/**/*.json
lychee docs-new/**/*.md
```

### Шаг 7. Ввести PR-template

Добавить PR-template для документации и code PR.

### Шаг 8. Решить, нужны ли первые ADR

```text
0001-use-docs-as-code.md
0002-use-markdown-as-source-format.md
0003-use-mkdocs-material-for-documentation-portal.md
0004-use-frontmatter-for-document-metadata.md
0005-use-pull-request-review-for-doc-changes.md
```

Эти файлы создаются только при отдельном решении вести самостоятельные ADR.

### Шаг 9. Создать module template

```text
04_domain_modules/00_module_documentation_template.md
```

---

## 27. MVP документационной системы

Для первого запуска достаточно:

```text
- структура docs-new/;
- Markdown documents;
- frontmatter;
- MkDocs Material;
- build --strict;
- broken links check;
- PR-template;
- ADR template;
- module documentation template;
- тематические глоссарии `11_glossary/*_terms.md`;
- module index;
- черновик каталога стабильных кодов.
```

Не обязательно сразу внедрять:

```text
- Backstage;
- Antora;
- сложный style guide;
- автоматическое выявление дрейфа между документацией, контрактами и кодом;
- полное управление контрактами;
- сложную публикацию портала с разграничением по ролям.
```

---

## 28. Ключевые принципы

1. Git является источником истины для утвержденной документации.
2. Markdown является основным форматом нормативной документации.
3. Опубликованная документация собирается из файлов, прошедших проверку.
4. Каждое важное архитектурное решение фиксируется в ADR.
5. У каждого документа есть владелец, статус и область действия.
6. Каждый модуль документируется по единому шаблону.
7. Контракты должны быть машинно-читаемыми там, где это возможно.
8. Стабильные коды должны быть каталогизированы и защищены от произвольного переименования.
9. AI помогает готовить и проверять документацию, но не заменяет проверку.
10. Изменения документации являются частью инженерной работы, а не постфактум-дополнением.

---

## 29. Итоговое решение

Для Digital Manufacturing Platform стартовая стратегия документации следующая:

```text
Использовать docs-as-code.
Использовать Markdown как основной исходный формат.
Хранить утверждаемую документацию в Git.
Публиковать документацию через MkDocs Material.
Требовать PR-проверку для утверждаемых документов.
Использовать ADR для архитектурных решений.
Использовать OpenAPI, AsyncAPI и JSON Schema для контрактов.
Использовать единый шаблон документации для всех прикладных модулей.
Использовать CI-проверки, чтобы предотвращать битые ссылки, отсутствие метаданных и дрейф документации.
```

Это решение достаточно простое для старта, но масштабируется на большую команду, десятки прикладных модулей, параллельную разработку и AI-assisted editing с обязательной проверкой.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.5 | 2026-08-27 15:50 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: status; Базовая структура `docs-new/`; 1 `00_governance/`; 3 `02_architecture/`; 4 `03_platform/`; 5 `04_domain_modules/`; 6 `05_contracts/`; 7 `06_runtime/`; 6 Контракты; 8 `07_operations/`; 9 `08_testing/`; 7 Условные будущие разделы; 10 `09_decisions/`; 11 `10_backlog/`; 8 `10_backlog/`; 12 `11_glossary/`; 9 `11_glossary/`; 13 `12_appendices/`; 10 `12_appendices/`; Frontmatter документа; 1 Назначение ID по пути; 2 Допустимые `type`; 4 Как выбирать значения полей; 4 `03_object_runtime_model.md`; 14 `13_operations.md`; 1 Индекс модулей обязателен; Ссылки между документами; ADR; Каталог стабильных кодов; 1. Единый формат тематических глоссариев; Ownership; Шаг 2. Создать структуру папок; Шаг 3. Перенести существующие документы; Шаг 6. Настроить CI; Шаг 8. Создать первые ADR; Шаг 8. Решить, нужны ли первые ADR | Согласовать стратегию документации с проверками PR | [351678eb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/351678eb99dd120aebae2789e995a844bdb6b8ff) |
