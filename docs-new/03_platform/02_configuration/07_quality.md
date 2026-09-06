---
id: DOC-03-02-07
title: 'Качество — Configuration'
type: assurance
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: configuration
holder: '@axelprosoft'
created_at: 2026-08-25 17:20
created_by: '@axelprosoft'
updated_at: 2026-09-03 14:49
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Качество — Configuration

[tests]: ../../../tests/DMP.Platform.IntegrationTests/Configuration/
[runtime-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/
[project]: ../../../src/Platform/DMP.Platform.Configuration/
[package-validator]: ../../../src/Platform/DMP.Platform.Configuration/Application/Baseline/Services/ConfigurationBaselinePackageValidator.cs
[artifact-validator]: ../../../src/Platform/DMP.Platform.Configuration/Domain/Services/ArtifactValidator.cs
[version-validator]: ../../../src/Platform/DMP.Platform.Configuration/Application/Validation/Services/ConfigurationVersionValidationService.cs
[access-guard]: ../../../src/Platform/DMP.Platform.Configuration/Application/Services/Core/ConfigurationAccessGuard.cs

## 1. Назначение документа

Документ фиксирует проверяемые свойства Configuration, текущие тестовые свидетельства и известные ограничения. Он не заменяет test plan и не объявляет покрытие, которое не подтверждено запуском тестов.

## 2. Надёжность

| Отказ | Гарантия | Предел | Восстановление | Проверка |
| --- | --- | --- | --- | --- |
| Некорректная schema, reference или localization issue | Validation блокирует публикацию | Гарантия действует только для проверок, реализованных текущим validator pipeline | Исправить draft и повторить validation | `ArtifactValidatorIntegrationTests`, validation tests |
| Отсутствует опубликованная parent-base version | Non-root draft не создаётся | Правило действует для текущей цепочки областей | Сначала опубликовать parent version | `ConfigurationContextCreateDraftIntegrationTests`, `EffectiveConfigurationExactParentBaseChainIntegrationTests` |
| Bootstrap/import завершается ошибкой | Failed draft сохраняет диагностическое состояние | Автоматическое внешнее восстановление не гарантируется | Диагностировать и повторить bootstrap | `ConfigurationBootstrapIntegrationTests`, `ConfigurationBaselineImportIntegrationTests` |

## 3. Наблюдаемость

| Сигнал | Источник | Корреляция | Потребитель | Диагностический пробел |
| --- | --- | --- | --- | --- |
| Failed draft diagnostics | Bootstrap/import pipeline | Correlation id операции | Оператор и support | Единый health/metrics contract не закреплён |
| Publication/change records | Publication и authoring pipeline | Scope/version и actor fields | Configuration и будущий Audit History | Retention/export и общий audit event не определены |
| Cache invalidation | Publication/archive и effective cache | Scope keys | Runtime consumers | Cross-instance invalidation не является текущей гарантией |

## 4. Производительность

| Сценарий нагрузки | Предел или SLO | Метод | Результат |
| --- | --- | --- | --- |
| Effective configuration cache hit/miss | SLO не установлен | Runtime cache integration tests и profiling | Поведение кэша проверяется; production SLO не установлен |
| Bootstrap/import published baseline | SLO не установлен | Bootstrap profiling/integration tests | Тесты существуют; нормативный предел не установлен |
| Explorer/editor API | SLO не установлен | API integration tests | Контрактные проверки есть; нагрузочный результат не зафиксирован |

## 5. Проверки и результаты

| Сценарий проверки | Уровень | Источник теста | Результат запуска | Пробел |
| --- | --- | --- | --- | --- |
| Authoring и lifecycle | Integration | `ConfigurationAuthoringIntegrationTests`, `ConfigurationApiIntegrationTests` | Тесты существуют; в рамках этого документа не запускались | Нет сводной матрицы endpoint -> test |
| Scope и parent chain | Integration | `EffectiveConfigurationExactParentBaseChainIntegrationTests`, `ConfigurationContextCreateDraftIntegrationTests` | Тесты существуют; в рамках этого документа не запускались | Нет отдельного production acceptance criteria |
| Baseline | Integration | `ConfigurationBootstrapIntegrationTests`, `ConfigurationBaselineImportIntegrationTests`, `BaselinePackageCatalogIntegrityIntegrationTests` | Тесты существуют; в рамках этого документа не запускались | Recovery и downgrade проверены не как единая эксплуатационная процедура |
| Schema и validation | Integration | `ArtifactSchemaRegistryIntegrationTests`, `ArtifactValidatorIntegrationTests` и validation tests | Тесты существуют; в рамках этого документа не запускались | Нет полной матрицы всех артефактов и правил |
| Editor | Integration | `ArtifactEditor*IntegrationTests`, `ArtifactCommandServiceIntegrationTests` | Тесты существуют; в рамках этого документа не запускались | Нужна отдельная проверка contract coverage |
| Specialized artifacts | Integration | `*NodeBuilder*`, `*PublishValidation*` и lifecycle tests | Наборы тестов существуют для части типов | Не для каждого типа есть отдельная матрица тестов; спецификации `Report`, `Output` и `NumberingRule` созданы, но их покрытие требует отдельной проверки |
| Runtime consumption | Integration | [runtime tests][runtime-tests] | Тесты существуют; в рамках этого документа не запускались | Production SLO и cross-instance cache не проверены |

### 5.1. Матрица проверок перед публикацией

Матрица показывает не один универсальный validator, а последовательность проверок
на пути к публикации. Статус «подтверждено» означает, что соответствующая проверка
найдена в текущем коде; это не означает, что весь Configuration-контракт покрыт
одним тестом или что проверка выполняется на каждом API-пути.

Ссылки между записями одной проверяемой draft-версии допустимы: весь набор
публикуется как единый слой после общей проверки. Это не означает, что сама
draft-версия может участвовать в effective configuration: effective resolver
выбирает только опубликованный слой; `Draft` и `Failed` не возвращаются
runtime-потребителю.

| Проверка | Что проверяется | Где выполняется | Результат при ошибке | Статус MVP |
| --- | --- | --- | --- | --- |
| базовый пакет конфигурации | `ModuleCode`, уникальность `OriginKey`, формат `OriginKey`, `ParentOriginKey`, циклы родителей, устаревшая форма local condition и ссылки на зарегистрированные `Action`, `View`, `Dataset`, `Permission` | [Baseline package validator][package-validator] при import/bootstrap | Пакет не принимается; import не продолжается | Подтверждено для baseline package |
| Схема артефакта | Наличие schema, обязательные свойства, типы значений, дочерние коллекции, schema rules и специальные семантические проверки артефакта | [Artifact validator][artifact-validator] при authoring и import | Команда или import получает ошибку validation | Подтверждено для реализованных правил |
| Значения свойств | Допустимость значений по schema, статическим каталогам и динамическим источникам; для `Bool`, `Number` и `Json` проверяется формат, а не только наличие текста | Schema registry и проверки, вызывающие [Artifact validator][artifact-validator] | Команда или import получает ошибку validation | Подтверждено не для каждого свойства; полный каталог остаётся в `artifact_types/` |
| Состояние версии | Версия существует и находится в `Draft`; только draft допускается к publish validation | [Version validator][version-validator] и publication service | Публикация блокируется | Подтверждено |
| Идентичность записи | Уникальность `OriginKey` в версии и допустимость локального `ParentEntryId` | [Version validator][version-validator]; уникальный индекс persistence | Публикация блокируется или запись не сохраняется | Подтверждено |
| Контекст модуля и типа | Известные `ModuleCode`, `ObjectTypeCode` и совместимость модуля с типом объекта | [Version validator][version-validator] | Публикация блокируется | Подтверждено |
| Ссылки на артефакты и каталоги | Разрешаются ссылки на `View`, `Action`, `Dataset`, `Permission`, `Rule`, `ValueSet` и `SystemEnum` по доступному каталогу | [Version validator][version-validator] | Публикация блокируется при неизвестной ссылке | Подтверждено для перечисленных ссылок |
| Локализации | Конфликты локализованных значений, включая несовместимые варианты явного удаления в слоях | [Version validator][version-validator] | Публикация блокируется при обнаруженном конфликте | Подтверждено для реализованных конфликтов |
| Специализированные правила | Связь `NumberingRule` с `ObjectType` и полем, разрешение нумерации, тип поля, момент назначения, источники и неоднозначные правила; отдельные проверки `Report` | [Version validator][version-validator], [Artifact validator][artifact-validator] и `ReportPublishValidator` | Публикация блокируется | Подтверждено частично, только для реализованных правил |
| Цепочка версий | Допустимый parent-base для `Corporate`, `Tenant` и `Site`; опубликованный статус базы; отсутствие циклов при разрешении эффективной конфигурации | Authoring/context services и effective resolver; часть локальных связей проверяется version validator | Draft не создаётся или эффективная конфигурация не строится | Подтверждено, но не является одной проверкой содержимого версии |
| Контекст и права операции | Разрешение действующего пользователя читать или менять scope и выполнять publish | [Configuration access guard][access-guard] до authoring/operation | Операция отклоняется до validation | Подтверждено отдельно от validation |
| Draft и Failed в эффективной конфигурации | Effective resolver выбирает только опубликованный слой; в сохранённой lineage допускается опубликованный или архивный parent; `Draft` и `Failed` не участвуют | Effective resolver и запрос выбора published version | Draft/Failed не возвращаются как эффективная конфигурация | Подтверждено |
| Пустой draft | Отсутствие активных entries | [Version validator][version-validator] | Warning; само по себе не блокирует publish | Подтверждено как предупреждение |

Проверки persistence и атомарности публикации не являются строками этой матрицы:
они описаны в `04_runtime.md` как свойства операции сохранения и транзакции, а не
как semantic validation. Аналогично, отсутствие runtime-поддержки отдельного
значения артефакта не исправляется этой матрицей: такое расхождение остаётся в
`90_traceability.md`.

### 5.2. Проверяемые инварианты

- published effective configuration строится из допустимой цепочки областей и версий;
- non-root draft создаётся только при наличии опубликованной parent-base version;
- baseline import и bootstrap не должны нарушать существующий published state;
- validation блокирует публикацию при обнаруженных schema, reference или localization issues;
- доступ к чужому scope не должен превращаться в успешное чтение или изменение;
- editor command и persistence pipeline сохраняют согласованное состояние артефакта.

## 6. Неподтверждённые свойства

### 6.1. Известные ограничения

- Этот документ содержит карту тестов, но не текущую матрицу покрытия по каждому endpoint и артефакту.
- Performance, cache warm-up, observability и production recovery требуют отдельной проверки; наличие profiling tests не означает установленный production SLO.
- Общий audit trail, distributed consistency и миграция на Java/PostgreSQL не являются завершёнными quality guarantees Configuration.

### 6.2. Требуемые дальнейшие проверки

1. Сформировать матрицу `contract -> test` для публичных Configuration endpoints.
2. Зафиксировать минимальные performance и cache acceptance criteria.
3. Добавить отдельные проверки миграций и восстановления после неуспешного bootstrap/import.
4. Согласовать границы интеграционных тестов с Tenant/Security, Object Runtime и фронтенд-платформа.

### 6.3. Источники подтверждения

Кодовая область — [Configuration project][project], тестовые классы — [Configuration integration tests][tests] и runtime tests.

## 7. Проверки файловой схемы

Публикационные проверки должны отклонять настройки файла для datatype, отличного
от `File`, недопустимые extension/size/count и неподдерживаемый editor component.
Проверка не означает загрузку бинарного содержимого на стороне Configuration.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.2 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Проверки файловой схемы | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
