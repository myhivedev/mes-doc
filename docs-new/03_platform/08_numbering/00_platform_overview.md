---
id: DOC-03-08-00
title: 'Обзор платформенной области — Numbering'
type: design
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: numbering
holder: '@axelprosoft'
created_at: 2026-08-27 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Обзор платформенной области — Numbering

## 1. Назначение области

Numbering выдаёт последовательные значения для нумеруемых строковых полей
прикладных объектов. Область хранит runtime-состояние счётчика, атомарно
выделяет следующее значение и фиксирует факт выдачи.

Правило формирования номера хранится в Configuration как артефакт
`NumberingRule`. Numbering не владеет схемой этого артефакта.

## 2. Место в платформе

```text
Configuration
  опубликованный NumberingRule
          |
          v
Object Runtime -- создаёт объект и передаёт контекст
          |
          v
Numbering
  выбирает правило -> выделяет номер -> форматирует значение
          |
          v
Object Runtime записывает значение в поле объекта
```

Общая карта взаимодействий находится в
[интеграционной архитектуре](../../02_architecture/07_integration_architecture.md).

## 3. Основные возможности

| Возможность | Назначение | Статус | Основание |
| --- | --- | --- | --- |
| Выдача следующего номера | Выделить значение по правилу, счётчику и разделу | Подтверждено MVP | `INumberingService`, runtime и тесты |
| Форматирование | Подставить порядковый номер, дату и разрешённые значения объекта | Подтверждено MVP | `NumberingService` и тесты |
| Разделение последовательностей | Вести независимые счётчики по `PartitionKey` | Подтверждено MVP | `INumberingCounterStore` и тесты |
| Интеграция с созданием объекта | Заполнить пустое поле на `OnCreate` до основной проверки объекта | Подтверждено MVP | Object Runtime hook и тесты |
| Журнал выдачи | Сохранить сведения о выданном номере после успешного результата операции | Подтверждено MVP | `NumberingIssueLogEntry` и тесты |

## 4. Ключевые решения

| Решение | Суть | Документ-владелец |
| --- | --- | --- |
| Владелец правила | `NumberingRule` является артефактом Configuration | [спецификация NumberingRule](../02_configuration/artifact_types/numbering_rule.md) |
| Владелец счётчика | `NumberingCounter` и `NumberingIssueLog` принадлежат Numbering | `02_architecture.md` |
| Момент V1 | Автоматическая выдача выполняется только на `OnCreate` для пустого поля | `04_runtime.md` |
| Ключ счётчика | Уникальность счётчика определяется `RuleIdentity + ScopeId + PartitionKey` | `02_architecture.md`, `04_runtime.md` |

## 5. Зависимости

| Зависимость | Использование | Владелец | Документ |
| --- | --- | --- | --- |
| Configuration | Публикует `NumberingRule` и свойства `ObjectMember`, разрешающие нумерацию | Configuration | `../02_configuration/` |
| Object Runtime | Передаёт контекст создания, вызывает hook и сохраняет выданное значение | Object Runtime | `../03_object_runtime/` |
| Foundation | Предоставляет `ITenantContext`, `ICorrelationContext`, `IClock` и общие результаты | Foundation | `../00_foundation/` |
| Tenant/Security | Косвенно предоставляет tenant/site/user контекст и права исходной операции | Tenant/Security | `../01_tenant_and_security/` |
| Integration Events | Не является обязательным каналом выдачи номера в текущем контракте | Integration Events | `../../02_architecture/07_integration_architecture.md` |

## 6. Статус реализации

| Состояние | Значение | Основание |
| --- | --- | --- |
| Подтверждено MVP | Счётчик, атомарное выделение, форматирование, журнал и hook создания объекта реализованы | Код и интеграционные тесты |
| Подтверждено, но ограничено | Публикуемый resolver подключается host-приложением; базовая регистрация Numbering содержит пустой fallback resolver | Composition root и `MissingNumberingRuleResolver` |
| Будущая доработка | `OnSave`, ручной запрос, workflow-переходы, административное редактирование счётчиков и отдельный интерфейс Numbering не являются текущими гарантиями | `90_traceability.md` |

## 7. Состав документов

| Документ | Содержание |
| --- | --- |
| `00_platform_overview.md` | Роль, границы, зависимости и статус области |
| `01_scope.md` | Ответственность Numbering и границы владельцев |
| `02_architecture.md` | Компоненты, модель Numbering и хранение |
| `03_contracts.md` | Application-порты и контракты области |
| `04_runtime.md` | Выбор правила, выдача и интеграция с созданием объекта |
| `05_security_and_audit.md` | Права, контроли, журнал выдачи и пробелы |
| `07_quality.md` | Проверки, свидетельства и ограничения качества |
| `08_operations.md` | Запуск, миграции, диагностика и восстановление |
| `90_traceability.md` | Источники, решения, расхождения и маршрут будущих работ |

Отдельный `06_user_experience.md` не создаётся: самостоятельный frontend-контракт
Numbering не подтверждён. Редактор `NumberingRule` описывается Configuration,
а общая оболочка и маршрутизация интерфейса принадлежат фронтенд-платформа.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
