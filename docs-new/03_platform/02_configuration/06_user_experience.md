---
id: DOC-03-02-06
title: 'Пользовательский слой — Configuration'
type: design
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: configuration
holder: '@axelprosoft'
created_at: 2026-08-25 17:20
created_by: '@axelprosoft'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Пользовательский слой — Configuration

[api-client]: ../../../src/Frontend/packages/api-client/src/index.ts
[contracts]: ../../../src/Frontend/packages/contracts/src/configuration.ts
[studio]: ../../../src/Frontend/apps/studio/src/features/
[context]: ../../../src/Frontend/apps/studio/src/app/useConfigurationContext.ts

## 1. Назначение документа

Документ описывает только интерфейсную часть Configuration, подтверждённую frontend-кодом: Studio, управление контекстом конфигурации, explorer и editor артефактов. Реализация общего shell, routing, session/bootstrap, shared UI и приложений описана в [фронтенд-платформе](../12_frontend_platform/00_platform_overview.md), а граница владельцев — в её [UX-документе](../12_frontend_platform/06_user_experience.md).

## 2. Frontend-контракты

Configuration-specific frontend использует server contracts области; общий shell,
session, routing и shared runtime packages принадлежат фронтенд-платформе.

| Раздел или сценарий | Представление или пакет | API | Право | Статус реализации |
| --- | --- | --- | --- | --- |
| Configuration context и version management | `context-version-management` | Context/version API и action state | Права Configuration context | Подтверждено кодом |
| Configuration Explorer | `configuration-explorer` | Explorer list/detail API | `Configuration.Catalog.View` | Подтверждено кодом |
| Artifact Editor | `configuration-artifact-editor` | Editor session, query, command и validation contracts | `Configuration.Entry.Edit` | Подтверждено кодом |
| Effective configuration | Explorer/effective projection | Effective configuration API | `Configuration.Effective.View` | Подтверждено кодом |

## 3. Разделы интерфейса и сценарии

| Сценарий | Интерфейсная часть | Серверный контракт |
| --- | --- | --- |
| Выбор configuration context | Studio context state и context operations. | Configuration context API. |
| Просмотр scope и version | Context Version Management и список версий. | Context/version responses и action state. |
| Просмотр каталога | Configuration Explorer. | Explorer list/detail API. |
| Редактирование артефакта | Configuration Artifact Editor, дерево и поля узлов. | Editor session, query, command и validation contracts. |
| Работа с draft | Create, discard, validate, publish, rebase и другие действия, если они разрешены action state. | Configuration context action contracts. |
| Просмотр эффективной конфигурации | Обозреватель конфигурации и effective projection. | Effective configuration API. |

Реальные API-вызовы и типы находятся в [frontend API client][api-client] и [frontend contracts][contracts].

## 4. Навигация

В текущем коде Configuration-specific функции находятся в `src/Frontend/apps/studio`:

- `context-version-management` отвечает за scope/version context и представление доступных действий;
- `configuration-explorer` отвечает за каталог и инспектор конфигурации;
- `configuration-artifact-editor` отвечает за чтение и изменение дерева артефакта;
- `common/config` содержит тексты и представления configuration-specific состояний.

Наличие Configuration capability в Admin для управления ролями и permissions не делает Admin владельцем Configuration UX: Admin использует общий Tenant/Security workflow назначения доступа.

| Пункт навигации | Родитель | Цель | Право | Порядок | Источник |
| --- | --- | --- | --- | --- | --- |
| Configuration Explorer | Configuration context | Explorer configuration scopes и artifacts | `Configuration.Catalog.View` | Определяется приложением | [Studio features][studio], [frontend contracts][contracts] |
| Configuration Artifact Editor | Configuration Explorer | Редактор выбранного артефакта | `Configuration.Entry.Edit` | После выбора артефакта | [Studio features][studio], [frontend contracts][contracts] |
| Context Version Management | Configuration context | Управление scope/version и действиями контекста | Права Configuration context | До explorer/editor | [configuration context][context] |

### 4.1. Переходные названия интерфейсных моделей

В UX-источниках `Navigation`, `Lookup`, `GridUi` и `EnumPresentation` могут
обозначать экран, компонент или целевую модель, но это не означает наличие
одноимённого корневого Configuration-артефакта. В текущем контракте:

- `Menu` поставляет иерархические `NavigationItem`, а их отображение и
  маршрутизация выполняются frontend-потребителем;
- `Lookup` является сценарием выбора значения, который использует `View`, в
  том числе `LookupListView`, а не самостоятельным артефактом;
- `GridUi` описывается только подтверждёнными узлами `View` и не вводит
  отдельный универсальный grid-контракт;
- `EnumPresentation` не является зарегистрированным типом; неподтверждённые
  варианты представления не считаются частью MVP.

Полная классификация переходных материалов и их владельцев приведена в
[границе области](01_scope.md), раздел 4.1.

## 5. Локализация

| Ресурс | Ключ | Язык | Резервное значение | Владелец | Покрытие |
| --- | --- | --- | --- | --- | --- |
| Configuration context | Configuration-specific context/version texts | `ru-RU`, `en-US` | Техническое имя или серверное значение | Configuration frontend feature | Подтверждено частично; полный каталог проверяется отдельно |
| Explorer/editor | Configuration-specific labels and states | `ru-RU`, `en-US` | Технический code или API label | Configuration frontend feature | Подтверждено частично |

## 6. Ограничения

Интерфейс отображает и включает операции по состоянию, которое возвращает Configuration context API. В частности, `CreateDraft` не должен быть доступен, если для non-root scope отсутствует опубликованная parent-base version. Это правило дополнительно проверяется authoring API и не считается только frontend-ограничением.

В пользовательском тексте используются `конфигурация`, `область конфигурации`, `версия`, `черновик`, `опубликованная версия`, `эффективная конфигурация`, `обозреватель конфигурации` и `редактор артефакта`. Английские code names (`Configuration`, `ConfigurationScope`, `ConfigurationVersion`, `ConfigurationArtifactEditor`) сохраняются в техническом описании. Термины сверяются с [configuration glossary](../../11_glossary/configuration_terms.md).

- Общая модель frontend apps, shell, routing и shared runtime packages описана в отдельной [фронтенд-платформе](../12_frontend_platform/00_platform_overview.md), хотя часть целевых возможностей ещё не подтверждена кодом.
- Не все source UI requirements подтверждены текущим кодом; они остаются материалом трассировки, а не обещанием MVP.
- Доступность действий в UI должна оставаться производной от server action state, а не от самостоятельной frontend-логики.

Граница общей фронтенд-платформы и Configuration-specific UX ведётся в [трассировке Configuration](90_traceability.md): `CFG-DEC-05` — Frontend Studio boundary. Этот документ фиксирует только подтверждённые Configuration-сценарии, а общая frontend-модель описана в `12_frontend_platform`.

### 6.1. Источники подтверждения

Основные подтверждения: [API client][api-client], [frontend contracts][contracts], [Studio features][studio] и [configuration context][context].

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
