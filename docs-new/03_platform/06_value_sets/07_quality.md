---
id: DOC-03-06-07
title: 'Качество — Value Sets'
type: assurance
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

# Качество — Value Sets

## 1. Назначение документа

Документ связывает инварианты Value Sets с тестовым свидетельством и известными
пробелами. Он не подменяет сами тесты.

## 2. Надёжность

| Отказ | Гарантия | Предел | Восстановление | Проверка |
| --- | --- | --- | --- | --- |
| Неверная область или родитель | Запрос отклоняется кодом `VALUE_SET_*` | Только проверки текущего API | Исправить запрос и повторить | `ValueSetsApiIntegrationTests` |
| Повторный запуск миграции | История миграций предотвращает повторное применение | Путь миграций SQL Server | Повторить запуск после диагностики | `ValueSetsDatabaseInitializer` |
| Отсутствуют начальные данные | Пакет может быть зарегистрирован, но набор данных может быть пустым | Зависит от адаптера host-приложения | Проверить baseline/seed и повторить | Интеграционные тесты начальных данных |

## 3. Наблюдаемость

| Сигнал | Источник | Корреляция | Потребитель | Диагностический пробел |
| --- | --- | --- | --- | --- |
| Код `VALUE_SET_*` в ответе | API Value Sets | Контекст запроса host-приложения | API-клиент и оператор | Отдельные метрики Value Sets не подтверждены |
| Пустой результат чтения | Query service | `ValueSetCode` и текущая область | Runtime/API-потребитель | Отдельное событие инвалидации кэша не подтверждено |

## 4. Производительность

| Сценарий нагрузки | Предел или SLO | Метод | Результат |
| --- | --- | --- | --- |
| Чтение страницы элементов | SLO не задан | Интеграционные API-тесты проверяют корректность, не норматив времени | Не измерялось |
| Построение effective-набора | SLO не задан | Интеграционные тесты и проверка сервиса | Не измерялось |

## 5. Проверки и результаты

Проверяемый путь для API: запрос, авторизация, проверка области, сервис чтения или
изменения, проверка инвариантов и ответ с результатом или техническим кодом.

| Сценарий проверки | Уровень | Источник теста | Результат запуска | Пробел |
| --- | --- | --- | --- | --- |
| API Runtime возвращает элементы | Integration | [`ValueSetsApiIntegrationTests`][api-tests] | Подтверждено существующим тестом | Полное покрытие host-вариантов не заявляется |
| Baseline/bootstrap создаёт данные | Integration | [`GeneralMasterDataValueSetsBootstrapIntegrationTests`][bootstrap-tests] | Подтверждено существующим тестом | Конфликт seed с пользовательскими данными не определён |
| Модель хранения и ограничения ключей | Интеграция и хранение | [`ValueSetsDbContext`][db-context] и миграция | Структура подтверждена кодом | Отдельный тест производительности не найден |
| Эффективный набор и область данных | Интеграция | API-тесты и `ValueSetDataEditorService` | Поведение описано и покрыто частично | Полный сквозной сценарий Studio не подтверждён тестами этого пакета |
| Регистрация редактора данных в Studio | Frontend | [`studioConfiguredRouteRegistry.test.ts`][studio-route-tests] и [`studio.value-set-data.runtime.test.ts`][studio-runtime-tests] | Маршрут, представления и поставщик функциональности проверяются существующими тестами | Браузерный сквозной сценарий не запускался |

## 6. Неподтверждённые свойства

- Отдельное тестовое свидетельство для события аудита не заявляется: событие не найдено.
- Семантика метки удаления (tombstone) для унаследованного элемента отсутствует в текущем API.
- Полный сквозной тестовый сценарий Studio не подтверждён тестами этого пакета.
- Инвалидация кэша как отдельное событие не подтверждена.

Проверки при ревью документации:

- [ ] Схема `ValueSet` не смешана с `ValueSetData`.
- [ ] Стабильные коды отделены от внутренних идентификаторов.
- [ ] Приоритет области указан одинаково во всех документах.
- [ ] Иерархия описывает физического и эффективного родителя отдельно.
- [ ] Имена прав совпадают с кодом.
- [ ] Требования UI не выданы за реализацию фронтенда; подтверждённая интеграция Studio отделена от общей фронтенд-платформа.

[api-tests]: ../../../tests/DMP.Platform.IntegrationTests/Configuration/ValueSetsApiIntegrationTests.cs
[bootstrap-tests]: ../../../tests/DMP.Platform.IntegrationTests/Configuration/GeneralMasterDataValueSetsBootstrapIntegrationTests.cs
[db-context]: ../../../src/Platform/DMP.Platform.ValueSets/Infrastructure/Persistence/ValueSetsDbContext.cs
[studio-route-tests]: ../../../src/Frontend/apps/studio/src/router/studioConfiguredRouteRegistry.test.ts
[studio-runtime-tests]: ../../../src/Frontend/apps/studio/src/features/value-set-data-editor/runtime/studio.value-set-data.runtime.test.ts

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
