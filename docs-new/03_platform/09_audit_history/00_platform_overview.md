---
id: DOC-03-09-00
title: 'Обзор платформенной области - Audit History'
type: design
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: audit_history
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed
---

# Обзор платформенной области - Audit History

## 1. Назначение области

Audit History предоставляет общий прикладной контракт для записи значимых
операций и хранит записи аудита в отдельной модели хранения. Потребители
передают `AuditRecord` через `IAuditHistoryWriter`; область не знает бизнес-смысл
операции и не формирует его из UI.

В текущем MVP реализованы запись, чтение снимка записей и два провайдера
хранения: SQL Server и хранение в памяти (`InMemory`). Отдельный HTTP API, экран просмотра истории
и универсальная модель пользовательской истории в коде не подтверждены.

## 2. Место в платформе

```text
Tenant/Security, Object Runtime, Workflow, Settings, Domain Modules
                              |
                              | AuditRecord
                              v
                   IAuditHistoryWriter
                              |
                 PersistentAuditHistoryWriter
                              |
                audit_history.audit_records
```

Общая карта межмодульных взаимодействий находится в
[`07_integration_architecture.md`](../../02_architecture/07_integration_architecture.md).
Этот пакет описывает только локальную границу Audit History.

## 3. Основные возможности

| Возможность | Назначение | Статус | Основание |
| --- | --- | --- | --- |
| Запись `AuditRecord` через `IAuditHistoryWriter.WriteAsync` | Приём записи значимой операции от потребителя | Подтверждено MVP | `AuditRecord`, `IAuditHistoryWriter`, провайдеры хранения |
| Чтение `IReadOnlyCollection<AuditRecord>` через `GetSnapshot` | Получение полного снимка доступных записей | Подтверждено, но ограничено | Интерфейс и реализации; отдельный HTTP-контракт чтения отсутствует |
| Хранение в SQL Server | Сохранение записей в схеме `audit_history` и таблице `audit_records` | Подтверждено MVP | `AuditHistoryDbContext`, `AuditHistoryDatabaseInitializer` |
| Хранение в памяти | Тестовые и локальные запуски без SQL Server | Подтверждено MVP | `InMemoryAuditHistoryWriter` |

Audit History не определяет бизнес-смысл операции, не формирует записи из UI и
не предоставляет в текущем MVP отдельный HTTP API или экран просмотра истории.

## 4. Ключевые решения

| Решение | Суть | Документ-владелец |
| --- | --- | --- |
| Потребитель формирует запись | Область-потребитель выбирает `ActionCode`, контекст и `Details`; Audit History сохраняет полученное значение | [`03_contracts.md`](03_contracts.md); [`04_runtime.md`](04_runtime.md) |
| Audit History не владеет предметной историей | История Workflow, история бизнес-объекта, техническая трассировка и интеграционные события описываются их владельцами | [`01_scope.md`](01_scope.md); [`05_security_and_audit.md`](05_security_and_audit.md) |
| Текущий интерфейс чтения ограничен снимком | `GetSnapshot` возвращает коллекцию без фильтрации, пагинации и HTTP-представления | [`03_contracts.md`](03_contracts.md); [`04_runtime.md`](04_runtime.md) |
| Политика production-хранения не задана кодом | SQL Server выбирается при наличии `ConnectionStrings:Platform`, иначе используется InMemory; целевая политика находится в трассировке | [`08_operations.md`](08_operations.md); [`90_traceability.md`](90_traceability.md) |

## 5. Зависимости

| Зависимость | Использование | Владелец |
| --- | --- | --- |
| Tenant/Security, Object Runtime, Workflow, Settings и Domain Modules | Формируют и передают `AuditRecord` через `IAuditHistoryWriter` | Каждая область владеет смыслом своей операции |
| Foundation и Platform Runtime | Предоставляют контекст tenant, пользователя и корреляции, если он нужен потребителю | Foundation / Platform Runtime |
| Host composition | Регистрирует провайдер записи и запускает инициализацию базы | API host и Audit History |
| Integration Events | Отдельный канал доставки событий; запись аудита не является событием | Integration Events |
| фронтенд-платформа | Владеет общим интерфейсом и маршрутизацией; собственный экран просмотра Audit History не подтверждён | фронтенд-платформа |

## 6. Статус реализации

| Возможность | Статус сведения | Основание |
| --- | --- | --- |
| Запись `AuditRecord` через `IAuditHistoryWriter.WriteAsync` | Подтверждено MVP | `AuditRecord`, `IAuditHistoryWriter`, провайдеры хранения |
| Чтение `IReadOnlyCollection<AuditRecord>` через `GetSnapshot` | Подтверждено, но ограничено | Интерфейс и реализации; отдельный HTTP-контракт чтения отсутствует |
| Хранение SQL Server в схеме `audit_history` и таблице `audit_records` | Подтверждено MVP | `AuditHistoryDbContext`, `AuditHistoryDatabaseInitializer` |
| Провайдер хранения в памяти для тестовых или локальных запусков | Подтверждено MVP | `InMemoryAuditHistoryWriter` |
| Связь записи с tenant, user и correlation | Подтверждено MVP | Поля `AuditRecord` и вызовы потребителей |
| Неизменяемость на уровне базы данных, политика хранения и архивирование | Не подтверждено | В коде нет политики хранения или запрета изменения на уровне базы данных |
| Общая пользовательская история объекта | Будущая доработка | Отдельная модель и интерфейс чтения не найдены |

## 7. Состав документов

| Документ | Содержание |
| --- | --- |
| `01_scope.md` | Граница области и маршрутизация обязанностей соседних владельцев |
| `02_architecture.md` | Компоненты, модель записи и модель хранения |
| `03_contracts.md` | `AuditRecord`, `IAuditHistoryWriter` и границы контракта |
| `04_runtime.md` | Порядок записи, сериализация и чтение снимка |
| `05_security_and_audit.md` | Контекст, чувствительные значения и ограничения доступа |
| `07_quality.md` | Тестовые свидетельства и пробелы покрытия |
| `08_operations.md` | Регистрация, инициализация базы и диагностика |
| `90_traceability.md` | Маршрут старых материалов, ограничения MVP и открытые темы |

Отдельный `06_user_experience.md` не создаётся: собственная интерфейсная
функция, маршрут или экран просмотра Audit History в текущем коде не подтверждены.

### 7.1 Локальные термины

Таблица повторяет только термины, необходимые для чтения этого пакета. Общий
список терминов и их определения находятся в [платформенном глоссарии](../../11_glossary/platform_terms.md).

| Русский термин | Техническое имя | Значение | Источник |
| --- | --- | --- | --- |
| Запись аудита | `AuditRecord` | Запись о значимой операции, переданная потребителем в Audit History | [Платформенный глоссарий][glossary] |
| Средство записи аудита | `IAuditHistoryWriter` | Прикладной порт записи и чтения снимка записей аудита | [Платформенный глоссарий][glossary] |
| Код операции аудита | `ActionCode` | Строковый код операции, который назначает потребитель | [Платформенный глоссарий][glossary] |
| Контекст корреляции | `CorrelationId` | Идентификатор, связывающий запись с выполняемым сценарием | [Платформенный глоссарий][glossary] |
| Снимок записей | `GetSnapshot` | Набор записей, возвращаемый текущей реализацией без HTTP-пагинации | [Контракт средства записи][writer] |
| Детали записи | `Details` / `DetailsJson` | Произвольный объект при записи и его JSON-представление в модели хранения | [Контракт записи][record] |

### 7.2 Источники

- [проект Audit History][project];
- [контракт записи][record];
- [контракт средства записи][writer];
- [интеграционные тесты][tests].

[project]: ../../../src/Platform/DMP.Platform.AuditHistory/
[record]: ../../../src/Platform/DMP.Platform.AuditHistory/Abstractions/AuditRecord.cs
[writer]: ../../../src/Platform/DMP.Platform.AuditHistory/Abstractions/IAuditHistoryWriter.cs
[tests]: ../../../tests/DMP.Platform.IntegrationTests/
[glossary]: ../../11_glossary/platform_terms.md

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
