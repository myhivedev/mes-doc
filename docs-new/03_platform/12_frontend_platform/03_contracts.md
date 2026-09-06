---
id: DOC-03-12-03
title: 'Контракты — Фронтенд-платформа'
type: contract
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: frontend_platform
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-09-04 11:54
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Контракты — Фронтенд-платформа

[runtime-contracts-index]: ../../../src/Frontend/packages/runtime-contracts/src/index.ts
[runtime-react-index]: ../../../src/Frontend/packages/runtime-react/src/index.ts
[ui-index]: ../../../src/Frontend/packages/ui/src/index.ts
[ui-types]: ../../../src/Frontend/packages/ui/src/types.ts
[api-client-package]: ../../../src/Frontend/packages/api-client/src/index.ts
[auth-package]: ../../../src/Frontend/packages/auth/src/index.ts
[platform-context-package]: ../../../src/Frontend/packages/platform-context/src/index.ts
[contracts-package]: ../../../src/Frontend/packages/contracts/src/index.ts
[tenant-security-api]: ../../../src/Frontend/packages/tenant-security-api/src/index.ts
[tenant-security-contracts]: ../../../src/Frontend/packages/tenant-security-contracts/src/index.ts
[runtime-app]: ../../../src/Frontend/apps/runtime/src/App.tsx
[backend-runtime-provider]: ../../../src/Frontend/packages/runtime-react/src/provider/runtimeConfigurationProvider.ts
[runtime-request-host]: ../../../src/Frontend/packages/runtime-react/src/host/RuntimeResolvedRequestHost.tsx
[runtime-view-host]: ../../../src/Frontend/packages/runtime-react/src/host/RuntimeResolvedViewHost.tsx
[runtime-contract-source]: ../../../src/Frontend/packages/contracts/src/runtime.ts
[configuration-contract-source]: ../../../src/Frontend/packages/contracts/src/configuration.ts
[configuration-api-client]: ../../../src/Frontend/packages/api-client/src/index.ts
[studio-editor-provider]: ../../../src/Frontend/apps/studio/src/features/configuration-artifact-editor/model/configurationArtifactEditorFeatureProviderDefinition.tsx

## 1. Назначение и границы

Документ описывает публичные контракты фронтенда общей платформы: экспорты
пакетов, типы настроенного пользовательского слоя, хосты, адаптеры, провайдеры,
рендереры и границы вызова API серверной части. Предметная семантика API принадлежит
областям-владельцам.

## 2. Источники истины и владельцы

Источниками формы контрактов фронтенда являются публичные экспорты и типы пакетов
`@dmp/runtime-contracts`, `@dmp/runtime-react`, `@dmp/ui`, `@dmp/auth` и
`@dmp/platform-context`. Источниками серверной семантики остаются контракты
Configuration, Object Runtime, Workflow и Tenant Security.

## 3. Карта контрактов

| Контракт | Вид | Владелец | Потребитель | Статус сведения | Подробное описание |
| --- | --- | --- | --- | --- | --- |
| Публичные экспорты пакетов | Фронтенд | Фронтенд-платформа | Приложения и соседние пакеты | Подтверждено MVP | `### 5.2. Публичная граница @dmp/runtime-react` и ссылки на экспорты |
| Типы UI runtime | Фронтенд | Фронтенд-платформа по форме; Configuration/Object Runtime по семантике | `@dmp/runtime-react`, приложения | Подтверждено MVP | `### 5.1. Типы настроенного пользовательского слоя` |
| Свойства/типы UI-компонентов | Фронтенд | Фронтенд-платформа | Приложения и провайдеры функций | Подтверждено MVP | `### 5.3. Контракты компонентов @dmp/ui` |
| Клиенты API серверной части | HTTP-клиент | Соответствующая область серверной части | Приложения и провайдеры | Подтверждено, но ограничено | `## 10. Границы с другими владельцами` |

## 4. HTTP-контракты

Фронтенд-платформа не владеет семантикой API серверной части, но описывает
наблюдаемые вызовы, которые необходимы общим приложениям и runtime-слою.
Подробные модели запросов и ответов, а также правила операций остаются в документах
владельцев соответствующих API.

| Вызов | Потребитель | Назначение на стороне фронтенда | Владелец семантики | Статус |
| --- | --- | --- | --- | --- |
| `/api/runtime/entry-point` | `apps/runtime` | Получить начальные коды модуля, типа объекта, представления и меню для построения маршрута. | Object Runtime | Подтверждено кодом |
| `/api/runtime/views/resolve` | `RuntimeConfigurationProvider` | Получить `RuntimeViewResolveResponse` для построения настроенного пользовательского слоя. | Object Runtime / Configuration | Подтверждено кодом |
| API контекста и редактора Configuration | `apps/studio` | Загрузить контекст и проекцию редактора, отправить проверку, сохранение или отмену изменений. | Configuration | Контракт потребляется; полный каталог операций описан у владельца |
| API предметных функций Admin/Studio | Провайдеры функций приложений | Загрузить и изменить данные предметного сценария. | Владелец соответствующей области | Потребление подтверждено; семантика не дублируется |

Обязательные для клиента ошибки и ограничения фиксируются вместе с конкретным
API владельцем. Фронтенд не добавляет новые коды ошибок и не подменяет серверную
проверку права.

## 5. Общие типы и DTO

### 5.1. Типы настроенного пользовательского слоя

`@dmp/runtime-contracts` экспортирует типы для:

- `configuredObjectNavigationPolicy`;
- `uiConfigurationContracts`;
- `uiEnumContracts`;
- `uiGridContracts`;
- `uiLookupContracts`;
- `uiObjectRulesContracts`;
- `uiOperationContracts`;
- `uiPersonalization`;
- `uiViewContracts`;
- `uiWorkspaceContracts`;
- `UiContext`.

Эти типы описывают контракт пользовательского слоя на стороне фронтенда. Они не
заменяют серверные контракты Configuration/Object Runtime и не задают схему
артефактов. ([экспорты][runtime-contracts-index])

### 5.2. Публичная граница `@dmp/runtime-react`

`@dmp/runtime-react` экспортирует хосты, адаптеры, провайдеры, рендереры, хуки и
реестр. Для документации фронтенд-платформы важны следующие группы:

| Группа | Контракт |
| --- | --- |
| Хост | `ConfiguredViewHost`, `RuntimeResolvedViewHost`, `RuntimeResolvedRequestHost`, `RuntimeResolvedRouteHost` |
| Провайдеры данных и конфигурации | `configuredFeatureProvider`, `configuredObjectDataProvider`, `runtimeConfigurationProvider` |
| Адаптеры runtime | `runtimeViewAdapter`, `runtimeListAdapter`, `runtimeMenuAdapter`, `runtimeNavigationAdapter`, `runtimeActionAdapter`, `runtimeConditionAdapter` |
| Рендереры | `ConfiguredListView`, `ConfiguredObjectCrudModeView`, `ConfiguredObjectFormModeView`, `ConfiguredObjectListModeView`, `ConfiguredWorkspaceModeView`, `RuntimeObjectWorkflowPanel` |
| Состояние и хуки | Состояние списка, runtime-страницы объекта, выполнение операции изменения, защита от потери несохранённых изменений, состояние поля lookup |

Текущая публичная граница подтверждена списком экспортов. Семантика отдельных
серверных операций описывается у владельцев runtime и серверной части. ([экспорты][runtime-react-index])

### 5.3. Контракты компонентов `@dmp/ui`

`@dmp/ui` экспортирует переиспользуемые компоненты и типы. Для платформенной
совместимости важны `NavigationGroup`, `NavigationItem`, `DataTableColumn`,
`GridLayoutMode`, `GridSelectionMode`, `GridHeightMode`, `GridFilterMode`,
`GridFilterPlacement`, `GridStickyMode`, `PaginationMode`,
`ActionBarAction`, `ActionMenuAction`, `TopBarUserMenuAction` и типы разметки
форм. ([экспорты][ui-index]; [типы][ui-types])

Набор UI-компонентов не принимает на себя ответственность за решения о правах,
серверные фильтры или команды Workflow. Свойства компонентов должны получать
уже вычисленную модель из контрактов runtime и серверной части либо от провайдера
функции приложения.

### 5.4. Контракт связи Configuration и runtime UI

Фронтенд использует разные контракты для авторинга и выполнения. Это намеренное
разделение, а не две конкурирующие формы одного хранилища фронтенда.

| Сценарий | Основной контракт | Ключевые данные | Операции фронтенда | Каноническое хранилище |
| --- | --- | --- | --- | --- |
| Контекст Studio | `ConfigurationContextBootstrapResponse` | `scope`, `version`, `mode`, `readonly`, метаданные `base`/`previous`/`upgrade`, действия, инициируемые сервером | загрузка контекста, выбор `scope`/`version`/`mode`, вызов действий контекста | Configuration |
| Дерево и редактор артефакта | `ConfigurationArtifactEditorResponse` | `rootEntryId`, `selectedEntryId`, `artifactCode`, узлы, секции, поля, локализация, состояние переопределения, `revisionToken` | отображение проекции редактора, локальные изменения свойств и структуры | Configuration |
| Проверка и сохранение сессии редактора | `ValidateConfigurationArtifactEditorRequest` и `SaveAllConfigurationArtifactEditorRequest` | `baseRevisionToken`, `propertyChanges`, `structureChanges`; ответ возвращает проблемы, новый токен и соответствие временных идентификаторов | отправка накопленного буфера и обновление сессии клиента | Configuration |
| Runtime-представление экрана | `RuntimeViewResolveResponse` | метаданные `view`/`object`, столбцы и фильтры списка, элементы и разметка, требования к данным, действия, источники вариантов, права, меню и Workflow | получить модель, адаптировать к модели рендерера, загрузить данные и вызвать API | Материализатор runtime и владельцы контрактов серверной части |
| Данные объекта runtime | `RuntimeObjectListResponse`, `RuntimeObjectDetailsResponse` и ответы операций изменения, действий и Workflow | строки, значения, постраничная выдача, проверка, конкуренция, состояние и команды Workflow | хранить состояние текущего списка/формы, показать ответ, повторно запросить данные | Object Runtime, Workflow, предметный модуль |

Runtime-путь в текущем коде подтвержден как:

```text
RuntimeResolvedRouteHost
  -> RuntimeConfigurationProvider.resolveView
  -> POST /api/runtime/views/resolve
  -> RuntimeResolvedRequestHost
  -> RuntimeResolvedViewHost
  -> buildRuntimeConfiguredViewDefinition
  -> runtime-react registry/controllers/renderers
```

Провайдер серверной части также использует API runtime lookup для вариантов
(`options`). Локальный
провайдер допускается для переходных экранов Admin/Studio, но его контрактом
остаётся тот же `resolveView(): Promise<RuntimeViewResolveResponse>`. Локальная
`ConfiguredViewBaseline` может быть источником такой переходной проекции, но не
должна объявляться канонической моделью артефакта. Целевое требование для
локальных runtime-конфигураций Admin и Studio мягче, чем обязательная серверная
загрузка: они должны по возможности повторять каноническую структуру
артефактов и модели runtime (`payload`) и использовать те же технические имена. Для
сложных форм и специализированных экранов Studio допускаются дополнительные
поля и модели только фронтенда, пока общий runtime не покрывает нужный сценарий
взаимодействия; их границу и назначение нужно держать явными.

Редактор Studio, напротив, получает `ConfigurationArtifactEditorResponse` через
Configuration API и держит до сохранения исходное состояние и исправления на
стороне клиента. В текущем коде это состояние оркестрации в React; признаков
отдельного постоянного хранилища фронтенда для канонических артефактов не найдено.

Подробные поля моделей серверной части и их семантика принадлежат
`Configuration`, `Object Runtime` и `Workflow`. Здесь фиксируется только
граница фронтенда: какой контракт принимает слой отображения, какие изменения
можно временно держать в памяти и какой API вызывается для сохранения или
исполнения.

## 6. C#-контракты и точки расширения

Публичных C#-контрактов, принадлежащих фронтенд-пакетам, в этой области нет.
Граница расширения фронтенда выражена TypeScript-контрактами и реестрами:
`ConfiguredFeatureProvider`, `RuntimeConfigurationProvider`, хостами, адаптерами,
провайдерами и рендерерами. Серверные C#-контракты Configuration, Object Runtime,
Workflow и Tenant Security являются внешними для области и описываются их
владельцами.

| Точка расширения | Контракт фронтенда | Владелец семантики | Ограничение |
| --- | --- | --- | --- |
| Настроенная функция приложения | `ConfiguredFeatureProvider` и реестр провайдера | Область-владелец функции | Функция не должна менять общий runtime-контракт без согласования. |
| Разрешение runtime-представления | `RuntimeConfigurationProvider` и `RuntimeViewResolveResponse` | Object Runtime / Configuration | Локальная проекция Admin/Studio не становится каноническим хранилищем. |
| Отображение настроенной модели | Хосты, адаптеры и рендереры `@dmp/runtime-react` | Фронтенд-платформа по форме; владельцы API по смыслу | Рендерер не выводит схему и право из локальных предположений. |

## 7. Контракты событий

Отдельных событий фронтенда или контрактов брокера событий, принадлежащих этой
области, текущий код не подтверждает. События и записи, возникающие при
сохранении, изменении объекта, выполнении Workflow или аудите, принадлежат
соответствующим серверным областям.

| Вид события | Состояние | Владелец |
| --- | --- | --- |
| Событие интерфейса внутри React | Внутренний механизм состояния, не межмодульный контракт | Фронтенд-платформа |
| Событие сохранения или изменения объекта | Фронтенд вызывает API и отображает результат; отдельный контракт события фронтенда не подтверждён | Object Runtime / Configuration |
| Событие Workflow или аудита | Не принадлежит фронтенд-платформе | Workflow / Audit History |

## 8. Ошибки и отказоустойчивость

В текущем коде фронтенда подтверждены обработка состояния сессии и отображение
ошибок API на уровне приложений. Полный каталог кодов ошибок и политика
повторных запросов принадлежат контрактам серверной части и здесь не дублируются.

| Код или тип ошибки | Условие | HTTP или результат транспорта | Поле/path | Повторить запрос | Ответственный |
| --- | --- | --- | --- | --- | --- |
| Неавторизованный запрос | Сессия отсутствует или отклонена | `401`/`403` | — | После обновления сессии, если это разрешено владельцем API | Tenant Security / API owner |
| Ошибка разрешения runtime | Не удалось получить точку входа или разрешённое представление | Ошибка API; точный код задаёт серверная часть | — | Решается политикой runtime; текущий резервный переход ведёт на стартовую страницу | Object Runtime / Configuration |

## 9. Совместимость и изменение контрактов

Сохраняются имена JSON-полей, маршруты, коды объектов/действий/ошибок/прав и
семантика опубликованной конфигурации, если отдельное архитектурное решение не
устанавливает иное. Полный манифест совместимости и политика версионирования ещё не
подтверждены и записаны как `FE-DEC-01 — источник совместимости контрактов фронтенда`
в `90_traceability.md`.

| Изменение | Совместимо назад | Потребители | Миграция | Версия или решение |
| --- | --- | --- | --- | --- |
| Изменение публичного экспорта или типа фронтенда | Не утверждается без проверки потребителей. | Приложения и соседние пакеты | Обновить потребителей и контрактные тесты. | Требует манифеста `FE-DEC-01`. |
| Изменение имени/типа JSON, маршрута или кода | Требование совместимости сохраняется, если нет отдельного решения. | Провайдеры фронтенда и потребители API серверной части | Согласовать версионирование и переход с владельцем API. | Архитектурный результат; политика не завершена. |

## 10. Границы с другими владельцами

| Пакет | Ответственность | Владелец серверной семантики |
| --- | --- | --- |
| `@dmp/api-client` | Общий HTTP-клиент, ошибки и интеграция с контекстом платформы. | Владельцы API |
| `@dmp/auth` | Страница входа, вспомогательные средства хранения и запуска сессии. | Tenant Security по семантике сессии |
| `@dmp/platform-context` | Провайдер/значение контекста на стороне клиента. | Владельцы контекста платформы |
| `@dmp/contracts` | Общие типы TypeScript фронтенда для контрактов серверной части. | Соответствующие платформенные области серверной части |
| `@dmp/tenant-security-api` | Обёртки HTTP-клиента Tenant Security. | Tenant Security |
| Вызовы Runtime API из приложения Runtime | Точка входа, разрешение представления, данные/изменения объекта через провайдеры runtime. | Object Runtime / Configuration / Workflow по сегменту контракта |

Приложение Runtime вызывает `/api/runtime/entry-point` и открывает настроенный
маршрут только если ответ содержит коды модуля, типа объекта и представления.
([Runtime app][runtime-app])

## 11. Источники и тесты

Публичные экспорты и типы проверяются по исходным `index.ts`, а поведение
компонентов и навигации — по unit/component и e2e-тестам, перечисленным в
`07_quality.md`. Требования architectural outcome используются как ограничение
совместимости, а не как источник уже реализованного поведения.

## 12. Контракты файлов

Frontend получает типизированное значение `File`, `ContentResourceDescriptor` и
метаданные политики из Runtime. Загрузка использует типизированный Content API;
URL не строится из физического пути. Presentation выбирается по контексту:

| Контракт Configuration | Presentation-компонент | Семантика |
| --- | --- | --- |
| Свойство объекта `DataType=File` | `FileField` | Скалярное поле одного файла для карточки и списка, включая inline-редактирование. |
| Параметр action `DataType=File`, `Multiple=false/true`, `ComponentCode=FilePicker` | `FileUploadParameter` | Диалог выбора одного или нескольких файлов с загрузкой, заменой, очисткой и ошибками по каждому файлу. |

Оба компонента передают в runtime ссылки/descriptor, а не путь к файлу и не
бинарное содержимое. `FileField` не является множественным контролом; количество
файлов в `FileUploadParameter` определяется свойством `Multiple` параметра action.
Общие контракты
не содержат знаний о Document Management или demo.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-04 11:54 +03:00 | Олег Юрьев (@axelprosoft) | Контракты файлов | docs: уточнить файловые UI-паттерны | [18afd17c](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/18afd17c37fed8a2fa749321e286c6d6cd215bd0) |
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.2 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Контракты файлов | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
