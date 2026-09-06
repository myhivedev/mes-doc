---
id: DOC-04-02-10
title: 'Использование ValueSetData - 02 Product & Process Definition'
type: design
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 02_product_process_definition
holder: '@axelprosoft'
created_at: 2026-08-19 12:02
created_by: '@axelprosoft'
updated_at: 2026-08-19 17:22
last_modified_by: '@VeronikaV2121'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Использование ValueSetData - 02 Product & Process Definition

## 1. Назначение документа

Документ фиксирует использование платформенного механизма `ValueSetData` в модуле `02 Product & Process Definition`.

## 2. Решение

Модуль не вводит собственные `ValueSetData`.

## 3. Справочники и классификаторы

`LabourType`, `ProcessingOperation`, `ManufacturingOperationType` и `WorkEnvironment` являются объектами-справочниками модуля.

`PaymentGroup` является внешним справочником ресурсного модуля.

Эти ссылки не являются `ValueSetData` модуля.

## 4. Значения реквизитов GMD

`UseConditionParameterValue` и `ProductCompositionParameterValue` используют общий контракт значений реквизитов GMD.

Если реквизит GMD настроен на выбор значения из `ValueSetData`, модуль использует это значение через контракт GMD и не регистрирует собственный набор значений.

## 5. Baseline

Baseline модуля не регистрирует `ValueSetData`.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-19 12:02 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [87c0b0a5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/87c0b0a51fdabf232ce704082d41545ae007bedc) |
