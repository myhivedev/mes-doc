---
id: DOC-03-13-06
title: 'Пользовательский опыт — Platform Content Storage'
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

# Пользовательский опыт — Platform Content Storage

Content Storage не имеет самостоятельного экрана. Generic Frontend Runtime
встраивает file picker в action dialog, detail form и inline editor по metadata
настроенного runtime-контракта.

Пользователь видит имя, расширение, размер, состояние, ошибки и доступные
действия. Ошибки смешанного набора привязываются к конкретному файлу; поле в
режиме только для чтения показывает descriptor и скачивание без возможности
изменения. В disabled-состоянии выбор и очистка недоступны, а скачивание
сохранённого файла остаётся доступным. Физический путь, ключ провайдера и
бинарное содержимое в object payload не показываются.

Для action вложенной коллекции кнопка размещается в toolbar этой коллекции.
После подтверждения выбора action получает контекст владельца, создаёт
дочерние записи, а список обновляется через стандартное поведение
`RefreshHost`. Универсальный runtime не показывает в такой коллекции общий
`Добавить`, если дочерний объект объявлен как `AbstractReferenceOnly`; действия
над строкой (скачивание, назначение основным, отвязка) остаются отдельными
операциями.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | Пользовательский опыт — Platform Content Storage | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 1.0 | 2026-09-03 14:49 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #61: изменения в проектной документации ядра - новый модуль работы с файлами | [PR #61](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/61) |
| 0.1 | 2026-09-03 13:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | изменения в проектной документации ядра. основание - новый модуль по хранению и работе с файлами | [0f415f89](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/0f415f89b977fa123e52411717a66f94c2d327a3) |
