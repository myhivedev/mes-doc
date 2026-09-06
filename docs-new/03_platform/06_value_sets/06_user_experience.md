---
id: DOC-03-06-06
title: 'Пользовательский слой — Value Sets'
type: design
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: value_sets
holder: '@axelprosoft'
created_at: 2026-08-26 18:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Пользовательский слой — Value Sets

## 1. Назначение документа

Документ описывает конкретную интеграцию редактора данных Value Sets в приложении
Studio: маршрут, навигацию, feature provider, настроенные представления и локализацию.
Общая оболочка приложений, роутер и общие UI-компоненты принадлежат [фронтенд-платформе](../12_frontend_platform/06_user_experience.md).

## 2. Frontend-контракты

`StudioValueSetDataEditor` — зарегистрированный feature provider Studio. Он связывает
серверные API Value Sets с настроенными представлениями и передаёт в них права
`ValueSets.Data.Edit` и `ValueSets.Data.ManageScope`. Сервер остаётся источником истины
для проверки области, политики, родителя и допустимости операции.

| Контракт или компонент | Техническое имя | Потребитель | Источник | Статус реализации |
| --- | --- | --- | --- | --- |
| Поставщик функциональности | `StudioValueSetDataEditor` | Исполняющая среда настроенных объектов Studio | [определение поставщика][studio-provider] | Подтверждено в MVP |
| Набор представлений | `Studio.ValueSetData.*` | Исполняющая среда настроенных объектов Studio | [представления Value Sets][studio-runtime] | Подтверждено в MVP |
| Слой запросов данных | Поставщик данных исполнения | Поставщик функциональности Studio | [поставщик данных][studio-data-provider] | Подтверждено в MVP |
| Общая отрисовка CRUD | `RuntimeBridgeObjectCrudView` | Поставщик функциональности Studio | [определение поставщика][studio-provider] | Подтверждено в MVP |

Схема `ValueSet` редактируется в Configuration. Редактор Studio изменяет только
фактические `ValueSetItem` через HTTP API Value Sets.

## 3. Разделы интерфейса и сценарии

| Раздел или сценарий | Представление или пакет | API или источник данных | Право | Статус реализации |
| --- | --- | --- | --- | --- |
| Список наборов данных | `Studio.ValueSetData.Definitions`, `ObjectList` | `GET /api/platform/value-sets/data/definitions` | `ValueSets.Data.View` | Подтверждено в Studio |
| Карточка набора данных | `Studio.ValueSetData.Card`, `ObjectForm` | Данные выбранного набора | `ValueSets.Data.View` | Подтверждено в Studio |
| Список элементов | `Studio.ValueSetData.Items`, `ObjectList` | `GET /api/platform/value-sets/data/{valueSetCode}/items` | `ValueSets.Data.View` | Подтверждено в Studio |
| Карточка элемента | `Studio.ValueSetData.Item.Card`, `ObjectForm` | `POST` и `PUT` для items | `ValueSets.Data.Edit` | Подтверждено в Studio |
| Иерархический список | `TreeGrid` в `Studio.ValueSetData.Items` | `.../parent-candidates` и список items | `ValueSets.Data.View` | Включается для `Structure = Hierarchical` |
| Выбор `Tenant`, `Site` и родителя | `Studio.Platform.Tenants.Lookup`, `Studio.Platform.Sites.Lookup`, `Studio.ValueSetData.Items.Parent.Lookup` | Lookup API Value Sets/host | `ValueSets.Data.ManageScope` для расширенной области | Подтверждено в Studio |

Основной путь: открыть `Списки значений`, выбрать набор, открыть его элементы,
просмотреть эффективные или физические строки, создать/изменить строку и получить
результат серверной проверки.

## 4. Навигация

| Пункт навигации | Родитель | Цель | Право | Порядок | Источник |
| --- | --- | --- | --- | --- | --- |
| `Studio.Nav.ValueSetData` | `Studio.Nav.Group.Settings` | `/value-set-data`, `Studio.ValueSetData.Definitions` | Проверка доступа выполняется при запросе данных | `35` | `studio.navigation.runtime.ts`, `studioConfiguredRouteRegistry.ts` |

Пункт входит в меню `Studio.MainMenu`, группу `Настройки` и использует код значка
`studioValueSetData`. Общие правила меню и маршрутизации принадлежат
[фронтенд-платформе](../12_frontend_platform/06_user_experience.md).

## 5. Локализация

Локализованные подписи редактора определены в
[`studio.value-set-data.runtime.texts.ts`][studio-texts]. Значение `Site` в техническом контракте
документации сохраняется; русская подпись в текущем frontend-источнике требует
отдельной проверки терминологии и не меняет техническое имя поля.

| Ресурс | Ключ | Язык | Резервное значение | Владелец | Покрытие |
| --- | --- | --- | --- | --- | --- |
| Пункт меню | `Studio.Nav.ValueSetData` | `ru-RU` / `en-US` | `Value Set Data` | Навигация Studio | `Списки значений` / `Value Set Data` подтверждено |
| Страница данных | `viewValueSetData` | `ru-RU` / `en-US` | `Value Set Data` | Функциональность Value Sets | `Списки значений` / `Value Set Data` подтверждено |
| Элементы списка | `viewValueSetItems`, `viewValueSetItem` | `ru-RU` / `en-US` | Техническое имя | Функциональность Value Sets | `Значения списка` / `Значение списка` подтверждено |
| Поле области | `titleTenant`, `titleSite`, `titleScope` | `ru-RU` / `en-US` | `Tenant`, `Site`, `Scope` | Функциональность Value Sets | Переводы определены в frontend-источнике |
| Поля и действия редактора | `titleCode`, `titleName`, `titleActive`, `titleOrder`, `action*` | `ru-RU` / `en-US` | Техническое имя | Функциональность Value Sets | Переводы определены в frontend-источнике |

Отображаемые значения элементов, Tenant, Site и родителя дополнительно приходят от
серверных полей `DisplayValues` и `LookupDisplay`; UI не строит подписи из внутренних GUID.

## 6. Ограничения

Для `Flat` выбор родителя не показывается. Для `Hierarchical` используется `TreeGrid`,
кандидаты в родители и отображаемый путь; проверка циклов остаётся серверной.

| Состояние | Реакция интерфейса | Основание |
| --- | --- | --- |
| `Fixed` | Только чтение и причина блокировки | `CanEdit` и `DisabledReason*` |
| Нет `ValueSets.Data.Edit` | Действия изменения недоступны | Права сессии и ответ API |
| Унаследованная строка | Показать создание override, а не обычное сохранение | `CanCreateOverride` |
| Нет `ValueSets.Data.ManageScope` | Поля выбора `Tenant`/`Site` скрыты или недоступны | Feature provider и серверная проверка |
| `Site` без `Tenant` | Показать ошибку проверки | Контракт области данных |

Текущая реализация подтверждает Studio, но не заявляет отдельную интеграцию этого
редактора в Admin или других приложениях. Общая оболочка, маршрутизация и базовые UI
компоненты не становятся собственностью Value Sets.

[studio-provider]: ../../../src/Frontend/apps/studio/src/features/value-set-data-editor/model/valueSetDataEditorFeatureProviderDefinition.tsx
[studio-runtime]: ../../../src/Frontend/apps/studio/src/features/value-set-data-editor/runtime/studio.value-set-data.runtime.ts
[studio-data-provider]: ../../../src/Frontend/apps/studio/src/features/value-set-data-editor/model/valueSetDataRuntimeDataProvider.ts
[studio-texts]: ../../../src/Frontend/apps/studio/src/features/value-set-data-editor/runtime/studio.value-set-data.runtime.texts.ts

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
