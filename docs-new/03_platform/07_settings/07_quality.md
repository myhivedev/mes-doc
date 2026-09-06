---
id: DOC-03-07-07
title: 'Качество - Settings'
type: assurance
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: settings
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

# Качество - Settings

## 1. Назначение документа

Документ фиксирует проверяемые свойства Settings MVP, тестовые свидетельства и
ограничения покрытия. Он не объявляет реализованными распределённый cache,
restart/reload или custom validation pipeline.

## 2. Проверяемые гарантии

| Гарантия | Условие | Ожидаемый результат | Доказательство |
| --- | --- | --- | --- |
| Typed key сохраняет identity и type | Создан `ModuleSettingKey<T>` | ModuleCode, SettingCode и `SettingValueType` доступны consumer | [catalog tests][catalog-tests] |
| Catalog validation детерминирована | Builder собирает один и тот же catalog | Fingerprint совпадает | [catalog tests][catalog-tests] |
| Duplicate key отвергается | Один key добавлен дважды | `InvalidOperationException` | [catalog tests][catalog-tests] |
| Invalid default отвергается | Default не соответствует type/rules | Snapshot не строится | [catalog tests][catalog-tests] |
| Baseline сохраняет catalog snapshot | Публикуется `SystemBaseline` version | JSON и fingerprint записаны | [publication tests][publication-tests] |
| Corporate version не дублирует baseline snapshot | Публикуется Corporate version | Snapshot fields остаются null | [publication tests][publication-tests] |
| Runtime override сохраняется и сбрасывается | Upsert/reset через service | Effective value меняется, reset удаляет local row | [storage tests][storage-tests] |
| Scope inheritance работает | Есть значения в ancestor/selected scope | Ближайшее специфичное значение побеждает | [storage tests][storage-tests] |
| User preference изолирована | Два tenant/user context | Каждый видит только свои preferences | [preference tests][preference-tests] |
| API возвращает каталог и effective value | Вызван Settings page endpoint | DTO содержит navigation, definitions и audit descriptors | [API tests][api-tests] |
| Scope access не пересекает tenant boundary | Запрос чужого tenant/site | Возвращается forbidden result | [access tests][access-tests] |

## 3. Известные ограничения

- интеграционные тесты в рамках подготовки документации не запускались;
- проверка custom validator codes не выполняет внешний validator;
- входные запросы изменения и сброса не принимают ожидаемый token конкурентности;
- memory cache локален процессу; общая distributed invalidation policy не задана;
- apply policies `RequiresRestart` и `RequiresReload` не запускают отдельный
  механизм применения;
- отдельный benchmark есть для accessor, но промышленный SLO Settings не закреплён.

## 4. Наблюдаемость

Settings передаёт action codes и correlation id в Audit History. Отдельные
метрики cache hit/miss, длительности разрешения catalog, количества validation
ошибок и health check Settings в текущем контракте не зафиксированы. Это
ограничение эксплуатационной готовности, а не отсутствие описания текущих API.

## 5. Источники и статус проверки

Код и тестовые файлы сверены с `origin/master@3e2037ac37683eef331d70223c6babfc61aa0539`.
Тесты в рамках подготовки документации не запускались.
[catalog-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsCatalogContractTests.cs
[publication-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsCatalogPublicationIntegrationTests.cs
[storage-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsStorageIntegrationTests.cs
[preference-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsUserPreferenceIntegrationTests.cs
[api-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsApiIntegrationTests.cs
[access-tests]: ../../../tests/DMP.Platform.IntegrationTests/Settings/SettingsAccessGuardIntegrationTests.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
