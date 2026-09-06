---
id: DOC-04-01-10
title: 'Использование ValueSetData - 01 General Master Data'
type: design
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 01_general_master_data
holder: '@axelprosoft'
created_at: 2026-08-14 09:15
created_by: '@axelprosoft'
updated_at: 2026-08-19 17:22
last_modified_by: '@VeronikaV2121'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Использование ValueSetData - 01 General Master Data

## 1. Назначение документа

Документ фиксирует наборы значений, которые использует модуль General Master Data.

Системные enum модуля описаны в `02_domain_model.md`. Механизм хранения и переопределения ValueSetData описывается в платформенной документации и в этом документе не повторяется.

## 2. ValueSet модуля

| ValueSetCode | Где используется | Значения по умолчанию | Политика ведения |
|---|---|---|---|
| `GMD.ContractorContactType` | `ContractorContact.ContactType` | `Supervisor`, `ContactPerson` | Начальные значения поставляются модулем. Предприятие может добавлять свои типы контактов. |

## 3. Начальные значения

### `GMD.ContractorContactType`

| Код значения | Русское название | Назначение |
|---|---|---|
| `Supervisor` | Руководитель | Контакт руководителя контрагента. |
| `ContactPerson` | Контактное лицо | Обычное контактное лицо контрагента. |

## 4. Поля со ссылкой на произвольный ValueSet

| Поле | Назначение |
|---|---|
| `NomenclatureParameter.ValueSetCode` | Указывает общий набор значений для реквизита НП, если `AllowedValuesSource = ValueSet`. |
| `NomenclatureParameterValueBase.ReferenceCode` | Хранит код выбранного значения из ValueSet для строки значения реквизита. |

## 5. Правила использования

- `ValueSetCode` используется только для настраиваемых общих наборов значений.
- Фиксированные enum из ПР01 описываются как системные enum в `02_domain_model.md`.
- `Contractor.Type` и `Contractor.Category` не используют ValueSetData.
- `Nomenclature.WarehouseBinControlType`, `Nomenclature.TransportUnitControlType`, `Nomenclature.ObtainMethod` и `Nomenclature.ProcessType` не используют ValueSetData.
- Для `ContractorContact.ContactType` используется ValueSetData, потому что список типов контактов может отличаться у предприятия и не участвует в зафиксированной бизнес-логике ПР01.
- Для одного `Contractor + ContactType` допускается один контакт.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-14 09:15 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [7df442b0](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7df442b0d2aa5b6974014dc33bb9c34b7f978d9e) |
