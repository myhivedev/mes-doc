---
id: DOC-04-05-15
title: 'Меню навигации — Управление документацией'
type: design
status: approved
version: '1.3'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 05_document_management
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

# Меню навигации — Управление документацией

## 1. Назначение документа

Документ фиксирует пункты главного меню модуля «Управление документацией».

Меню открывает списки модуля. Состав полей, фильтров, карточек, вкладок и действий описан в [06_ui_views.md](06_ui_views.md).

## 2. Источник меню

Источник структуры меню — раздел «Меню навигации» ПР00. Ниже фиксируется
только целевая структура меню модуля 05.

## 3. Правила публикации меню

- меню публикуется как конфигурационный артефакт «Меню» (`Menu`) в общее главное меню среды выполнения (`RuntimeWeb_MainMenu`);
- пункты модуля публикуются как элементы навигации (`NavigationItem`);
- конечные пункты открывают представления из [06_ui_views.md](06_ui_views.md);
- карточки, окна выбора, вкладки и переключаемые варианты списка документов не получают самостоятельных пунктов;
- создание выполняется действиями соответствующих списков, отдельные пункты создания не публикуются;
- настройки предприятия редактируются в платформенной студии настроек (Studio Settings) и не получают пункт в меню модуля;
- пункт «Хранилища документов» не публикуется в MVP; пользовательское управление хранилищами относится к целевому расширению.

## 4. Структура меню

| Уровень | Код пункта | Русское название | Тип | Родитель | Целевое представление | Режим открытия | Порядок | Основание |
|---:|---|---|---|---|---|---|---:|---|
| 1 | `DocumentManagement` | Управление документацией | Группа | — | — | — | 500 | ПР00, общие функциональные требования |
| 2 | `DocumentManagement.Work` | Работа с документами | Группа | `DocumentManagement` | — | — | 10 | согласованное решение |
| 3 | `DocumentManagement.Work.Documents` | Документы | Пункт | `DocumentManagement.Work` | Список документов (`DocumentManagement.DocumentList`) | Раздел с тремя переключаемыми представлениями | 10 | ПР00 «Реестр документов», `06_ui_views.md` |
| 3 | `DocumentManagement.Work.Contents` | Содержимое документов | Пункт | `DocumentManagement.Work` | Содержимое документов (`DocumentManagement.DocumentContentList`) | Список | 20 | согласованное решение, `06_ui_views.md` |
| 2 | `DocumentManagement.Administration` | Администрирование | Группа | `DocumentManagement` | — | — | 20 | согласованное решение |
| 3 | `DocumentManagement.Administration.Folders` | Каталоги документов | Пункт | `DocumentManagement.Administration` | Список каталогов документов (`DocumentManagement.DocumentFolderList`) | Иерархический список | 10 | ПР00, `06_ui_views.md` |
| 3 | `DocumentManagement.Administration.Types` | Типы документов | Пункт | `DocumentManagement.Administration` | Список типов документов (`DocumentManagement.DocumentTypeList`) | Список | 20 | ПР00, `06_ui_views.md` |

## 5. Связь с правами

Пункт меню отображается только при наличии права просмотра целевого объекта. Предметные роли и права описаны в [11_permissions.md](11_permissions.md). Отдельные права на пункты меню не вводятся.

## 6. Навигационная граница

Platform Content Storage не добавляет самостоятельный пункт меню. Навигация
Document Management остаётся владельцем модуля; компонент выбора файла
появляется только в настроенных представлениях action и property.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.3 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | Связь с правами; Навигационная граница | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 1.2 | 2026-09-02 13:56 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: module; Источник меню; Представления без пункта меню; Поведение пункта «Документы»; Связь с правами | изменена струтура описания меню уже выпущенных модулей. содержание не изменено | [454ab8aa](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/454ab8aa3a684a27e9c7adfc61ced9407e1b77bb) |
| 1.1 | 2026-08-26 15:52 +03:00 | Воронкова Вероника (@VeronikaV2121) | Правила публикации меню; Представления без пункта меню; Связь с правами | docs: уточнить MVP-решения Document Management | [7db596eb](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/7db596eb0ba8f5a44ea61393ec5afd83ed8fd361) |
| 1.0 | 2026-08-21 12:16 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #31: 05 document management docs | [PR #31](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/31) |
| 0.1 | 2026-08-21 11:33 +03:00 | Донских Сергей (@Sergey020726) | Создание документа | docs: add document management module documentation | [ef648f68](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/ef648f6831ca38c686a44d67ae8abfe79815c9eb) |
