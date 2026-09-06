---
id: DOC-04-05-12
title: 'Аудит и история — Управление документацией'
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

# Аудит и история — Управление документацией

## 1. События изменения

Регистрируются: создание и изменение документа; публикация; возврат в черновик; архивирование; восстановление; копирование; создание новой версии; добавление, прикрепление и отвязка содержимого; назначение основного содержимого; создание и удаление связи с записью бизнес-объекта; изменение типа, каталога, хранилища и настроек.

## 2. События чтения содержимого

Исходные требования (`NFR-DM-003`) требуют регистрировать скачивание файла и открытие внешней ссылки. Для события фиксируются: время, пользователь, предприятие, документ, содержимое, вид действия и результат без сохранения двоичных данных, текста или секрета провайдера.

События записываются модулем через существующий `IAuditHistoryWriter`. Используются action codes `DocumentManagement.Content.Download` и `DocumentManagement.Content.OpenUrl`; в детали входят идентификаторы документа и содержимого, вид действия и результат. Общий аудит отвечает за сохранение контекста пользователя, предприятия, времени и корреляции. Новая платформенная задача для MVP не требуется.

## 3. История версий

Версии документа хранятся отдельными записями и показываются во вкладке «Версии». История аудита не заменяет цепочку версий и не создаёт новые версии автоматически.

## 4. Защита данных

В аудит не записываются: двоичное содержимое, полный текст содержимого, секреты и параметры подключения хранилища. Сведения о недоступной связанной записи не раскрываются.

## 5. Аудит Content операций

Платформа фиксирует загрузку, attach, detach и скачивание; модуль дополнительно
фиксирует предметные операции replace и clear. Запись содержит ссылку на
документ и ContentRef, предприятие, пользователя, операцию, результат и
correlation id без бинарных данных и секретов. Аудит Platform Content и
бизнес-аудит модуля дополняют друг друга.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.2 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | Защита данных; Аудит Content операций | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 1.1 | 2026-08-26 15:52 +03:00 | Воронкова Вероника (@VeronikaV2121) | События чтения содержимого; Защита данных | docs: уточнить MVP-решения Document Management | [7db596eb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7db596eb0ba8f5a44ea61393ec5afd83ed8fd361) |
| 1.0 | 2026-08-21 12:16 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #31: 05 document management docs | [PR #31](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/31) |
| 0.1 | 2026-08-21 11:33 +03:00 | Донских Сергей (@Sergey020726) | Создание документа | docs: add document management module documentation | [ef648f68](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/ef648f6831ca38c686a44d67ae8abfe79815c9eb) |
