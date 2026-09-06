---
id: DOC-03-00-01
title: 'Граница платформенной области — Foundation'
type: scope
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: foundation
holder: '@axelprosoft'
created_at: 2026-08-26 00:00
created_by: '@codex'
updated_at: 2026-08-27 16:52
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Граница платформенной области — Foundation

[domain-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Domain/
[application-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/
[common-contracts]: ../../../src/Platform/DMP.Platform.Contracts/Common/
[strategy]: ../../00_governance/00_documentation_strategy.md
[template]: ../00_platform_documentation_template.md

## 1. Назначение документа

Документ определяет, какие общие технические примитивы и application-контракты
принадлежат Foundation, а какие сведения должны описываться в платформенных
областях, доменных модулях или общей архитектуре системы.

## 2. Что входит

В Foundation входят:

- проекты `DMP.BuildingBlocks.Domain` и `DMP.BuildingBlocks.Application`;
- минимальные примитивы сущности, tenant-владения, аудита и агрегата;
- интерфейсы tenant/site scope и доменного события;
- контексты выполнения, доступа, языка и корреляции;
- регистрация доменного модуля и стабильных контрактных кодов;
- базовые контракты `IUseCase`, `IUnitOfWork`, результатов и структурированных ошибок;
- узкие application-порты к платформенным сервисам;
- универсальные операции нормализации списка, сортировки, локализации и разрешения reference display;
- общая часть `DMP.Platform.Contracts.Common`, если контракт используется несколькими областями.

## 3. Что не входит

Foundation не владеет:

- бизнес-сущностями и предметными инвариантами Tenant Security или доменных модулей;
- схемами, версиями, baseline и effective configuration;
- исполнением Object Runtime, Workflow, Rules, Value Sets, Settings, Numbering;
- хранением аудита и outbox, даже если Foundation задаёт порт их вызова;
- HTTP-контрактами конкретной платформенной области;
- event payload конкретной интеграционной capability;
- frontend shell, маршрутами, приложениями и рендерер;
- persistence-схемой и миграциями потребляющих модулей;
- собственной предметной моделью `00_common`.

## 4. Граница с соседними областями и модулями

| Соседний владелец | Foundation предоставляет | Соседний владелец сохраняет за собой |
| --- | --- | --- |
| Tenant Security | Контракты контекста, scope и authorization port. | Модель Tenant/Site/User, policy доступа и security manifest. |
| Configuration | Общие типы сообщений, запросов и application-контракты. | `ArtifactDocument`, схемы артефактов, публикацию и effective configuration. |
| Object Runtime | `ModuleExecutionContext`, результаты use-case, общие helper-правила и gateway-порты. | Descriptor, object contract, mutation pipeline и исполнение объектных операций. |
| Workflow / Rules | Форму вызова через application-порт. | Состояния, переходы, правила и их результаты. |
| Audit History / Integration Events | Порт аудита или событий, если он используется потребителем. | Запись аудита, outbox, доставку, replay и event envelope. |
| фронтенд-платформа | Общие transport-типы только при доказанной повторной потребности. | UI-модель, shell, приложения и клиентскую реализацию. |
| Domain Modules | Базовые сущности, контекст и общие conventions. | Бизнес-смысл, агрегаты, данные, правила и жизненный цикл. |

Foundation владеет формой общего порта, но не обязанностью каждого сервиса
реализовать все вложенные gateway. Потребитель обязан явно указать, какие порты
он использует и как ограниченная или no-op реализация влияет на его сценарий.

## 5. Соответствие требованиям

| Требование | Покрытие | Документ-владелец |
| --- | --- | --- |
| Единый tenant-aware контекст сценария | Реализовано с ограничениями текущей модели tenant/site | `02_architecture.md`, `03_contracts.md` |
| Запрет локальных альтернатив общим use-case и stable-code контрактам | Контракты и архитектурные правила описаны; conformance каждого модуля проверяется отдельно | `03_contracts.md`, документы модулей |
| Общий путь для platform service calls | Gateway-порт есть; полная реализация capability принадлежит соседним областям | `03_contracts.md`, `90_traceability.md` |
| Единые результаты и ошибки | Реализованы `UseCaseResult<T>` и `BulkUseCaseResult<TItem>` | `03_contracts.md`, `04_runtime.md` |

## 6. Ограничения версии

- Foundation не является отдельным процессом или HTTP-сервисом.
- `DMP.BuildingBlocks.Infrastructure` не имеет самостоятельного поведения в текущем снимке.
- Наличие интерфейса в Foundation не доказывает, что соответствующий platform service подключён в production composition root.
- `IUnitOfWork` задаёт только application-порт сохранения; полная транзакционная политика принадлежит владельцу операции.
- Вынесение типов между `BuildingBlocks` и `Platform.Contracts` не является частью текущего документа; необходимость такого изменения записана в [трассировке](90_traceability.md).

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 16:52 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | docs-new: синхронизировать типы документов со стандартом | [7c7ecbfb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7c7ecbfb4036c9ce3c6304f9ee3e4a6e48f7828c) |
