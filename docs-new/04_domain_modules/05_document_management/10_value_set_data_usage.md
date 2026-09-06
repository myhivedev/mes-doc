---
id: DOC-04-05-10
title: 'Использование настраиваемых наборов значений — Управление документацией'
type: design
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: document_management
holder: '@axelprosoft'
created_at: 2026-08-20 17:32
created_by: '@axelprosoft'
updated_at: 2026-09-03 16:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Использование настраиваемых наборов значений — Управление документацией

## 1. Итог

Модуль «Управление документацией» не использует и не поставляет данные настраиваемых наборов значений (`ValueSetData`).

## 2. Системные перечисления

Системное перечисление «Вид содержимого документа» (`DocumentContentKind`) описано в [02_domain_model.md](02_domain_model.md) и в этом документе не повторяется.


## 3. Файловые значения

Файловые ресурсы не являются Value Set и не добавляются в каталог значений.
В Configuration хранятся только тип и метаданные политики; descriptor и
жизненный цикл ресурса принадлежат Content Storage.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | Системные перечисления; Файловые значения | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 1.0 | 2026-08-21 12:16 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #31: 05 document management docs | [PR #31](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/31) |
| 0.1 | 2026-08-21 11:33 +03:00 | Донских Сергей (@Sergey020726) | Создание документа | docs: add document management module documentation | [ef648f68](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/ef648f6831ca38c686a44d67ae8abfe79815c9eb) |
