---
id: DOC-04-05-16
title: 'Настройки — Управление документацией'
type: design
status: approved
version: '1.2'
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

# Настройки — Управление документацией

## 1. Механизм

Настройки хранятся платформенным механизмом настроек предприятия. Их значения редактируются в платформенной студии настроек (Studio Settings) для предприятия. Отдельный объект «Настройки управления документацией» (`DocumentManagementSettings`), отдельная форма и пункт меню внутри модуля не создаются.

## 2. Настройки предприятия

| Пользовательское название | Ключ | Тип и начальное значение | Правило |
|---|---|---|---|
| Максимальный размер файла | `DocumentManagement.MaximumFileSizeBytes` | целое число; 100 МБ | Тип документа может только уменьшить значение. |
| Разрешённые расширения файлов | `DocumentManagement.AllowedFileExtensions` | утверждённый перечень из `05_rules.md` | Тип документа может только сузить перечень. |

## 3. Защищённая конфигурация

Регистрация провайдеров хранения, адреса подключения и секреты находятся в защищённой платформенной конфигурации. Они не хранятся в настройках предприятия и не возвращаются пользователю.

## 4. Отложенные возможности

Пользовательские хранилища, внешние провайдеры, их защищённая конфигурация и расширенный жизненный цикл относятся к целевому расширению после MVP (`DM-Q-024`, `DM-Q-025`). В MVP системная логическая запись хранилища «База данных» создаётся начальными данными модуля и не редактируется пользователем; физическое хранилище создаётся и обслуживается платформой.

## 5. Настройки Content Storage

Настройки модуля задают только более строгие `MaximumFileSizeBytes`,
`AllowedFileExtensions` и количество файлов относительно ограничений платформы.
Provider, секреты, физическое размещение и политика очистки в настройках модуля
не публикуются. `DM-Q-008` остаётся отложенным.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.2 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | Отложенные возможности; Настройки Content Storage | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 1.1 | 2026-08-26 15:52 +03:00 | Воронкова Вероника (@VeronikaV2121) | Настройки предприятия; Открытое ограничение; Отложенные возможности | docs: уточнить MVP-решения Document Management | [7db596eb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7db596eb0ba8f5a44ea61393ec5afd83ed8fd361) |
| 1.0 | 2026-08-21 12:16 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #31: 05 document management docs | [PR #31](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/31) |
| 0.1 | 2026-08-21 11:33 +03:00 | Донских Сергей (@Sergey020726) | Создание документа | docs: add document management module documentation | [ef648f68](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/ef648f6831ca38c686a44d67ae8abfe79815c9eb) |
