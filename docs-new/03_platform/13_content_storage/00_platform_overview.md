---
id: DOC-03-13-00
title: 'Обзор — Platform Content Storage'
type: design
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

# Обзор — Platform Content Storage

## 1. Назначение

Platform Content Storage предоставляет платформенную capability для файловых
ресурсов. Она хранит бинарное содержимое, возвращает opaque `ContentRef` и
отдаёт descriptor или поток только после серверной проверки доступа.

## 2. Граница владения

- хранилище, таблицы, миграции и cleanup принадлежат `DMP.Platform.Content`;
- Configuration.ContentBlobs не используется;
- доменный модуль хранит только `ContentRef` и свои metadata;
- Content Storage не знает о Document Management, demo или конкретном типе
  бизнес-объекта;
- Document Management и Product Definition demo используют capability через
  публичные контракты и owner policy.

## 3. MVP

MVP поддерживает DB-backed provider, протокол `initiate -> upload -> finalize`,
opaque reference, descriptor-only object payload, tenant isolation, owner
authorization, idempotency, TTL и cleanup незакреплённых ресурсов.
Внешние providers, resumable upload, preview, OCR, antivirus и full-text
search находятся за пределами MVP.

## 4. Состав реализации

| Область | Реализация |
| --- | --- |
| Public contracts | `src/Platform/DMP.Platform.Contracts/Content` |
| Provider/API | `src/Platform/DMP.Platform.Content` |
| Runtime integration | typed `File` value, action parameter и lifecycle hook |
| Frontend | generic file picker, descriptor renderer и download client |
| Demo evidence | `NomenclatureAttachment_UploadFiles` и `NomenclatureAdditionalUnit.File` |

Связанные документы: [контракты](03_contracts.md), [runtime](04_runtime.md),
[безопасность](05_security_and_audit.md), [операции](08_operations.md) и
[трассировка](90_traceability.md).

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | Состав реализации | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
