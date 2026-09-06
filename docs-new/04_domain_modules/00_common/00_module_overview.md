---
id: DOC-04-00-00
title: 'Обзор модуля - 00 Common'
type: module-spec
status: approved
version: '1.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: domain
module: 00_common
holder: '@axelprosoft'
created_at: 2026-08-04 16:24
created_by: '@axelprosoft'
updated_at: 2026-09-02 11:50
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Обзор модуля - 00 Common

## 1. Назначение модуля

`00_common` - базовый прикладной модуль DMP для общих свойств и правил прикладных управляемых объектов.

Модуль вводит две целевые базовые модели:

```text
CommonObject
CommonCatalogObject
```

`CommonObject` используется для прикладных объектов без обязательных `Code` и `Name`.

`CommonCatalogObject` используется для справочников и каталоговых объектов, где `Code` и `Name` являются частью бизнес-идентификации.

## 2. Зачем нужен Common

Common нужен, чтобы прикладные модули одинаково описывали:

- идентичность объекта;
- внешний идентификатор;
- поля создания и изменения;
- архивирование;
- мягкое удаление;
- базовую справочную модель с `Code` и `Name`;
- подключение к Object Runtime.

Главная цель - предсказуемость. Если объект построен на базе Common, разработчик и пользователь должны понимать его базовое поведение без чтения каждого прикладного модуля заново.

## 3. Граница модуля

Common является прикладным модулем, а не всем платформенным ядром DMP.

Common не заменяет:

- `BuildingBlocks.Entity`;
- Object Runtime;
- Workflow;
- IAM;
- Audit History;
- Configuration;
- Integration Foundation / Integration Service.

Платформенные и системные сущности не обязаны наследоваться от `CommonObject`. Object Runtime может использоваться для них отдельно, без перехода на базовые типы Common.

## 4. Что входит в Common v1

В Common v1 входит:

- `CommonObject`;
- `CommonCatalogObject`;
- `ExternalId`;
- поля создания и изменения;
- архивные поля;
- поля мягкого удаления;
- стандартные коды `Archive` и `Restore`;
- правила видимости архивных и удаленных объектов;
- системная секция "Администрирование";
- общие настройки Common, владельцем смысла которых является модуль;
- системные enum общего смысла, которые используются этими настройками;
- завершенный переход прикладных каталогов на `CommonCatalogObject`;
- решение о границе между Common и платформенными модулями ядра.

## 5. Что не входит в Common v1

В Common v1 не входит:

- поле `Status` в базовом объекте;
- перенос `DmpStatusBase`;
- обязательный `SourceSystemCode` в `CommonObject`;
- полная модель внешних систем и интеграционных соответствий;
- полный workflow-шаблон жизненного цикла;
- универсальный `TenantId` в `CommonObject`;
- универсальный `SiteId` в `CommonObject`;
- автоматическое архивирование всех объектов;
- автоматическое разрешение удаления всех объектов;
- массовый перевод платформенных сущностей ядра на `CommonObject`.

## 6. Основные зависимости

Common опирается на:

- `DMP ПР00 Общие требования`;
- `DMP ПР01 Общая НСИ`;
- `DMP ПР09 Производственная логистика` - только для настройки `DurationFormat`, вынесенной из старых параметров предприятия в Common и переименованной в целевой модели в `TimeAmountFormat`;
- текущую реализацию `DMP.Modules.Common`;
- текущие паттерны Object Runtime;
- решения по Audit History, Workflow, IAM и Integration.

Полная ответственность этих платформенных механизмов остается в соответствующих модулях.

## 7. Карта документов

| Документ | За что отвечает |
|---|---|
| `02_domain_model.md` | Базовые типы, поля и доменная семантика Common. |
| `03_object_runtime_model.md` | Как модель Common публикуется и управляется через Object Runtime. |
| `05_rules.md` | Нормативные правила создания, изменения, архивирования, удаления и видимости объектов. |
| `16_settings.md` | Настройки, владельцем смысла которых является Common, и правила их использования прикладными модулями. |
| `90_traceability_pr00.md` | Сопоставление требований ПР00 с целевой моделью Common v1. |
| `20_implementation_backlog.md` | Приоритизированный бэклог реализации и миграции Common v1. |
| `_working/decision_log.md` | Рабочий журнал обсуждения. Не является финальным дизайн-документом. |
| `src/Modules/DMP.Modules.Common/README.md` | Практическое руководство по подключению Common к новому object type. |

## 8. Основные риски

| Риск | Решение |
|---|---|
| `CommonObject` начинают считать универсальной базой для всей DMP. | Common применяется к прикладным управляемым объектам. Платформенные сущности переходят на Common только по отдельному решению. |
| Архивирование и мягкое удаление начинают использовать как одно и то же. | Архив остается бизнес-фактом и виден по сохраненным ссылкам. Удаление исключает объект из обычной рабочей модели. |
| Старый `DmpStatusBase` возвращается как обязательный родитель каталогов. | `Status` не входит в `CommonObject` и `CommonCatalogObject`. Жизненный цикл проектируется отдельно. |
| `ExternalId` начинают использовать как полную модель интеграционного сопоставления. | `ExternalId` остается простым необязательным полем. Множественные внешние системы описывает интеграционный модуль. |

## 9. Состояние реализации

Common v1 реализован как reference implementation на `ProductDefinition.Nomenclature`. Система разворачивается на пустой БД сразу в целевой схеме; `BusinessEntity`, alias и перенос legacy-данных не поддерживаются.

Актуальные точки реализации:

- доменная модель, contracts и baseline — `src/Modules/DMP.Modules.Common`;
- централизованные query и mutation policies — `src/Platform/DMP.Platform.Runtime`;
- синхронизация архивного workflow и actor display — composition root `src/Hosts/DMP.Platform.Api`;
- пример persistence mapping, permissions и UI — `src/Modules/DMP.Modules.ProductDefinition`.

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.1 | 2026-09-02 11:50 +03:00 | Олег Юрьев (@axelprosoft) | Граница модуля; Карта документов | мелкие структурные правки в основном доменной модели и связей модулей без изменения содержания | [4cbd3069](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/4cbd3069ec4a3421adbad1f0c9801b00f28405d7) |
| 1.0 | 2026-08-19 17:22 +03:00 | Воронкова Вероника (@VeronikaV2121) | YAML-шапка: review_status, status | feat(docs): automate PR lifecycle approval | [3a2710e5](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3a2710e5d46a57a3074b89360c0f0b2a4e84cf52) |
| 0.1 | 2026-08-04 16:24 +03:00 | axelprosoft (@axelprosoft) | Содержание документа | Создание документа | [d8f70f25](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/d8f70f2534c7c4f54f1351ad43e53f96da6a63b8) |
