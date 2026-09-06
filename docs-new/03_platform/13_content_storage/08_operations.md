---
id: DOC-03-13-08
title: 'Эксплуатация — Platform Content Storage'
type: operation
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

# Эксплуатация — Platform Content Storage

## 1. Хранилище и миграция

Provider использует отдельный `ContentDbContext`, SQL schema
`platform_content`, platform-owned migration и таблицу ресурсов с binary и
metadata. Обычная миграция модуля не должна копировать или читать эту таблицу.
При отсутствии SQL connection в локальном demo используется отдельная
in-memory database name.

## 2. Ограничения MVP

| Настройка | Значение по умолчанию |
| --- | --- |
| Максимальный размер файла | 100 MB |
| TTL upload session | 30 минут |
| retention unattached resource | 24 часа |
| cleanup interval | 1 час, с operational minimum 1 минута |

Web host/request limits должны быть не меньше platform limit с учётом protocol
overhead. Потребитель может только сузить allowlist, размер и count.

## 3. Cleanup и recovery

Background cleanup повторно удаляет истёкшие sessions и unattached resources,
не затрагивая `Attached`. Ошибка одной итерации логируется без содержимого и не
останавливает host. Ручное восстановление: проверить correlation id и state,
повторить upload для `Created`, повторить attach для `Uploaded`; attached
resource не удалять вручную в обход owner policy.

## 4. Ограничения поставки

Пересборка persisted baseline snapshot не является частью Content migration.
Baseline demo импортируется штатным Configuration pipeline. Перед release
обязательна отдельная SQL Server/Windows Authentication проверка согласно
`AGENTS.md`; полный integration suite в development не запускается.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
