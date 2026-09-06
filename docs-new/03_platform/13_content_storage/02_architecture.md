---
id: DOC-03-13-02
title: 'Архитектура — Platform Content Storage'
type: architecture
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

# Архитектура — Platform Content Storage

## Компоненты

`DMP.Platform.Contracts.Content` содержит публичные records/interfaces.
`DMP.Platform.Content` содержит API, application service, domain resource,
EF Core persistence и hosted cleanup. Host composition регистрирует capability
одинаково для production API и Development demo API.

## Поток данных

```text
Frontend -> initiate/upload/finalize -> Content Store -> platform_content
Frontend -> standard action/property -> Object Runtime -> owner policy
Frontend -> authorized stream read -> Content Store -> binary response
```

Обычные object snapshots не содержат binary. Доменный модуль не получает
зависимость на `ContentDbContext` или таблицу provider.

## Архитектурные инварианты

- server-generated opaque ref;
- tenant boundary на каждом запросе;
- policy callback до attach/read;
- transaction boundary объекта и resource link не размывается предметными
  контроллерами;
- provider registry остаётся расширением после MVP.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
