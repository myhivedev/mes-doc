---
id: DOC-03-08-90
title: 'Трассировка и открытые решения — Numbering'
type: traceability
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

# Трассировка и открытые решения — Numbering

## 1. Назначение документа

Документ показывает, какие сведения о Numbering подтверждены текущим кодом и
тестами, какие решения приняты, а какие требуют будущей работы.

## 2. Источники и требования

| Источник | Что использовано | Ревизия или состояние |
| --- | --- | --- |
| `src/Platform/DMP.Platform.Numbering` | Application ports, runtime, entities, persistence и DI | `origin/master@3e2037ac37683eef331d70223c6babfc61aa0539` |
| `src/Hosts/DMP.Platform.Api/Composition` | Подключение resolver опубликованной Configuration и baseline | Та же ревизия |
| `src/Platform/DMP.Platform.Configuration` | Schema и publish validation `NumberingRule` | Та же ревизия |
| `tests/DMP.Platform.IntegrationTests/Numbering` | Выдача, pipeline, условия, разделы и ошибки | Та же ревизия |
| `tests/DMP.Platform.IntegrationTests/Configuration` | Schema и publish validation правила | Та же ревизия |
| Переходная папка `11_numbering` | Предыдущая структура и проектные решения | Содержание распределено ниже; копия удалена после проверки ссылок |

### 2.1. Маршрут файлов переходной папки

| Файл | Куда перенесено содержание | Что не переносится как текущая гарантия |
| --- | --- | --- |
| `01_overview.md` | `00_platform_overview.md`, `01_scope.md`, `04_runtime.md` | Целевые и не подтверждённые моменты выдачи остаются в разделе 4 |
| `02_design_decisions.md` | `00_platform_overview.md`, `02_architecture.md`, `03_contracts.md`, `04_runtime.md`, `05_security_and_audit.md` | Нереализованные расширения остаются будущими задачами |
| `03_model.md` | `02_architecture.md`, `03_contracts.md` | Внутренние проектные сущности, не подтверждённые текущим кодом, не объявляются контрактом |
| `04_runtime.md` | `03_contracts.md`, `04_runtime.md`, `07_quality.md`, `08_operations.md` | `OnSave` и другие моменты, отсутствующие в текущем hook, не объявляются поддержанными |
| `05_object_runtime_integration.md` | `01_scope.md`, `03_contracts.md`, `04_runtime.md` | Mutation pipeline принадлежит Object Runtime; Numbering описывает только свою часть интеграции |
| `06_security.md` | `05_security_and_audit.md`, `08_operations.md` | Права корректировки счётчиков и отдельный пользовательский интерфейс не подтверждены |

Эта таблица является самостоятельным маршрутом миграции. После удаления копии
исходной папки её содержимое не требуется открывать для понимания текущего
пакета Numbering.

## 3. Принятые решения

| Решение | Состояние | Где отражено |
| --- | --- | --- |
| `NumberingRule` принадлежит Configuration | Принято | `01_scope.md`, `02_architecture.md` |
| Счётчик не является конфигурационным артефактом | Принято | `02_architecture.md`, `04_runtime.md` |
| В V1 автоматическая выдача выполняется на `OnCreate` для пустого поля | Принято для текущего MVP | `04_runtime.md` |
| Ключ счётчика включает rule identity, scope и partition | Принято | `02_architecture.md` |
| Выданный номер не возвращается после ошибки сохранения | Принято текущим runtime | `04_runtime.md` |

## 4. Расхождения и открытые решения

| Тема | Подтверждённое состояние | Влияние на текущую документацию | Следующий шаг | Владелец | Статус |
| --- | --- | --- | --- | --- | --- |
| Resolver правил | Host API регистрирует published Configuration resolver; базовый пакет Numbering имеет пустой fallback | Реальная выдача зависит от host composition | Сохранить проверку host registration | Host + Configuration | Не блокирует описание MVP |
| Моменты выдачи | Код и активный каталог подтверждают `OnCreate`; `OnSave` объявлен, но hook для него не подтверждён | Документ не выдаёт `OnSave` за текущую гарантию | Решить полный набор моментов и изменить все владельцы согласованно | Numbering + Object Runtime + Configuration | Будущая доработка |
| Семантика `InheritanceMode` | Правило хранится в Configuration; полная runtime-семантика не принадлежит Numbering | Numbering описывает только получение effective candidate | Уточнять в Configuration | Configuration | Не блокирует |
| Конкурентность production provider-а | Store использует unique index и row version; отдельного provider-specific теста не найдено | Нельзя утверждать полную нагрузочную гарантию | Добавить отдельное испытание для каждого поддерживаемого provider-а | Numbering + Platform Operations | Будущая доработка |
| Администрирование счётчиков | Runtime-хранилище есть; интерфейс и ручная корректировка не подтверждены | Не описывать операции редактирования счётчика | При необходимости создать отдельный административный контракт | Numbering + Tenant/Security | Будущая доработка |
| Общий аудит и события | Есть `NumberingIssueLog`; собственного integration event не найдено | Не заявлять outbox, replay или доставку события | Решить, нужен ли межмодульный контракт аудита | Numbering + Audit History + Integration Events | Будущая доработка |

## 5. Маршрут в целевые документы

| Тема | Документ-владелец |
| --- | --- |
| Schema и свойства `NumberingRule` | `../02_configuration/artifact_types/numbering_rule.md` |
| Effective configuration и publish | `../02_configuration/04_runtime.md` |
| Mutation pipeline | `../03_object_runtime/04_runtime.md` |
| Общая карта взаимодействий | `../../02_architecture/07_integration_architecture.md` |
| Общие application-контракты | `../00_foundation/03_contracts.md` |

Содержание переходных материалов, которое не является текущим контрактом,
отмечено как будущая доработка или граница другого владельца; оно не переносится
в Numbering как реализованная гарантия.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
