---
id: DOC-04-03-10
title: 'Использование ValueSetData - 03 Plant Structure'
type: design
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 03_plant_structure
holder: '@axelprosoft'
created_at: 2026-08-14 12:39
created_by: '@axelprosoft'
updated_at: 2026-08-31 22:00
last_modified_by: '@A-Zhigalin'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Использование ValueSetData - 03 Plant Structure

## 1. Назначение документа

Документ фиксирует использование Platform ValueSetData в модуле `03 Plant Structure`.

Системные enum ПР03 описаны в `02_domain_model.md` и в этом документе не дублируются. Поля контроля учетных разрезов переиспользуют внешний SystemEnum `InventoryLocationControlType` из `01_general_master_data`; он также не является `ValueSetData` и не регистрируется повторно модулем ПР03.

## 2. Решение

Модуль не вводит собственные `ValueSetData`.

## 3. Внешние справочники

Поля `InventoryStatus`, `TransferYieldDefaultStatus`, `TransferRejectDefaultStatus`, `TransferScrapDefaultStatus` и `WarehouseBin.InventoryStatus` предназначены для будущих ссылок на внешний справочник статусов запаса.

В v1 они сохраняются только как nullable GUID-колонки и не публикуются в Object Runtime, baseline и UI. Эти поля не являются `ValueSetData` модуля; после появления стабильного контракта `InventoryStatus` они активируются как object references.

## 4. Baseline

Baseline модуля не регистрирует `ValueSetData`.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-08-31 22:00 +04:00 | A-Zhigalin (@A-Zhigalin) | Назначение документа; Внешние справочники | Уточнения документации | [cb99e83e](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/cb99e83e553f9dc034dd6808f07d230b09ceed28) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-14 12:39 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [abbdee0a](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/abbdee0a5a61f2bb2fa49cfafd1858b7100cbd8a) |
