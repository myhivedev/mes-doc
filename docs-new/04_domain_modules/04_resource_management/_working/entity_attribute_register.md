# Рабочий реестр объектов и областей ПР04

Рабочий реестр для подготовки проектной документации модуля `04_resource_management`. Источники сверки: ПР04, решения в `decision_log.md`, `DMP_DATA`, Common, GMD, платформа и исходный код. Реестр не является целевой доменной моделью.

| Объект / область | Роль в ПР04 | Источник ПР04 / `DMP_DATA` | Целевое решение v1 | Целевой документ | Статус |
|---|---|---|---|---|---|
| `WorkPlace` | Рабочее место | ПР04, `WorkPlace` | Concrete object type, владелец ПР04 | `02_domain_model.md` | Accepted |
| `EquipmentUnit` | Внешняя единица оборудования | ПР04, внешняя ссылка | Внешний объект; автоматическое создание `WorkPlace` не входит в v1 | `01_scope.md`, `02_domain_model.md` | External |
| `WorkPlaceGroup` | Группа рабочих мест | ПР04, `WorkPlaceGroup` | Concrete object type, собственная группа | `02_domain_model.md` | Accepted |
| `WorkPlaceInWorkPlaceGroup` | Членство рабочего места в группе | ПР04, `WorkPlaceInWorkPlaceGroup` | Отдельный dependent object type | `02_domain_model.md` | Accepted |
| `Personnel` | Сотрудник | ПР04, `Personnel` | Concrete object type, владелец ПР04 | `02_domain_model.md` | Accepted |
| `PersonnelGroup` | Группа сотрудников | ПР04, `PersonnelGroup` | Concrete object type, собственная группа | `02_domain_model.md` | Accepted |
| `PersonnelInPersonnelGroup` | Членство сотрудника в группе | ПР04, `PersonnelInPersonnelGroup` | Отдельный dependent object type | `02_domain_model.md` | Accepted |
| `Profession` | Профессия | ПР04, `Profession` | Справочный объект ПР04 | `02_domain_model.md` | Accepted |
| `PaymentGroup` | Группа оплаты | ПР04, `PaymentGroup` | Справочный объект ПР04; используется ПР02 как внешняя ссылка | `02_domain_model.md` | Accepted |
| `PersonnelQualification` | Квалификация сотрудника | ПР04, `PersonnelQualification` | Факт квалификации сотрудника | `02_domain_model.md` | Accepted |
| `PersonnelAllowanceType` | Вид допуска | ПР04, `PersonnelAllowanceType` | Справочный объект ПР04 | `02_domain_model.md` | Accepted |
| `PersonnelAllowance` | Допуск сотрудника к рабочему месту | ПР04, `PersonnelAllowance` | Факт допуска с периодом действия | `02_domain_model.md` | Accepted |
| `ToolBase` | Логический базовый тип оснастки | ПР04, логическая модель | Abstract/reference-only тип; отдельный пользовательский объект не создается | `02_domain_model.md`, `03_object_runtime_model.md` | Accepted |
| `Tooling` | Технологическая оснастка | ПР04, `Tooling` | Concrete type в общем физическом хранилище `ToolBase` | `02_domain_model.md` | Accepted |
| `Gage` | Контрольно-измерительный инструмент | ПР04, логическая модель | Concrete type в общем физическом хранилище `ToolBase` | `02_domain_model.md` | Accepted |
| `WorkPlaceLocation` | Место установки рабочего места | ПР04, `WorkPlaceLocation` | Concrete dependent object type | `02_domain_model.md` | Accepted |
| `PersonnelLocation` | Место работы сотрудника | ПР04, `PersonnelLocation` | Concrete dependent object type | `02_domain_model.md` | Accepted |
| `ResourceLocationBase` | Общая логика местоположения | ПР04, `ResourceLocationBase` | Только логический базовый тип, без parent table | `02_domain_model.md` | Accepted |
| `WorkSchedule` | График работы | ПР04, `WorkSchedule` | Самостоятельный объект графика | `02_domain_model.md` | Accepted |
| `DayType` | Тип рабочего дня | ПР04, `DayType` | SystemEnum / классификатор по результату сверки | `02_domain_model.md` | Needs enum check |
| `ShiftRotationModel` | Модель чередования смен | ПР04, `ShiftRotationModel` | Самостоятельный объект графика/сменности | `02_domain_model.md` | Accepted |
| `ResourceWorkSchedule` | Логический базовый тип назначения графика | ПР04, `ResourceWorkSchedule` | Только логический базовый тип | `02_domain_model.md` | Accepted |
| `*WorkSchedule` assignments | Назначения графиков | ПР04, concrete assignment tables | Собственные concrete типы по владельцу | `02_domain_model.md` | Accepted |

## Правило использования реестра

Каждое поле целевой доменной модели должно быть дополнительно сверено с обязательностью, типом и ограничениями исходного ПР и отражено в целевом документе или трассировке. Статус `Accepted` означает принятое рабочее решение, а не завершение переноса в документацию.
