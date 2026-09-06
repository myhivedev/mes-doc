---
id: DOC-03-12-00
title: 'Обзор платформенной области — Фронтенд-платформа'
type: design
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: frontend_platform
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Обзор платформенной области — Фронтенд-платформа

[frontend-workspace]: ../../../src/Frontend/
[frontend-package]: ../../../src/Frontend/package.json
[frontend-workspace-file]: ../../../src/Frontend/pnpm-workspace.yaml
[admin-app]: ../../../src/Frontend/apps/admin/
[runtime-app]: ../../../src/Frontend/apps/runtime/
[studio-app]: ../../../src/Frontend/apps/studio/
[runtime-react]: ../../../src/Frontend/packages/runtime-react/
[runtime-contracts]: ../../../src/Frontend/packages/runtime-contracts/
[ui-package]: ../../../src/Frontend/packages/ui/
[shared-package]: ../../../src/Frontend/packages/shared/
[auth-package]: ../../../src/Frontend/packages/auth/
[api-client-package]: ../../../src/Frontend/packages/api-client/
[platform-context-package]: ../../../src/Frontend/packages/platform-context/
[app-runtime]: ../../../src/Frontend/apps/runtime/src/App.tsx
[runtime-main]: ../../../src/Frontend/apps/runtime/src/main.tsx
[admin-main]: ../../../src/Frontend/apps/admin/src/main.tsx
[studio-main]: ../../../src/Frontend/apps/studio/src/main.tsx
[backlog]: ../../10_backlog/roadmap/preparation/platform_core_documentation_backlog.md

## 1. Назначение области

Фронтенд-платформа описывает общие контракты фронтенда, пакеты runtime,
оболочку приложений и три платформенных приложения: `@dmp/app-admin`,
`@dmp/app-runtime` и `@dmp/app-studio`. Область фиксирует только механизмы,
которые повторно используются разными платформенными и прикладными функциями:
запуск сессии, маршрутизацию, контекст платформы, общую навигацию, отображение
настроенного runtime-представления, общие UI-компоненты и локализацию фронтенда.

Платформа не владеет предметными сценариями модулей. Например, редактор Value
Sets описывается в Value Sets, права и роли — в Tenant Security, схема
артефактов `ObjectType`, `View`, `Action` и `Menu` — в Configuration. Фронтенд-
платформа владеет тем, как общий механизм фронтенда использует эти контракты и
как приложения подключают функциональные сценарии.

### 1.1. Как читать документацию фронтенд-разработчику

Фронтенд-разработчик начинает с этого раздела и переходит к документу
функционального владельца только для конкретного экранного сценария. Читать весь
каталог платформенных областей для реализации одного экрана не требуется.

| Задача | Документы фронтенд-платформы | Документ функционального владельца |
| --- | --- | --- |
| Общая оболочка, навигация, узел маршрута и состояния интерфейса | `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `06_user_experience.md` | Не требуется |
| Экран предприятий, пользователей, ролей или прав в Admin | `02_architecture.md`, `03_contracts.md`, `06_user_experience.md` | `01_tenant_and_security/03_contracts.md`, `01_tenant_and_security/06_user_experience.md` |
| Configuration Explorer, Context или Artifact Editor в Studio | `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `06_user_experience.md` | `02_configuration/03_contracts.md`, `02_configuration/06_user_experience.md`, нужный файл `artifact_types/` |
| Редактор данных Value Sets в Studio | `02_architecture.md`, `03_contracts.md`, `06_user_experience.md` | `06_value_sets/03_contracts.md`, `06_value_sets/06_user_experience.md` |
| Каталог и редактор Settings в Studio | `02_architecture.md`, `03_contracts.md`, `06_user_experience.md` | `07_settings/03_contracts.md`, `07_settings/06_user_experience.md` |
| Настроенный экран runtime | `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `06_user_experience.md` | `02_configuration/artifact_types/view.md`, Object Runtime `03_contracts.md` и `04_runtime.md` по используемому контракту |

## 2. Место в платформе

Фронтенд-платформа является общим клиентским слоем для приложений `Admin`,
`Runtime` и `Studio`. Она предоставляет приложениям общие пакеты, оболочку,
маршрутизацию, локализацию и рендеринг настроенного пользовательского слоя.
Семантика Configuration, Object Runtime, Workflow и Tenant Security остаётся у
соответствующих владельцев.

## 3. Основные возможности

| Возможность | Назначение | Статус | Основание |
| --- | --- | --- | --- |
| Общая рабочая область фронтенда | Сборка трёх приложений и общих пакетов из `src/Frontend`. | Подтверждено MVP | [`pnpm-workspace.yaml`][frontend-workspace-file]; [`package.json`][frontend-package] |
| Оболочка и запуск приложения | Запуск сессии (`bootstrap`), `BrowserRouter`, `PlatformContextProvider`, общая навигация и базовые состояния интерфейса. | Подтверждено MVP | [`Admin`][admin-app]; [`Runtime`][runtime-app]; [`Studio`][studio-app] |
| Настроенный пользовательский слой | Получение разрешённой модели и её передача в хосты, адаптеры, провайдеры и рендереры. | Подтверждено MVP | [`runtime-react`][runtime-react]; [`runtime-contracts`][runtime-contracts] |
| Профили приложений | Общие правила и различия профилей `Admin`, `Runtime`, `Studio`. | Подтверждено MVP | Код приложений и решение `FE-DEC-08 — одна область с профилями приложений` в [`90_traceability.md`](90_traceability.md) |

## 4. Ключевые решения

| Решение | Суть | Документ-владелец |
| --- | --- | --- |
| Одна область с тремя профилями приложений | `Admin`, `Runtime` и `Studio` документируются внутри одной фронтенд-платформы. | `01_scope.md`, `90_traceability.md` |
| Разделение общего и предметного слоя | Общие механизмы: оболочка, запуск, пакеты и рендерер принадлежат фронтенд-платформе; предметные сценарии принадлежат функциональным областям. | `01_scope.md`, `06_user_experience.md` |
| Серверная семантика остаётся у владельца контракта | Фронтенд отображает полученную модель и координирует взаимодействие, но не становится источником прав, схемы или исполнения операций. | `03_contracts.md`, `05_security_and_audit.md` |

## 5. Зависимости

Платформа зависит от API серверной части и контрактов Configuration, Object Runtime,
Workflow и Tenant Security. Исходными техническими границами клиента являются
`@dmp/api-client`, `@dmp/auth`, `@dmp/platform-context`,
`@dmp/runtime-contracts`, `@dmp/runtime-react` и `@dmp/ui`. Подробные границы и
неподтверждённые решения приведены в `03_contracts.md` и `90_traceability.md`.

## 6. Статус реализации

| Слой | Код | Подтверждённое состояние |
| --- | --- | --- |
| Рабочая область | `@dmp/frontend-workspace` | `pnpm-workspace.yaml` включает `apps/*` и `packages/*`; корневые скрипты запускают lint, проверку локализации, typecheck, build, модульные тесты и e2e. ([рабочая область][frontend-workspace-file]; [пакет][frontend-package]) |
| Admin | `@dmp/app-admin` | Приложение React/Vite с запуском входа, `BrowserRouter`, `PlatformContextProvider`, оболочкой Admin, локальным/удалённым путём навигации и функциональными маршрутами. ([приложение][admin-app]; [запуск][admin-main]) |
| Runtime | `@dmp/app-runtime` | Приложение React/Vite для опубликованного runtime: точка входа, маршрут runtime, хост разрешённого представления, навигация runtime и настроенный пользовательский слой объекта. ([приложение][runtime-app]; [оболочка][app-runtime]; [запуск][runtime-main]) |
| Studio | `@dmp/app-studio` | Приложение React/Vite для авторинга конфигурации и провайдеров функций Studio. ([приложение][studio-app]; [запуск][studio-main]) |
| Контракты runtime | `@dmp/runtime-contracts` | Контракты TypeScript для настроенного пользовательского слоя, `grid`, `lookup`, `operation`, `personalization`, `workspace` и политики навигации. ([пакет][runtime-contracts]) |
| Runtime React | `@dmp/runtime-react` | Хосты, адаптеры, провайдеры, рендереры и хуки для настроенного пользовательского слоя runtime. ([пакет][runtime-react]) |
| Набор UI-компонентов | `@dmp/ui` | Общие компоненты оболочки, форм, списков и действий, типы DataTable, диалоги, навигация и компоненты разметки. ([пакет][ui-package]) |
| Общие средства | `@dmp/shared` | Средства локализации, выбор локали, действия меню пользователя и общие помощники представления. ([пакет][shared-package]) |

## 7. Состав документов

| Документ | Назначение | Статус |
| --- | --- | --- |
| `00_platform_overview.md` | Карта области, возможности, зависимости и статус. | Создан |
| `01_scope.md` | Граница общей фронтенд-платформы, трех приложений и соседних владельцев. | Создан |
| `02_architecture.md` | Рабочая область, приложения, пакеты, оболочка, отображение runtime и архитектура навигации. | Создан |
| `03_contracts.md` | Экспорты пакетов фронтенда, контракты приложений, контракты пользовательского слоя runtime и граница серверной части. | Создан |
| `04_runtime.md` | Запуск, сессия, маршрут, навигация, разрешение/отображение и сценарии списка/формы/действия. | Создан |
| `05_security_and_audit.md` | Видимость пользовательского слоя, защиты маршрутов, работа сессии и граница безопасности/аудита. | Создан |
| `06_user_experience.md` | Общие сценарии интерфейса и профили приложений для Admin, Runtime и Studio. | Создан |
| `07_quality.md` | Проверки, уровни тестирования и известные пробелы. | Создан |
| `08_operations.md` | Node/pnpm, скрипты, переменные окружения и эксплуатационные ограничения. | Создан |
| `90_traceability.md` | Маршрут старых UI/UX материалов, открытые решения и термины-кандидаты. | Создан |

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
