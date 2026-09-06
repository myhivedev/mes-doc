---
id: DOC-03-13-90
title: 'Трассировка — Platform Content Storage'
type: traceability
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: content_storage
holder: '@axelprosoft'
created_at: 2026-09-03 00:00
created_by: '@codex'
updated_at: 2026-09-03 16:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
---

# Трассировка — Platform Content Storage

| Backlog | Кодовая область | Документ |
| --- | --- | --- |
| `DM-CAP-CORE-01..07` | Contracts, provider, API, cleanup | `03`, `04`, `05`, `08` |
| `DM-CAP-CONF-01..04,06` | Configuration schemas, baseline, materializers | platform Configuration docs |
| `DM-CAP-RT-01..06,08..10` | Object Runtime mutation and projections | Object Runtime docs |
| `DM-CAP-FE-01..09` | API client, runtime-react, generic picker/renderer | Frontend Platform docs |
| `DM-CAP-DEMO-01..06` | `NomenclatureAttachment_UploadFiles` in embedded collection toolbar and `NomenclatureAdditionalUnit.File` property | Product Definition README |
| `DM-CAP-DM-01..08` | ContentRef boundary and deferred DM-Q-008 | Document Management docs |

## Границы и отложенные решения

- `DM-Q-008` (динамические обязательные поля типа документа) не нужен для
  Content capability MVP и не реализуется;
- provider registry, external storage, preview/OCR/antivirus/full-text и
  resumable upload относятся к P1 или более позднему этапу;
- demo не является реализацией Document Management и не импортирует его
  contracts, baseline или permissions.

## Подтверждение

Основные evidence: `DMP.Platform.Content`, public contracts, Object Runtime
file hook, frontend generic file mapping, demo baseline and the focused build,
typecheck and runtime-react test runs documented in the implementation change.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | Трассировка — Platform Content Storage | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
