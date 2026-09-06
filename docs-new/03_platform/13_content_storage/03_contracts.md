---
id: DOC-03-13-03
title: 'Контракты — Platform Content Storage'
type: contract
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

# Контракты — Platform Content Storage

## 1. Публичная модель

`ContentRef` — строковое представление идентификатора, сгенерированного
сервером. Он не является путём, URL, именем таблицы или ключом провайдера.
`ContentResourceDescriptor` содержит ref, безопасное имя, extension, MIME,
размер, hash, время создания, state и optional owner. Бинарное содержимое в
descriptor отсутствует.

`ContentResourcePolicy` задаёт список разрешённых расширений, максимальный размер и
максимальное количество файлов. Политика запроса может только сузить системную
политику.

## 2. Жизненный цикл

| State | Значение |
| --- | --- |
| `Created` | создана сессия загрузки, поток ещё не принят |
| `Uploaded` | поток принят и проверен, ресурс ожидает владельца |
| `Attached` | ресурс связан с доменным объектом |
| `Expired` | сессия или ресурс больше не может использоваться |
| `Deleted` | ресурс удалён процессом очистки |

## 3. Операции

`IContentStore` предоставляет initiate, upload, finalize, cancel, attach,
detach, descriptor/read и cleanup. Policy владельца получает предприятие,
пользователя, `ContentRef`, ссылку на владельца и operation (`Read`, `Attach`,
`Detach`). Замена и очистка файлового поля в MVP выражаются операциями attach и
detach; значения `Replace` и `Clear` зарезервированы для расширения контракта.

Ошибки используют стабильные коды `CONTENT_*`: недопустимое имя, расширение,
MIME или размер, истёкшая сессия, незавершённая загрузка, запрет, отсутствие
ресурса и повторное прикрепление.

## 4. Инварианты

- предприятие и пользователь берутся из доверенного контекста;
- attach допускается только для финализированного ресурса в состоянии `Uploaded`;
- read допускается только для ресурса в состоянии `Attached` и разрешённого владельца;
- списки и карточки объектов возвращают descriptor, а не бинарное содержимое;
- физический путь и ключ провайдера никогда не попадают в публичный ответ;
- повторная операция с тем же idempotency key не создаёт новую сессию.

Исходный код: `src/Platform/DMP.Platform.Contracts/Content/`.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
