---
id: DOC-03-12-02
title: 'Архитектура — Фронтенд-платформа'
type: architecture
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

# Архитектура — Фронтенд-платформа

[workspace]: ../../../src/Frontend/pnpm-workspace.yaml
[frontend-package]: ../../../src/Frontend/package.json
[runtime-react-index]: ../../../src/Frontend/packages/runtime-react/src/index.ts
[runtime-contracts-index]: ../../../src/Frontend/packages/runtime-contracts/src/index.ts
[ui-index]: ../../../src/Frontend/packages/ui/src/index.ts
[ui-types]: ../../../src/Frontend/packages/ui/src/types.ts
[runtime-app]: ../../../src/Frontend/apps/runtime/src/App.tsx
[runtime-main]: ../../../src/Frontend/apps/runtime/src/main.tsx
[admin-app]: ../../../src/Frontend/apps/admin/src/App.tsx
[studio-app]: ../../../src/Frontend/apps/studio/src/App.tsx
[admin-navigation]: ../../../src/Frontend/apps/admin/src/ui-runtime/navigation/admin.navigation.runtime.ts
[studio-navigation]: ../../../src/Frontend/apps/studio/src/ui-runtime/navigation/studio.navigation.runtime.ts
[runtime-route-host]: ../../../src/Frontend/packages/runtime-react/src/host/RuntimeResolvedRouteHost.tsx
[runtime-registry]: ../../../src/Frontend/packages/runtime-react/src/runtimeRegistry.tsx

## 1. Назначение документа

Документ описывает компоненты, архитектурную модель и точки расширения общей
фронтенд-платформы. Подробные граничные контракты, сценарии и эксплуатационные
действия вынесены в `03_contracts.md`, `04_runtime.md` и `08_operations.md`.

## 2. Граница и компоненты

| Компонент | Ответственность | Зависимости | Подтверждение |
| --- | --- | --- | --- |
| Рабочая область фронтенда | Управляет приложениями, пакетами и общими скриптами. | Node, pnpm | [`pnpm-workspace.yaml`][workspace]; [`package.json`][frontend-package] |
| Приложения | Три приложения с отдельными точками входа и собственным профилем оболочки. | Общие пакеты | [`Admin`][admin-app]; [`Runtime`][runtime-app]; [`Studio`][studio-app] |
| Пакеты платформы | Общие пакеты для API, аутентификации, контекста, контрактов, отображения runtime, UI и вспомогательных средств. | Приложения и API серверной части | `src/Frontend/packages/*` |
| Слой UI runtime | Преобразует настроенную модель в компоненты React и состояние клиента. | `@dmp/runtime-contracts`, `@dmp/ui` | [`runtime-react`][runtime-react-index] |
| Набор UI-компонентов | Предоставляет оболочку, навигацию, элементы управления списками, формами и действиями, а также компоненты разметки. | React и пакеты приложений | [`ui`][ui-index] |

## 3. Архитектурная модель и инварианты

### 3.1. Три приложения как профили

| Понятие или архитектурный тип | Техническое имя | Владелец | Связи | Инвариант | Подтверждение |
| --- | --- | --- | --- | --- | --- |
| Профиль администрирования | `@dmp/app-admin` | Фронтенд-платформа | Tenant Security, общие пакеты | Использует общий запуск и оболочку; локальная и удалённая навигация остаются различимыми путями. | [`Admin`][admin-app] |
| Профиль опубликованного runtime | `@dmp/app-runtime` | Фронтенд-платформа | Configuration, Object Runtime, Workflow | Экран строится по разрешённой модели runtime, а не по локальной схеме артефакта. | [`Runtime`][runtime-app] |
| Профиль авторинга | `@dmp/app-studio` | Фронтенд-платформа | Configuration, Value Sets, Settings | Провайдеры функций Studio подключаются к общей оболочке и механизмам runtime. | [`Studio`][studio-app] |

Общее между приложениями: React, Vite, `BrowserRouter`, `TenantLoginPage`,
`useAuthSessionBootstrap`, `PlatformContextProvider`, `PlatformHttpClient`,
общая локализация и `@dmp/ui/styles.css`. Отличия приложений описываются как профиль
поверх общего механизма, а не как отдельные платформенные области.

### 3.2. Конвейер runtime-рендеринга

`@dmp/runtime-react` экспортирует несколько групп архитектурных элементов:

| Группа | Примеры экспортов | Ответственность |
| --- | --- | --- |
| Хосты | `ConfiguredViewHost`, `RuntimeResolvedViewHost`, `RuntimeResolvedRouteHost` | Подключение разрешённой/настроенной модели к дереву React. |
| Адаптеры | `runtimeViewAdapter`, `runtimeListAdapter`, `runtimeNavigationAdapter`, `runtimeLayoutAdapter` | Приведение модели runtime и серверной части к модели представления фронтенда. |
| Провайдеры | `configuredFeatureProvider`, `configuredObjectDataProvider`, `runtimeConfigurationProvider` | Источник настроенных представлений, данных объекта и конфигурации runtime. |
| Рендереры | `ConfiguredListView`, `ConfiguredObjectFormModeView`, `ConfiguredObjectListModeView`, `ConfiguredWorkspaceModeView`, `RuntimeObjectWorkflowPanel` | Отображение фрагментов списка, формы, карточки, рабочего места, действия и Workflow. |
| Hooks и модель | `useConfiguredListDataLoader`, `useConfiguredObjectPageRuntime`, `useConfiguredUnsavedChangesGuard` | Состояние клиента, загрузка данных, навигация, состояние черновика/изменений, действия и работа lookup. |

Приложение Runtime использует `RuntimeResolvedRouteHost` как основной связующий
компонент между параметрами маршрута и настроенным пользовательским слоем объекта.
([приложение Runtime][runtime-app]; [хост маршрута][runtime-route-host])

### 3.3. Связь артефакта, editor-модели и runtime-проекции

В платформе нельзя считать все объекты, которые используются интерфейсом,
одной моделью. Серверная часть Configuration хранит канонические
`ArtifactDocument`/`ArtifactNode`/`ArtifactValue` и строит
`EffectiveArtifactDocument`. Фронтенд получает не внутреннее хранилище, а
специализированные проекции для двух разных сценариев:

| Слой | Модель или контракт | Источник | Потребитель | Что находится на фронтенде | Владелец смысла |
| --- | --- | --- | --- | --- | --- |
| Авторинг конфигурации | `ConfigurationContextBootstrapResponse`, `ConfigurationExplorerResponse`, `ConfigurationArtifactEditorResponse` | Configuration API | `apps/studio` | Состояние React, исходное состояние узлов, `patchBuffer`, ожидающие структурные изменения, признак изменений | Configuration |
| Изменение авторинга | `ValidateConfigurationArtifactEditorRequest`, `SaveAllConfigurationArtifactEditorRequest`, `DiscardConfigurationArtifactEditorRequest` | API-клиент фронтенда | Редактор Studio | Накопленные изменения до отправки; `revisionToken` используется для проверки конкуренции | Configuration |
| Эффективная конфигурация | `EffectiveConfigurationResponse` с `root`, `nodes`, `properties` | Configuration API | Explorer, диагностика, материализатор серверной части | Временная клиентская проекция для отображения и диагностики | Configuration |
| Разрешённое runtime-представление | `RuntimeViewResolveResponse` с `view`, `object`, `list`, `elements`, `layout`, `dataRequirements`, `actions`, `lookups`, `valueSets`, `permissions`, `menu` и связанными полями | Runtime API или локальный провайдер runtime | `apps/runtime`, части Admin/Studio, использующие runtime-модель | Модель в памяти и производное представление; каноническая конфигурация не сохраняется | Configuration, Object Runtime, Workflow по своим частям |
| Настроенное локальное представление | `ConfiguredViewBaseline`, локальные определения представления и `buildConfiguredRuntimeViewResolveResponse` | Реестр/конфигурация фронтенда, переходный слой | Admin и отдельные экраны Studio | Статическая или временная локальная проекция; не второй источник истины серверной части | Фронтенд-платформа отвечает за отображение, владельцы API — за семантику |
| Данные объектов | `RuntimeObjectListResponse`, `RuntimeObjectDetailsResponse`, ответы операций изменения, действий и Workflow | Runtime API | Контроллер runtime и рендереры | Состояние списка, формы, выбора и отображаемых изменений | Object Runtime, Workflow, предметный модуль |

Поток runtime-представления имеет следующую форму:

```text
Хранилище Configuration
  -> опубликованная/эффективная конфигурация
  -> материализатор Runtime API
  -> RuntimeViewResolveResponse
  -> RuntimeResolvedRouteHost
  -> RuntimeResolvedViewHost
   -> адаптеры, контроллеры и рендереры `runtime-react`
```

Поток авторинга Studio отделён от него:

```text
Configuration API
  -> проекция редактора/контекста
  -> исходное состояние Studio + локальные исправления
  -> запрос validate/save/discard
  -> сохранение Configuration и жизненный цикл версии
```

Таким образом, `View` как конфигурационный артефакт, ответ редактора Studio и
`RuntimeViewResolveResponse` не являются взаимозаменяемыми моделями. Runtime
приложение получает разрешённую сервером проекцию экрана, а Studio редактирует
отдельную editor-проекцию конфигурации. Для runtime-like экранов `Admin` и
`Studio` локальный источник допустим: его runtime-конфигурацию следует по
возможности выражать теми же кодами, связями и структурой, что и канонические
артефакты и `RuntimeViewResolveResponse`. Специализированный экран может иметь
дополнительную модель фронтенда для сложной формы или рабочего пространства,
если общий runtime пока не поддерживает нужное поведение. Такое расширение не
должно менять смысл серверного артефакта или становиться вторым источником
предметной истины.

## 4. Persistence-модель и хранение

| Данные | Техническое представление | Владелец | Хранение | Ключ или ограничение | Подтверждение |
| --- | --- | --- | --- | --- | --- |
| Состояние пользовательского интерфейса | Состояние React и локальное хранилище браузера | Фронтенд-платформа | Память браузера; локаль, тема, настройки боковой панели и состояние сессии в `local storage` | Не является каноническим предметным состоянием. Политика хранения и маскирования для рабочей среды не завершена. | `@dmp/shared`, `@dmp/access-control`, `@dmp/ui`, приложения |
| Сессия редактора | `baselineByEntryId`, `patchBuffer`, `pendingStructureActions`, `revisionToken`, признак изменений | Studio для управления состоянием клиента; Configuration для сохранения | Память вкладки до `SaveAll`/`Discard`; серверная проверка идёт через API | Не является самостоятельным хранилищем артефактов и не заменяет хранилище версий/черновиков Configuration. | Редактор артефакта Studio, `@dmp/contracts`, `@dmp/api-client` |
| Канонические артефакты, версии и эффективное состояние | `ArtifactDocument`, `ArtifactNode`, `ArtifactValue`, `EffectiveArtifactDocument` и серверные проекции ответов | Configuration | Хранилище серверной части и материализация на сервере | Фронтенд не читает внутренние таблицы и не сохраняет каноническую схему самостоятельно. | Контракты и API Configuration |
| Предметные данные и настройки | Модель API и состояние runtime | Configuration / Object Runtime / Tenant Security / Value Sets / Settings | Хранилище серверной части | Фронтенд отображает данные и отправляет команды через соответствующий API. | `03_contracts.md`, контракты владельцев |

## 5. Зависимости и точки расширения

Фронтенд-платформа зависит от серверной части через HTTP/API-клиенты и контракты, но не
дублирует логику владельцев серверной части. Runtime API, Configuration API и Tenant Security
API остаются у своих владельцев. Фронтенд-платформа документирует потребление
контрактов, ограничения совместимости и поведение отображения на клиенте.

### 5.1. Общий набор UI-компонентов

`@dmp/ui` экспортирует компоненты уровня оболочки (`AppShell`, `TopBar`, `Sidebar`,
`PageLayout`, `PageHeader`) и элементы управления для списков, фильтров, форм, диалогов,
действий, вкладок, заголовков представления объекта и проверки. Типы `DataTableColumn`,
`GridLayoutMode`, `GridSelectionMode`, `GridFilterPlacement`, `GridHeightMode`,
`PaginationMode`, `NavigationGroup` и `NavigationItem` задают контракт фронтенда
для переиспользуемых UI-компонентов. ([экспорты][ui-index]; [типы][ui-types])

Компонентный слой не является владельцем серверной семантики: фильтры,
сортировка, доступность действий и состояние Workflow должны приходить из
соответствующих контрактов runtime и серверной части.

### 5.2. Архитектура навигации

Фронтенд-платформа различает:

- модель навигации оболочки: `NavigationGroup` / `NavigationItem` в `@dmp/ui`;
- проекцию меню runtime: адаптеры из `@dmp/runtime-react`;
- материализацию навигации конкретного приложения: реестры Admin и Studio;
- серверную/эффективную навигацию: опубликованный `Menu` и модель runtime (`payload`), а также путь
  удалённой навигации Admin.

Admin и Studio уже имеют материализаторы навигации runtime. Приложение Runtime
строит навигацию из разрешённого представления/меню и сохраняет состояние боковой панели
по идентификатору пользователя.
([Admin navigation][admin-navigation]; [Studio navigation][studio-navigation];
[Runtime app][runtime-app])

### 5.3. Статус целевых шаблонов

Старые UI-требования предлагают расширять общий runtime через
`SplitExplorerTemplate`, `TreeInspectorTemplate`, `WorkspaceTemplate` и другие
шаблоны. Текущий код подтверждает регистрацию `WorkspaceTemplate`,
`SplitExplorerTemplate` и `TreeInspectorTemplate` в
`configuredTemplateRegistry.tsx`, но соответствующие рендереры пока передают
`children` без самостоятельной реализации семантики разметки режимов `split`, `tree` и `workspace`.
`WizardTemplate` в текущем реестре не найден.

Поэтому эти имена документируются как точки расширения общего runtime, а не как
готовая гарантия пользовательского поведения. Требования к конкретному
Configuration Explorer, редакторам, публикации/импорту/rebase и графическому рабочему месту
остаются у Configuration; фронтенд-платформа владеет только переиспользуемой
разметкой, хостом и контрактами компонентов. Полная трассировка приведена в
`90_traceability.md`.

## 6. Технические ограничения

Фронтенд-платформа не является владельцем серверной семантики, решений о правах,
схемы артефактов, команд Workflow и предметного состояния. Семантика
`ExternalUrl`, `Action`, полной совместимости контрактов и гарантий разметки,
которая не подтверждена кодом, зафиксирована в `90_traceability.md`.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
