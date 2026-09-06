---
id: DOC-03-03-07
title: 'Качество — Object Runtime'
type: assurance
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: object_runtime
holder: '@axelprosoft'
created_at: 2026-08-26 00:00
created_by: '@codex'
updated_at: 2026-09-03 14:49
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Качество — Object Runtime

[runtime-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/
[arch-tests]: ../../../tests/DMP.Platform.ArchTests/
[business-contract-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/BusinessObjectContractFoundationIntegrationTests.cs
[descriptor-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ObjectRuntimeDescriptorFoundationIntegrationTests.cs
[registry-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ObjectRuntimeDescriptorRegistryIntegrationTests.cs
[startup-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ObjectRuntimeDescriptorStartupValidationIntegrationTests.cs
[entrypoint-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ObjectRuntimeEntrypointIntegrationTests.cs
[provider-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/GenericRuntimeObjectProviderIntegrationTests.cs
[mutation-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ObjectMutationPipelineIntegrationTests.cs
[contract-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ObjectMutationContractIntegrationTests.cs
[boolean-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ObjectRuntimeBooleanCreateDefaultsIntegrationTests.cs
[common-foundation-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/CommonObjectFoundationIntegrationTests.cs
[common-policy-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/CommonObjectMutationPolicyIntegrationTests.cs
[quality-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/Backlog18ObjectRuntimeQualityIntegrationTests.cs
[application-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/ApplicationRuntimeIntegrationTests.cs

## 1. Назначение документа

Документ фиксирует проверяемые свойства качества Object Runtime, подтверждения тестами, известные разрывы и риски вычитки. Полный набор интеграционных тестов не выполнялся в рамках подготовки документации, потому что задача не меняет код.

## 2. Подтверждение тестами

| Область | Подтверждение | Что проверяется |
| --- | --- | --- |
| Построитель контракта | [BusinessObjectContractFoundationIntegrationTests][business-contract-tests] | Полный и минимальный контракт, метаданные командных действий и ассоциированных коллекций, унаследованные метаданные и предсказуемые ошибки проверки. |
| Общая модель объекта | [CommonObjectFoundationIntegrationTests][common-foundation-tests] | Общие свойства объекта и политика owned-коллекций, включая значение `Cascade` для агрегированной коллекции. |
| Общая политика изменения | [CommonObjectMutationPolicyIntegrationTests][common-policy-tests] | Политики create/update/delete, мягкого удаления, архивирования, восстановления и ограничения удаления. |
| Форма описания | [ObjectRuntimeDescriptorFoundationIntegrationTests][descriptor-tests] | Преобразование контракта в исполняемое описание без отдельного provider-описания. |
| Реестр | [ObjectRuntimeDescriptorRegistryIntegrationTests][registry-tests] | Поиск описания, нормализация составного кода типа объекта, неизвестный объект и дубли. |
| Совместимость при запуске | [ObjectRuntimeDescriptorStartupValidationIntegrationTests][startup-tests] | Совместимость опубликованных `ObjectType`, наборов данных, `View`, `Action`, `Workflow` и границы миграции. |
| Точка входа runtime | [ObjectRuntimeEntrypointIntegrationTests][entrypoint-tests] | Передача операции исполнителю описания и раннее отклонение неизвестных набора данных или действия. |
| Универсальный исполнитель | [GenericRuntimeObjectProviderIntegrationTests][provider-tests] | Фильтры flags, области удалённых и архивных строк, запросы и изменения иерархии, обработчики чтения, корень запроса репозитория, фильтрация, сортировка, группировка, дополнительные поля и предсказуемое отсутствие хранилища. |
| Конвейер изменения | [ObjectMutationPipelineIntegrationTests][mutation-tests] | Порядок выполнения, блокировка проверками, транзакция, аудит и outbox, снимки, набор изменений, профили, контрактные проверки и жизненный цикл. |
| Контракты изменения | [ObjectMutationContractIntegrationTests][contract-tests] | Каноническая проекция запроса, разделение коллекций, профиль группового действия по умолчанию и проекция ответа. |
| Значения Boolean по умолчанию | [ObjectRuntimeBooleanCreateDefaultsIntegrationTests][boolean-tests] | Отсутствие неявных primitive defaults, приоритет явных значений и поставщиков, а также отклонение отсутствующих обязательных Boolean и числовых значений. |
| Срез ProductDefinition | [Backlog18ObjectRuntimeQualityIntegrationTests][quality-tests] | Состав runtime, опубликованные baseline-артефакты, список/карточка/создание/изменение/действия Nomenclature и проверки. |
| Фасад runtime | [ApplicationRuntimeIntegrationTests][application-tests] | Разрешение `View`, права, ссылочные lookup-ы, lookup для типа `AbstractReferenceOnly`, создание на основании существующего, действия, групповые действия, каскадное удаление, проекция workflow и сценарии состояния списка. |

## 3. Надёжность

| Свойство качества | Текущая защита | Разрыв |
| --- | --- | --- |
| Предсказуемые ошибки описания | Неизвестные описания, наборы данных и действия, а также дубли имеют явные ошибки и тесты. | Единая модель ошибок всех HTTP-маршрутов требует общего решения платформы. |
| Отклонение неподдерживаемых операций | Неподдерживаемые create/update/delete/action/list отклоняются до выполнения. | Перенос этой гарантии на Java/PostgreSQL относится к будущему архитектурному этапу. |
| Проверка до записи | Конвейер блокирует выполнение при ошибках и не вызывает аудит/outbox для недействительного результата. | Гарантия транзакции между возможностями платформы не определена. |
| Совместимость при запуске | Фоновая служба проверяет описания относительно опубликованных артефактов; несовместимость приводит к ошибке запуска. | Политика выпуска при ошибке проверки относится к Operations и не является гарантией Object Runtime. |
| Сокрытие чувствительных значений в аудите | Приёмник аудита скрывает чувствительные поля описания. | Полная классификация данных находится за пределами пакета. |

## 4. Наблюдаемость и диагностика

Текущий код передаёт идентификатор корреляции во внешних оболочках аудита и outbox: в `AuditRecord` и `IntegrationEventEnvelope`, а не в `ObjectMutationAuditDetails` или `ObjectMutationCompletedIntegrationEventPayload`. Для проверки совместимости описаний используются предсказуемые типы исключений. Отдельные метрики, трассировочные интервалы и проверки работоспособности Object Runtime не подтверждены как реализованная возможность этого пакета.

## 5. Производительность

| Область | Текущее поведение | Риск |
| --- | --- | --- |
| Постраничная выдача | Размер страницы ограничен значением `500`. | Предел зашит в коде и не связан с эксплуатационной политикой. |
| Запрос репозитория | Для источников запросов фильтры, сортировка и группировка могут применяться до постраничной выдачи. | Сложные предикаты и Standalone Dataset не покрыты. |
| Постобработка workflow и поиска | Фасад runtime может загрузить до `500` строк для добавления виртуальных полей workflow или поиска. | Большие списки требуют производственного проектирования запросов. |
| Дополнительные значения | Runtime дополняет дополнительные значения и координирует их фильтрацию и сортировку. | Производительность хранилищ дополнительных значений требует проверки в каждом модуле. |
| Агрегаты | Контракт агрегатов существует, но Object Runtime их не выполняет. | Требуется решение по Dataset / Read Query. |

## 6. Известные разрывы

| Разрыв | Статус сведения | Маршрут |
| --- | --- | --- |
| Владелец Dataset / Read Query | Будущая доработка | [Трассировка](90_traceability.md): `ORT-DEC-01 — владелец Dataset / Read Query` |
| Групповые операции по фильтру и профиль импорта | Будущая доработка | [Трассировка](90_traceability.md): `ORT-DEC-02 — групповые операции по фильтру и профиль импорта` |
| Производственное обновление и предварительное заполнение кэша | Будущая доработка | [Трассировка](90_traceability.md): `ORT-DEC-03 — обновление и предварительное заполнение кэша runtime` |
| Семантика целевой Java/PostgreSQL-платформы | Будущая доработка | [Трассировка](90_traceability.md): `ORT-DEC-06 — семантика целевой Java/PostgreSQL-платформы` |
| Полная принадлежность интерфейсного runtime | Владелец другой области | фронтенд-платформа |

## 7. Проверки документации

Для этого пакета документации выполняются проверки на отсутствие ссылок на сырьевые каталоги миграции, неопределённых маркеров и whitespace-ошибок в diff. Фактический список выполненных команд фиксируется в отчёте после подготовки.

Интеграционные тесты runtime не требуются для этого изменения документации; их следует запускать выборочно только при изменении реализации.

## 8. Проверки файловой возможности

Для file property и file action обязательны проверки descriptor-only projection,
невалидного/просроченного ref, attach/detach, null clear, tenant boundary и
concurrency. Binary не должен появляться в list/details snapshots.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.2 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Проверки файловой возможности | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
