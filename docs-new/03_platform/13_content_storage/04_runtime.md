---
id: DOC-03-13-04
title: 'Исполнение — Platform Content Storage'
type: runtime
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

# Исполнение — Platform Content Storage

## 1. Поток загрузки

1. Runtime frontend вызывает initiate с именем, размером, MIME и effective
   policy.
2. Content API создаёт короткоживущую сессию.
3. Клиент передаёт бинарное содержимое отдельным stream request.
4. Finalize проверяет состояние и возвращает descriptor со state `Uploaded`.
5. Стандартный action или mutation property связывает ресурс с owner.

Прямой multipart upload в доменное action и передача физического пути не
поддерживаются.

## 2. Файловый параметр action

Файловый параметр материализуется как `DataType=File`,
`ComponentCode=FilePicker`, optional `Multiple` и file policy. В action
передаются `ContentRef` и метаданные, а не bytes. Для нескольких файлов результат содержит общий итог и
`ContentActionItemResult` по каждому ref.

## 3. Файловое свойство

Свойство `File` — nullable single-value member. В create/edit mutation принимается
только финализированный ref; очистка передаётся как `null`. Runtime hook валидирует ref
до записи, а attach/detach выполняются в общей mutation boundary. Старый ref не
теряется при неуспешной замене.

Обычные ответы list/details/collection используют descriptor projection.
Бинарное содержимое читается через отдельную защищённую конечную точку stream.

## 4. Встроенная коллекция

Для новой строки дочерней коллекции ref может быть загружен до появления
сохранённого идентификатора владельца. Mutation сохраняет ref вместе со строкой;
отмена строки оставляет ресурс unattached, после чего его обрабатывает retention cleanup.

Runtime integration реализована в `DMP.Platform.Runtime` через общий mutation
validator/lifecycle hook и не добавляет предметные ветви для Document Management.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
