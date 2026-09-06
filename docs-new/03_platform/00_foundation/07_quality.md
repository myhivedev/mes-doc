---
id: DOC-03-00-07
title: 'Качество — Foundation'
type: assurance
status: in-review
version: '0.1'
owner: '@axelprosoft'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: platform
module: foundation
holder: '@axelprosoft'
created_at: 2026-08-26 00:00
created_by: '@codex'
updated_at: 2026-08-27 15:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: awaiting_reviewer
supersedes: []
source: authored
source_revision: origin/master@3e2037ac37683eef331d70223c6babfc61aa0539
---

# Качество — Foundation

[application-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/
[domain-project]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Domain/
[architecture-tests]: ../../../tests/DMP.Platform.ArchTests/Architecture/
[localization-tests]: ../../../tests/DMP.Platform.ArchTests/Localization/LocalizedTextResolverTests.cs
[runtime-tests]: ../../../tests/DMP.Platform.IntegrationTests/Runtime/
[common-boundary]: ../../../tests/DMP.Platform.ArchTests/Architecture/CommonModuleStructureArchTests.cs

## 1. Назначение документа

Документ фиксирует проверяемые свойства Foundation и границы доказательств.
Он не заменяет тестовую документацию отдельных platform areas и не объявляет
гарантами свойства, для которых нет реализации или test evidence.

## 2. Надёжность

| Отказ | Гарантия | Предел | Восстановление | Проверка |
| --- | --- | --- | --- | --- |
| Неизвестное поле сортировки/группировки | Запрос отклоняется до выполнения с reason code | Только для операций, использующих validators Foundation | Исправить запрос или allow-list потребителя | [ListOrdering][ordering] |
| Неизвестный reference source | Registry не выбирает произвольный resolver и возвращает пустой результат | Provider должен быть зарегистрирован | Зарегистрировать provider владельцем source | [Registry][registry] |
| Не подключён platform gateway | Foundation предоставляет no-op fallback, если его выбрал composition root | Production-допустимость не подтверждена | Подключить реализацию владельца capability и проверить composition | [Fallback][fallback] |
| Несовместимый локальный контекст | Общий `ModuleExecutionContext` фиксирует единый набор координат | Не препятствует произвольному private context, если он не пересекает общую границу | Убрать локальный public substitute или оформить решение | [Context][context] |

## 3. Наблюдаемость

Foundation передаёт `CorrelationId` через `ModuleExecutionContext` и допускает
стабильные коды проблем и полей. Метрики, трассировочные интервалы, журналы,
health checks и correlation policy промышленной среды принадлежат host или
общей Operations/Platform area, а не библиотеке Foundation.

## 4. Производительность

В текущем коде есть синхронные операции нормализации, сортировки и разрешения
локализованного значения, а registry reference display выполняет асинхронный
вызов provider. Foundation не задаёт SLO, размер кэша, лимит коллекции или
профиль нагрузки: эти свойства принадлежат потребителю и владельцу данных.

## 5. Проверки и результаты

| Сценарий проверки | Уровень | Источник теста | Результат запуска | Пробел |
| --- | --- | --- | --- | --- |
| Правила направления зависимостей Foundation и Common module | Архитектурный | [Common boundary tests][common-boundary] | Реализация тестов найдена; запуск в этой задаче не выполнялся | Нужен отчёт CI для актуальной ревизии |
| Fallback exact → neutral → invariant → technical code | Архитектурный/unit | [LocalizedTextResolverTests][localization-tests] | Реализация тестов найдена; запуск в этой задаче не выполнялся | Нужен отчёт CI |
| Интеграционные потребители результатов и gateway | Интеграционный | [Runtime tests][runtime-tests] | Покрытие потребителей найдено; запуск в этой задаче не выполнялся | Полный контракт production composition не подтверждён |

## 6. Неподтверждённые свойства

- Нет подтверждения промышленной допустимости `AllowAllPlatformRuntimeServicesGateway`.
- Нет общего SLO для helper-алгоритмов и gateway-вызовов.
- Нет собственной persistence-схемы Foundation, поэтому требования к хранению должны подтверждаться в документах потребителей.
- Нет отдельного test evidence для каждого потребляющего модуля; conformance проверяется пакетами владельцев.
[ordering]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Ordering/ListOrdering.cs
[registry]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Common/ReferenceDisplayResolverRegistry.cs
[fallback]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/AllowAllPlatformRuntimeServicesGateway.cs
[context]: ../../../src/BuildingBlocks/DMP.BuildingBlocks.Application/Abstractions/ModuleExecutionContext.cs

## История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 0.1 | 2026-08-27 15:04 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | подготовлен драфт новой проектной документации на ядро, прикладные модули, а так же термины и общие концептуальные документы | [5ecb26b1](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/5ecb26b1c3ff3cfcdc13018e59526ae684ca6007) |
