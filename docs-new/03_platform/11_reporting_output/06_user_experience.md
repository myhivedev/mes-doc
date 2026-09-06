---
id: DOC-03-11-06
title: 'Пользовательские сценарии — Reporting и Output'
type: design
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: reporting_output
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@66d9ecbb1cf30df64e04f137b301279ea86e83ed
---

# Пользовательские сценарии — Reporting и Output

## 1. Назначение документа

Документ описывает специфичные для отчётов UI-сценарии. Общая оболочка приложений,
маршрутизация, базовые пакеты рендеринга и общие тексты принадлежат
[Frontend Platform](../12_frontend_platform/00_platform_overview.md).

## 2. Frontend-контракты

| Сценарий | Frontend-контракт | Серверная граница | Владелец |
| --- | --- | --- | --- |
| Редактирование design отчёта | [reportDesign.ts](../../../src/Frontend/packages/contracts/src/reportDesign.ts) | Операции Configuration с design | Configuration + reporting UI |
| Панель design в Studio | [reportDesignPanel.tsx](../../../src/Frontend/apps/studio/src/features/configuration-artifact-editor/model/reportDesignPanel.tsx) | API Configuration | Studio |
| Запуск Output | `GenerateOutputRequest` | `POST /api/runtime/outputs/generate` | Runtime / Reporting Output |
| Визуализация результата | `GenerateOutputResponse` | Бинарное содержимое и метаданные | Приложение, вызывающее runtime |

## 3. Разделы интерфейса и сценарии

1. В Studio пользователь выбирает `Report` и открывает панель design.
2. Пользователь может запросить schema, sample XML или design по умолчанию.
3. Design можно импортировать, проверить и открыть в сеансе preview.
4. В runtime-интерфейсе доступный output появляется только при наличии effective launch
   binding и опубликованного `Output`.
5. Запуск передаёт контекст View, фильтры, текущий объект, язык и параметры.
6. Клиент получает файл или сообщение об ошибке с `requestId`.

## 4. Навигация

Reporting/Output не владеет общей навигацией. Код launch binding материализуется
из effective-конфигурации для View; его проекция и размещение остаются частью
связки runtime и frontend. Общая маршрутизация описывается Frontend Platform.

## 5. Локализация

Код языка передаётся в runtime-контекст и разрешается серверным resolver языка.
Названия отчёта, полей и параметров могут быть локализованы через определения
Configuration. Общий механизм хранения и выбора локализованных
значений принадлежит соответствующим владельцам, а не Reporting/Output.

## 6. Ограничения

- Отдельный универсальный редактор Reporting не считается существующим: подтверждена
  специфичная для отчётов панель Studio.
- Общая оболочка и рендеринг не дублируются здесь.
- Функциональность внешней доставки, расписаний и фоновых задач не считается
  доступной возможностью интерфейса без отдельного контракта.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
