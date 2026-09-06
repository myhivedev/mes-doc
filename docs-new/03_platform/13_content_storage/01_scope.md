---
id: DOC-03-13-01
title: 'Границы — Platform Content Storage'
type: scope
status: approved
version: '1.0'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: content_storage
holder: '@axelprosoft'
created_at: 2026-09-03 00:00
created_by: '@codex'
updated_at: 2026-09-03 14:49
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: authored
---

# Границы — Platform Content Storage

## Входит

- DB-backed хранение бинарных файлов и metadata;
- upload session, `ContentRef`, descriptor, stream read;
- tenant/owner authorization, idempotency, TTL и cleanup;
- generic integration с Object Runtime и Frontend Runtime.

## Не входит

- Document/DocumentType/Folder и бизнес-правила модуля 05;
- внешние DMS/object-storage providers, secrets, preview, OCR, antivirus и
  full-text index;
- отдельный предметный UI или route-specific frontend code.

Соседние области используют capability через public contracts; provider остаётся
заменяемой внутренней реализацией.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
