---
id: DOC-03-13-05
title: 'Безопасность и аудит — Platform Content Storage'
type: assurance
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

# Безопасность и аудит — Platform Content Storage

## 1. Границы доступа

Content Storage проверяет tenant, state и зарегистрированную
`IContentResourceAccessPolicy`. Знание `ContentRef` не даёт право чтения.
Доменный потребитель сам решает, может ли текущий actor читать или изменять
конкретного owner; Content Storage не встраивает права Document Management.

В demo policy использует стандартный Tenant Security и права Product Definition.
В модуле 05 policy должна использовать права Document Management и проверку
документа-владельца. Эти регистрации независимы.

## 2. Аудит

Инициирование, завершение, attach, detach, cancel и read проходят через
`IAuditHistoryWriter`. Запись содержит tenant, actor, operation, ref, имя,
размер и correlation context, но не бинарное содержимое, authorization headers, секреты или
физический путь. Доменный модуль дополнительно пишет свои бизнес-действия в
своём action namespace.

## 3. Защиты

- имя файла не может содержать сегменты пути или управляющие символы;
- расширение нормализуется с ведущей точкой и сравнивается без учёта регистра;
- MIME синтаксически проверяется сервером и не считается доверенным;
- размер проверяется до и после чтения потока;
- незавершённый ресурс не читается и не attach-ится.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
