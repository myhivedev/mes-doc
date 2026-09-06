---
id: DOC-12-99-01
title: 'Быстрый старт разработчика прикладного модуля'
type: appendix
status: approved
version: '1.2'
owner: '@axelprosoft'
reviewers: []
scope: appendices
module: application_modules
holder: '@axelprosoft'
created_at: 2026-09-03 10:28
created_by: '@axelprosoft'
updated_at: 2026-09-03 16:04
last_modified_by: '@axelprosoft'
last_reviewed: null
review_status: 'approved'
supersedes: []
source: authored
---

# Быстрый старт разработчика прикладного модуля

## 1. Назначение

Этот документ показывает, как разработчику создать первый простой прикладной
модуль DMP и проверить, что он подключается к платформе через Object Runtime.

Инструкция рассчитана на серверного разработчика, который уже умеет собирать
решение, но ещё не писал свои прикладные модули для DMP. Каждый шаг содержит:

- готовые команды;
- минимальные фрагменты кода;
- краткое объяснение;
- проверку результата;
- ссылки на существующий демо-модуль `ProductDefinition`.

В качестве рабочего примера используется учебный модуль
`DMP.DemoModules.WorkshopCatalog` с одним объектом `Workshop`.

Этот модуль является именно `Demo`: он расположен в `src/Demos`, имеет
пространство имён `DMP.DemoModules.*` и нужен для обучения и проверки
платформенных возможностей. Поэтому в шагах подключения и запуска он
добавляется в `DMP.Platform.DemoApi`. Реальный прикладной модуль размещается в
`src/Modules`, использует пространство имён `DMP.Modules.*` и подключается к
production host `DMP.Platform.Api`. Ниже для обоих вариантов приведены
отдельные инструкции.

Термин production host обозначает состав приложения, предназначенный для
развёртывания, а не текущее окружение ASP.NET Core: локально
`DMP.Platform.Api` также можно запускать с профилем `Development`.

Важно: названия DSL-узлов, свойств, значений артефактов и API-термины не
переводятся. В тексте они пишутся так же, как в коде или `baseline`:
`list`, `detail`, `lookup`, `layout`, `View`, `Action`, `ObjectTypeCode`,
`fields`, `baseline`.

## 2. Что получится в конце

После выполнения шагов у вас будет модуль со следующей минимальной цепочкой:

```text
Workshop (доменная сущность)
  -> WorkshopObjectContract
  -> WorkshopModuleRegistration
  -> WorkshopBaselineConfiguration
  -> AddWorkshopCatalogModule(...)
  -> ObjectRuntimeDescriptor
  -> DemoApi (для учебного примера) / Runtime API
```

Это тот же путь, который использует эталонный демо-модуль:

- модуль: `src/Demos/DMP.DemoModules.ProductDefinition`;
- объект: `Objects/Nomenclature`;
- регистрация: `Configuration/Registration/DemoProductDefinitionModuleRegistration.cs`;
- baseline: `Configuration/Baseline/DemoProductDefinitionBaselineConfiguration.cs`;
- DI: `DemoProductDefinitionDependencyInjection.cs`;
- запуск: `src/Hosts/DMP.Platform.DemoApi`.

## 3. Перед началом

Выполните команды из корня репозитория:

```powershell
cd C:\TFS2026\dmp
dotnet --info
dotnet restore DMP.PlatformFoundation.sln
dotnet build src\Demos\DMP.DemoModules.ProductDefinition\DMP.DemoModules.ProductDefinition.csproj --no-restore
```

Что происходит:

- проверяется локальный .NET SDK;
- восстанавливаются зависимости решения;
- собирается существующий демо-модуль, чтобы убедиться, что эталонная точка
  входа работает.

Если сборка демо-модуля не проходит, сначала чините окружение или текущую
ветку. Новый модуль лучше не начинать поверх сломанной эталонной точки.

Справочные документы:

- [Обзор Object Runtime](../03_platform/03_object_runtime/00_platform_overview.md);
- [Контракты Object Runtime](../03_platform/03_object_runtime/03_contracts.md);
- [Configuration: ObjectType](../03_platform/02_configuration/artifact_types/object_type.md);
- старый большой дизайн-документ: [dmp-module-object-runtime.md](../../docs/other_tech/dmp-module-object-runtime.md).

## 4. Граница ответственности разработчика модуля

Разработчик прикладного модуля реализует предметную модель, `object contracts`,
`baseline`, `permissions`, `validators`, `lifecycle handlers`, `command handlers`,
миграции и тесты своего модуля. Он не расширяет ядро платформы и общий рантайм
фронтенда в рамках задачи модуля без отдельной платформенной задачи.

Это правило важнее скорости локального обхода. Если текущих возможностей
Object Runtime, Configuration, Workflow, Rules, Content, Reporting, Tenant
Security или Frontend Platform не хватает, разработчик оформляет требование к
доработке платформы. До появления платформенной возможности модуль не должен
закрывать сценарий жёстко заданным кодом.

### 4.1. Что нельзя делать в задаче прикладного модуля

| Нельзя | Почему | Как правильно |
|---|---|---|
| Менять `src/Platform/*` только ради одного модуля | Это размывает границу владения и создаёт скрытый платформенный контракт | Оформить платформенную задачу и дождаться утверждённого публичного контракта |
| Добавлять условия под конкретный модуль в Object Runtime | Runtime должен исполнять универсальный объектный конвейер, а не знать предметные типы | Описать универсальную точку расширения или платформенную возможность |
| Жёстко привязывать `ObjectTypeCode`, `ViewCode`, `fields` или `actions` в общем рантайме фронтенда | Следующий модуль не сможет использовать такой сценарий через `baseline` | Расширить `runtime contracts` декларативно и покрыть `demo`/`conformance` тестом |
| Делать отдельный React-компонент под доменный объект вместо `runtime view` | UI перестаёт быть конфигурируемым и ломает единый конвейер фронтенда | Описать недостающий `editor`/`view`/`action contract` для Frontend Platform |
| Делать отдельный controller для стандартного CRUD/action-сценария | Появляется параллельный API в обход Object Runtime, `permissions`, `audit` и `events` | Использовать Runtime API или запросить расширение Object Runtime |
| Хранить платформенные данные в таблицах модуля | Модуль начинает владеть чужой lifecycle/security/persistence моделью | Запросить платформенную возможность с публичными контрактами |
| Читать или менять таблицы другого модуля напрямую | Нарушаются границы модулей, tenant/security и будущая сервисная декомпозиция | Использовать `reference`, `lookup`, публичный контракт или `projection` |
| Кодировать недостающий тип данных как `string`/`Json` без контракта | Потребители не знают семантику, UI и валидация становятся неявными | Добавить требование к `schema`/`runtime`/`frontend` contract |

### 4.2. Когда нужна заявка на доработку платформы

Заявка нужна, если модулю требуется хотя бы одно из следующего:

- новый тип поля или редактора, например `File`, `RichText`, `GeoPoint`;
- новый способ отображения списка, карточки, `inline`-коллекции или `action input`;
- новый протокол/API, например загрузка, скачивание, предпросмотр или длительная
  операция;
- новый вид проверки публикации `baseline` или проверки схемы;
- новая интеграция с workflow, rules, audit или events;
- новая граница безопасности, проверка прав или хранение с учётом tenant;
- изменение универсального поведения Object Runtime, которое должно работать для всех
  модулей, а не только для текущего.

Пример результата правильного процесса уже есть в бэклоге по работе с содержимым:
[Платформенная возможность для Document Management: содержимое, загрузка и UI рантайма](../../plans/033_document_management_content_capability_backlog.md).
Это не шаблон заявки разработчика, а уже проработанный архитектором ядра
бэклог реализации, возникший после выявленного разрыва между требованиями
прикладного модуля и возможностями платформы. В нём сценарий файлов не решается
отдельным `upload`-controller и доменной жёсткой привязкой во фронтенде; вместо
этого формируется платформенная возможность: `ContentRef`, протокол загрузки,
`authorization`, `Configuration`/`Baseline` schema, Object Runtime projection и
компоненты Frontend Runtime.

### 4.3. Минимальная форма заявки

Создайте документ бэклога или задачу с такой структурой:

```markdown
# Заявка на платформенную возможность: <короткое название>

## Статус и назначение

- Статус: черновик бэклога.
- Область: <область платформы>, <Frontend Platform>, <затронутый модуль>.
- Основание: <ссылка на документ модуля или требование>.
- Цель: <какой универсальный сценарий должен стать доступен без жёсткой
  привязки к одному модулю>.

## Разрыв между модулем и платформой

- Что модуль должен уметь.
- Что уже поддерживают Object Runtime / Configuration / Frontend Platform.
- Чего не хватает как `public contract`.

## Что не делать

- Какие локальные обходы запрещены.
- Какие решения под один модуль нельзя принимать за платформенную возможность.

## Предлагаемая платформенная возможность

- Новые или изменённые контракты.
- Изменения `baseline`/`schema`.
- Поведение Runtime.
- Поведение фронтенда.
- Безопасность, tenant-модель, аудит и эксплуатация.

## Критерии приёмки

- Тесты контрактов.
- Интеграционные тесты.
- Браузерные и компонентные тесты, если затронут фронтенд.
- Демонстрационный сценарий соответствия.
- Обновление документации.
```

Проверка заявки: из текста должно быть понятно, как новую возможность сможет
использовать второй прикладной модуль без копирования кода первого.

### 4.4. Пример черновика заявки от разработчика

Ниже пример входной заявки от разработчика прикладного модуля. Это не
архитектурное решение и не готовый бэклог ядра. Архитектор платформы может
переработать такой черновик в платформенную задачу, как это сделано в
[бэклоге по работе с содержимым Document Management](../../plans/033_document_management_content_capability_backlog.md).

Разработчик может использовать ИИ для подготовки черновика, но обязан сам
проверить факты: ссылку на требование модуля, текущее поведение платформы,
границы модулей и запрещённые локальные обходы.

```markdown
# Заявка на платформенную возможность: файлы у документа

## Статус и назначение

- Статус: черновик бэклога.
- Область: Object Runtime, Frontend Platform, Document Management.
- Основание: в модуле Document Management документ должен хранить вложенные файлы.
- Цель: дать прикладному модулю возможность объявить поле или коллекцию файлов
  через платформенный контракт, без отдельного controller-а загрузки и без
  React-компонента под один `ObjectTypeCode`.

## Разрыв между модулем и платформой

- Модуль должен уметь прикреплять файл к документу, показывать его в карточке,
  скачивать файл и учитывать права доступа.
- Сейчас обычные поля и представления объекта описываются через Object Runtime
  и `baseline`.
- Непонятно, каким публичным контрактом объявить файловое содержимое, как
  фронтенд должен отображать загрузку и скачивание, и где должны проверяться
  права.

## Что не делать

- Не добавлять `upload`-controller внутри Document Management в обход Runtime API.
- Не добавлять жёсткую привязку к `Document` или конкретному `ObjectTypeCode`
  в общем рантайме фронтенда.
- Не хранить платформенные метаданные файла в таблицах модуля без публичного
  контракта платформы.

## Ожидаемое поведение платформы

- Модуль декларативно объявляет, что объект поддерживает файловое содержимое.
- Runtime API возвращает фронтенду достаточно данных для отображения, загрузки
  и скачивания.
- Проверка прав, `tenant` и аудит выполняются единообразно для всех модулей.

## Критерии приёмки

- Возможность можно использовать минимум в двух прикладных модулях.
- Есть демонстрационный сценарий в демо-модуле.
- Есть тесты контрактов Runtime API.
- Есть браузерная или компонентная проверка фронтенда, если меняется UI.
- Документация описывает, как прикладной модуль подключает эту возможность.
```

## 5. Шаг 1. Создать проект модуля

Команды:

```powershell
cd C:\TFS2026\dmp
dotnet new classlib `
  --framework net9.0 `
  --name DMP.DemoModules.WorkshopCatalog `
  --output src\Demos\DMP.DemoModules.WorkshopCatalog

dotnet add src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj reference `
  src\BuildingBlocks\DMP.BuildingBlocks.Domain\DMP.BuildingBlocks.Domain.csproj `
  src\BuildingBlocks\DMP.BuildingBlocks.Application\DMP.BuildingBlocks.Application.csproj `
  src\Platform\DMP.Platform.Contracts\DMP.Platform.Contracts.csproj `
  src\Platform\DMP.Platform.Runtime\DMP.Platform.Runtime.csproj

dotnet add src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj package Microsoft.EntityFrameworkCore --version 9.0.9
dotnet add src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj package Microsoft.EntityFrameworkCore.SqlServer --version 9.0.9
dotnet add src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj package Microsoft.EntityFrameworkCore.InMemory --version 9.0.9

dotnet sln DMP.PlatformFoundation.sln add src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj
```

Что происходит:

- создаётся отдельная библиотека классов для модуля;
- подключаются building blocks, контракты платформы и Object Runtime;
- проект добавляется в решение.

Проверка:

```powershell
dotnet build src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj --no-restore
```

Эталон: `src/Demos/DMP.DemoModules.ProductDefinition/DMP.DemoModules.ProductDefinition.csproj`.

## 6. Шаг 2. Завести каталог стабильных кодов

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/WorkshopCatalogCodes.cs`:

```csharp
namespace DMP.DemoModules.WorkshopCatalog;

public static class WorkshopCatalogCodes
{
    public const string ModuleCode = "Demo.WorkshopCatalog";

    public static class Workshop
    {
        public const string ObjectTypeCode = "Workshop";
        public const string ListDatasetCode = "Workshop_ListDataset";
        public const string LookupDatasetCode = "Workshop_LookupDataset";
        public const string ListViewCode = "Workshop_ListView";
        public const string DetailViewCode = "Workshop_DetailView";
        public const string LookupViewCode = "Workshop_LookupView";

        public static class Fields
        {
            public const string Id = "Id";
            public const string TenantId = "TenantId";
            public const string ConcurrencyToken = "ConcurrencyToken";
            public const string Code = "Code";
            public const string Name = "Name";
            public const string Description = "Description";
            public const string IsActive = "IsActive";
        }

        public static class Permissions
        {
            public const string View = "Demo.WorkshopCatalog.Workshop.View";
            public const string Create = "Demo.WorkshopCatalog.Workshop.Create";
            public const string Edit = "Demo.WorkshopCatalog.Workshop.Edit";
            public const string Delete = "Demo.WorkshopCatalog.Workshop.Delete";
        }
    }
}
```

Что происходит:

- все публичные коды находятся в одном типизированном каталоге;
- в коде модуля не появляются строковые литералы вида `"Workshop"` или
  `"Demo.WorkshopCatalog.Workshop.View"` в разных местах;
- baseline, Object Runtime, права, тесты и API используют один источник.

Проверка:

```powershell
rg "Demo.WorkshopCatalog|Workshop_ListDataset|Workshop_DetailView" src\Demos\DMP.DemoModules.WorkshopCatalog
```

Эталон: `DemoProductDefinitionCodes.cs`.

## 7. Шаг 3. Создать доменную сущность

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/Objects/Workshop/Domain/Workshop.cs`:

```csharp
using DMP.BuildingBlocks.Domain;

namespace DMP.DemoModules.WorkshopCatalog.Domain.Entities;

public sealed class Workshop : Entity, ITenantOwnedEntity
{
    public static Workshop Create(Guid id, Guid tenantId, WorkshopMutationValues values)
    {
        ArgumentOutOfRangeException.ThrowIfEqual(id, Guid.Empty);
        ArgumentOutOfRangeException.ThrowIfEqual(tenantId, Guid.Empty);
        ArgumentNullException.ThrowIfNull(values);

        return new Workshop(
            id,
            tenantId,
            NormalizeRequired(values.Code, nameof(values.Code)),
            NormalizeRequired(values.Name, nameof(values.Name)),
            NormalizeOptional(values.Description),
            values.IsActive);
    }

    private Workshop(
        Guid id,
        Guid tenantId,
        string code,
        string name,
        string? description,
        bool isActive)
        : base(id)
    {
        TenantId = tenantId;
        Code = code;
        Name = name;
        Description = description;
        IsActive = isActive;
        ConcurrencyToken = Guid.NewGuid();
    }

    private Workshop()
        : base(Guid.Empty)
    {
    }

    public Guid TenantId { get; private set; }

    public string Code { get; private set; } = string.Empty;

    public string Name { get; private set; } = string.Empty;

    public string? Description { get; private set; }

    public bool IsActive { get; private set; }

    public Guid ConcurrencyToken { get; private set; }

    public bool Update(WorkshopMutationValues values)
    {
        ArgumentNullException.ThrowIfNull(values);

        var description = NormalizeOptional(values.Description);
        if (Code == values.Code
            && Name == values.Name
            && Description == description
            && IsActive == values.IsActive)
        {
            return false;
        }

        Code = NormalizeRequired(values.Code, nameof(values.Code));
        Name = NormalizeRequired(values.Name, nameof(values.Name));
        Description = description;
        IsActive = values.IsActive;
        ConcurrencyToken = Guid.NewGuid();

        return true;
    }

    private static string? NormalizeOptional(string? value) =>
        string.IsNullOrWhiteSpace(value) ? null : value.Trim();

    private static string NormalizeRequired(string? value, string paramName)
    {
        if (string.IsNullOrWhiteSpace(value))
        {
            ArgumentException.ThrowIfNullOrWhiteSpace(value, paramName);
        }

        return value.Trim();
    }

    public sealed record WorkshopMutationValues(
        string Code,
        string Name,
        string? Description,
        bool IsActive);
}
```

Что происходит:

- сущность хранит состояние и локальные инварианты;
- сущность не знает про HTTP, UI, DbContext, Workflow и Runtime API;
- метод `Update` возвращает `false`, если фактических изменений нет;
- для первого объекта поля `Code` и `Name` объявлены явно; наследование от
  `CommonCatalogObject` можно добавить позже по примеру ProductDefinition.

Проверка:

```powershell
dotnet build src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj --no-restore
```

Эталон:

- `Objects/Nomenclature/Domain/Nomenclature.cs`;
- `Objects/NomenclatureAdditionalUnit/Domain/NomenclatureAdditionalUnit.cs`.

## 8. Шаг 4. Добавить DbContext модуля

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/Application/Abstractions/IWorkshopCatalogDbContext.cs`:

```csharp
using DMP.DemoModules.WorkshopCatalog.Domain.Entities;
using Microsoft.EntityFrameworkCore;

namespace DMP.DemoModules.WorkshopCatalog.Application.Abstractions;

public interface IWorkshopCatalogDbContext
{
    DbSet<Workshop> Workshops { get; }

    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
}
```

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/Infrastructure/Persistence/WorkshopCatalogDbContext.cs`:

```csharp
using DMP.DemoModules.WorkshopCatalog.Application.Abstractions;
using DMP.DemoModules.WorkshopCatalog.Domain.Entities;
using Microsoft.EntityFrameworkCore;

namespace DMP.DemoModules.WorkshopCatalog.Infrastructure.Persistence;

internal sealed class WorkshopCatalogDbContext(DbContextOptions<WorkshopCatalogDbContext> options)
    : DbContext(options), IWorkshopCatalogDbContext
{
    public DbSet<Workshop> Workshops => Set<Workshop>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.HasDefaultSchema("demo_workshop_catalog");

        modelBuilder.Entity<Workshop>(entity =>
        {
            entity.ToTable("Workshops");
            entity.HasKey(x => x.Id);
            entity.Property(x => x.TenantId).IsRequired();
            entity.Property(x => x.ConcurrencyToken).IsRequired().IsConcurrencyToken();
            entity.Property(x => x.Code).HasMaxLength(64).IsRequired();
            entity.Property(x => x.Name).HasMaxLength(256).IsRequired();
            entity.Property(x => x.Description).HasMaxLength(1024).IsRequired(false);
            entity.Property(x => x.IsActive).IsRequired();
            entity.HasIndex(x => new { x.TenantId, x.Code }).IsUnique();
        });
    }
}
```

Что происходит:

- модуль владеет своей таблицей и схемой БД;
- Object Runtime получает данные через DbContext модуля, а не через общую
  таблицу всех бизнес-объектов;
- уникальность `TenantId + Code` защищает каталог внутри tenant.

Проверка:

```powershell
dotnet build src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj --no-restore
```

Эталон:

- `Infrastructure/Persistence/DemoProductDefinitionDbContext.cs`;
- `Application/Abstractions/IDemoProductDefinitionDbContext.cs`.

## 9. Шаг 5. Описать контракт Object Runtime

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/Objects/Workshop/Runtime/WorkshopChangeValues.cs`:

```csharp
namespace DMP.DemoModules.WorkshopCatalog.Runtime.Contracts;

public sealed record WorkshopChangeValues(
    string Code,
    string Name,
    string? Description,
    bool IsActive)
{
    public DMP.DemoModules.WorkshopCatalog.Domain.Entities.Workshop.WorkshopMutationValues ToDomainMutationValues() =>
        new(Code, Name, Description, IsActive);
}
```

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/Objects/Workshop/Runtime/WorkshopObjectContract.cs`:

```csharp
using DMP.DemoModules.WorkshopCatalog.Application.Abstractions;
using DMP.DemoModules.WorkshopCatalog.Domain.Entities;
using DMP.Platform.Runtime.Application.Abstractions;
using WorkshopFields = DMP.DemoModules.WorkshopCatalog.WorkshopCatalogCodes.Workshop.Fields;

namespace DMP.DemoModules.WorkshopCatalog.Runtime.Contracts;

public sealed class WorkshopObjectContract : BusinessObjectContract<Workshop>
{
    public override void Configure(BusinessObjectBuilder<Workshop> objectType)
    {
        objectType
            .Module(WorkshopCatalogCodes.ModuleCode)
            .ObjectType(WorkshopCatalogCodes.Workshop.ObjectTypeCode)
            .Presentation(presentation => presentation
                .DisplayName(display => display
                    .Fields(WorkshopFields.Code, WorkshopFields.Name)
                    .Format("{Code} - {Name}")
                    .Virtual()))
            .Key(WorkshopFields.Id, x => x.Id)
            .Tenant(WorkshopFields.TenantId, x => x.TenantId)
            .ConcurrencyToken(WorkshopFields.ConcurrencyToken, x => x.ConcurrencyToken)
            .Member(WorkshopFields.Code, x => x.Code, member => member.Required())
            .Member(WorkshopFields.Name, x => x.Name, member => member.Required())
            .Member(WorkshopFields.Description, x => x.Description)
            .Member(WorkshopFields.IsActive, x => x.IsActive)
            .Dataset(WorkshopCatalogCodes.Workshop.ListDatasetCode, dataset => dataset.List().Filtering().Sorting())
            .Dataset(WorkshopCatalogCodes.Workshop.LookupDatasetCode, dataset => dataset.Lookup().Filtering().Sorting())
            .Repository<IWorkshopCatalogDbContext>(repository => repository.Workshops)
            .Mapping(mapping => mapping
                .ReadProjectionFromMembers()
                .CreateUpdateUsingGeneratedWriter(writer => writer
                    .Values<WorkshopChangeValues>(values => values.BindByMemberCode())
                    .CreateUsing((context, values) =>
                        Workshop.Create(context.NewId, context.TenantId, values.ToDomainMutationValues()))
                    .UpdateUsing((entity, values) =>
                        entity.Update(values.ToDomainMutationValues()))))
            .GenerateDefaults(defaults => defaults
                .Value(WorkshopFields.IsActive, true));
    }
}
```

Что происходит:

- `BusinessObjectContract<T>` становится исполняемым описанием объекта;
- в `contract` объявляются `members`, `datasets`, `display name`, `repository`
  и `mapping`;
- `Key`, `Tenant` и `ConcurrencyToken` связывают runtime с техническими полями
  объекта;
- `generated writer` связывает `runtime values` с доменными методами `Create` и
  `Update`.

Проверка:

```powershell
dotnet build src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj --no-restore
```

Эталон:

- `Objects/Nomenclature/Runtime/NomenclatureObjectContract.cs`;
- `Objects/NomenclatureAdditionalUnit/Runtime/NomenclatureAdditionalUnitObjectContract.cs`.

## 10. Шаг 6. Описать регистрацию модуля

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/Configuration/Registration/WorkshopCatalogModuleRegistration.cs`:

```csharp
using DMP.DemoModules.WorkshopCatalog.Runtime.Contracts;
using DMP.Platform.Contracts.Configuration.Registration;

namespace DMP.DemoModules.WorkshopCatalog.Configuration.Registration;

public sealed class WorkshopCatalogModuleRegistration : IConfigurationModuleManifest
{
    public ConfigurationModuleRegistration Describe()
    {
        return ModuleRegistration
            .ForModule(
                WorkshopCatalogCodes.ModuleCode,
                "Workshop Catalog",
                moduleVersion: "0.1.0",
                contractsVersion: "0.1.0")
            .FromBusinessObject<WorkshopObjectContract>()
            .Permissions(
                new ConfigurationPermissionRegistration(WorkshopCatalogCodes.Workshop.Permissions.View, "Workshop View"),
                new ConfigurationPermissionRegistration(WorkshopCatalogCodes.Workshop.Permissions.Create, "Workshop Create"),
                new ConfigurationPermissionRegistration(WorkshopCatalogCodes.Workshop.Permissions.Edit, "Workshop Edit"),
                new ConfigurationPermissionRegistration(WorkshopCatalogCodes.Workshop.Permissions.Delete, "Workshop Delete"))
            .Build();
    }
}
```

Что происходит:

- платформа узнаёт, какие `object types`, `datasets` и `permissions` публикует модуль;
- объект подтягивается из `WorkshopObjectContract`, а не описывается повторно
  вручную;
- версия `contract` фиксирует форму публичных кодов.

Проверка:

```powershell
rg "FromBusinessObject<WorkshopObjectContract>|Workshop View" src\Demos\DMP.DemoModules.WorkshopCatalog
```

Эталон: `Configuration/Registration/DemoProductDefinitionModuleRegistration.cs`.

## 11. Шаг 7. Создать baseline configuration

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/Configuration/Baseline/WorkshopCatalogBaselineConfiguration.cs`:

```csharp
using DMP.DemoModules.WorkshopCatalog.Domain.Entities;
using DMP.DemoModules.WorkshopCatalog.Runtime.Contracts;
using DMP.Platform.Contracts.Configuration.BaselinePackages;
using DMP.Platform.Contracts.Configuration.BaselinePackages.Authoring;
using WorkshopFields = DMP.DemoModules.WorkshopCatalog.WorkshopCatalogCodes.Workshop.Fields;

namespace DMP.DemoModules.WorkshopCatalog.Configuration.Baseline;

public sealed class WorkshopCatalogBaselineConfiguration : IConfigurationBaselinePackageManifest
{
    public IReadOnlyCollection<ConfigurationBaselinePackage> DescribePackages()
    {
        return
        [
            ConfigurationBaselinePackageBuilder
                .ForModule(WorkshopCatalogCodes.ModuleCode, "Demo.WorkshopCatalog.Baseline", "v1")
                .Compatibility("0.1.0", ">=0.1.0")
                .FromBusinessObject<Workshop, WorkshopObjectContract>(objectType => objectType
                    .TitleText(t => t
                        .Invariant("Workshop")
                        .Translation("en-US", "Workshop")
                        .Translation("ru-RU", "Цех"))
                    .DefaultListView(WorkshopCatalogCodes.ModuleCode, WorkshopCatalogCodes.Workshop.ListViewCode)
                    .DefaultDetailView(WorkshopCatalogCodes.ModuleCode, WorkshopCatalogCodes.Workshop.DetailViewCode)
                    .DefaultLookupView(WorkshopCatalogCodes.ModuleCode, WorkshopCatalogCodes.Workshop.LookupViewCode)
                    .Member(x => x.Description, member => member
                        .TitleText(t => t
                            .Invariant("Description")
                            .Translation("en-US", "Description")
                            .Translation("ru-RU", "Описание")))
                    .Member(x => x.IsActive, member => member
                        .TitleText(t => t
                            .Invariant("Active")
                            .Translation("en-US", "Active")
                            .Translation("ru-RU", "Активен")))
                    .View(
                        WorkshopCatalogCodes.Workshop.ListViewCode,
                        view => view
                            .TitleText(t => t
                                .Invariant("Workshops")
                                .Translation("en-US", "Workshops")
                                .Translation("ru-RU", "Цеха"))
                            .ObjectList()
                            .Dataset(WorkshopCatalogCodes.Workshop.ListDatasetCode)
                            .Column(WorkshopFields.Code)
                            .Column(WorkshopFields.Name)
                            .Column(WorkshopFields.IsActive))
                    .View(
                        WorkshopCatalogCodes.Workshop.DetailViewCode,
                        view => view
                            .TitleText(t => t
                                .Invariant("Workshop card")
                                .Translation("en-US", "Workshop card")
                                .Translation("ru-RU", "Карточка цеха"))
                            .ObjectForm()
                            .Template("DetailTemplate")
                            .FormElement(WorkshopFields.Code)
                            .FormElement(WorkshopFields.Name)
                            .FormElement(WorkshopFields.Description)
                            .FormElement(WorkshopFields.IsActive))
                    .View(
                        WorkshopCatalogCodes.Workshop.LookupViewCode,
                        view => view
                            .TitleText(t => t
                                .Invariant("Workshop lookup")
                                .Translation("en-US", "Workshop lookup")
                                .Translation("ru-RU", "Выбор цеха"))
                            .LookupList()
                            .Dataset(WorkshopCatalogCodes.Workshop.LookupDatasetCode)
                            .Column(WorkshopFields.Code)
                            .Column(WorkshopFields.Name)))
                .Build()
        ];
    }
}
```

Что происходит:

- baseline создаёт публикуемые `configuration artifacts` для объекта;
- структурные поля берутся из `WorkshopObjectContract`;
- baseline добавляет пользовательские названия, `list`/`detail`/`lookup` views и
  размещение полей;
- бизнес-логику, `repository` и `lifecycle` в baseline не пишут.

Проверка:

```powershell
dotnet build src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj --no-restore
```

Эталон:

- `Objects/Nomenclature/Configuration/NomenclatureBaselineConfiguration.cs`;
- `Objects/NomenclatureAdditionalUnit/Configuration/NomenclatureAdditionalUnitBaselineConfiguration.cs`.

Если вы меняете baseline уже существующего модуля, в задаче или review нужно
явно написать, требуется ли пересборка или переиздание baseline.

## 12. Шаг 8. Подключить DI модуля

Создайте файл
`src/Demos/DMP.DemoModules.WorkshopCatalog/WorkshopCatalogDependencyInjection.cs`:

```csharp
using DMP.DemoModules.WorkshopCatalog.Application.Abstractions;
using DMP.DemoModules.WorkshopCatalog.Configuration.Baseline;
using DMP.DemoModules.WorkshopCatalog.Configuration.Registration;
using DMP.DemoModules.WorkshopCatalog.Infrastructure.Persistence;
using DMP.DemoModules.WorkshopCatalog.Runtime.Contracts;
using DMP.Platform.Contracts.Configuration.BaselinePackages;
using DMP.Platform.Contracts.Configuration.Registration;
using DMP.Platform.Runtime.Application.Abstractions;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DMP.DemoModules.WorkshopCatalog;

public static class WorkshopCatalogDependencyInjection
{
    public static IServiceCollection AddWorkshopCatalogModule(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("Platform");
        var inMemoryDatabaseName = configuration["Demo.WorkshopCatalog:InMemoryDatabaseName"] ?? "demo-workshop-catalog";

        services.AddDbContext<WorkshopCatalogDbContext>(options =>
        {
            if (string.IsNullOrWhiteSpace(connectionString))
            {
                options.UseInMemoryDatabase(inMemoryDatabaseName);
                return;
            }

            options.UseSqlServer(connectionString, sql =>
            {
                sql.MigrationsHistoryTable("__EFMigrationsHistory", "demo_workshop_catalog");
            });
        });

        services.AddScoped<IWorkshopCatalogDbContext>(sp => sp.GetRequiredService<WorkshopCatalogDbContext>());

        services.AddSingleton<WorkshopObjectContract>();
        services.AddSingleton<IBusinessObjectContract>(sp => sp.GetRequiredService<WorkshopObjectContract>());
        services.AddSingleton(sp => sp.GetRequiredService<WorkshopObjectContract>().Build());
        services.AddSingleton(sp => ObjectRuntimeDescriptor.FromContract(sp.GetRequiredService<WorkshopObjectContract>().Build()));

        services.AddSingleton<IConfigurationModuleManifest, WorkshopCatalogModuleRegistration>();
        services.AddSingleton<IConfigurationBaselinePackageManifest, WorkshopCatalogBaselineConfiguration>();
        services.AddSingleton(new ObjectRuntimeDescriptorMigrationModule(WorkshopCatalogCodes.ModuleCode));

        return services;
    }
}
```

Что происходит:

- DbContext модуля регистрируется в DI;
- контракт регистрируется как `IBusinessObjectContract`;
- контракт собирается в `BusinessObjectContractDefinition`;
- определение превращается в `ObjectRuntimeDescriptor`;
- baseline и регистрация модуля становятся доступными платформенной
  композиции.

Проверка:

```powershell
dotnet build src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj --no-restore
```

Эталон: `DemoProductDefinitionDependencyInjection.cs`.

## 13. Шаг 9. Подключить модуль к host

Выберите host по назначению модуля:

| Модуль | Расположение | Host |
|---|---|---|
| Учебный или демонстрационный | `src/Demos/DMP.DemoModules.*` | `src/Hosts/DMP.Platform.DemoApi` |
| Реальный прикладной | `src/Modules/DMP.Modules.*` | `src/Hosts/DMP.Platform.Api` |

### 13.1. Учебный модуль: подключение к DemoApi

Для создаваемого в этом документе `DMP.DemoModules.WorkshopCatalog` добавьте
ссылку на проект:

```powershell
dotnet add src\Hosts\DMP.Platform.DemoApi\DMP.Platform.DemoApi.csproj reference `
  src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj
```

В `src/Hosts/DMP.Platform.DemoApi/Program.cs` добавьте `using`:

```csharp
using DMP.DemoModules.WorkshopCatalog;
```

И зарегистрируйте модуль до `AddPlatformApi`:

```csharp
builder.Services.AddWorkshopCatalogModule(builder.Configuration);
builder.Services.AddPlatformApi(builder.Configuration);
```

Что происходит:

- демо-хост получает новый модуль только в композиции разработки;
- боевой хост не должен ссылаться на демо-модуль;
- платформа увидит registration, baseline и runtime descriptor на старте.

Проверка:

```powershell
dotnet build src\Hosts\DMP.Platform.DemoApi\DMP.Platform.DemoApi.csproj --no-restore
```

Эталон: `src/Hosts/DMP.Platform.DemoApi/Program.cs`.

### 13.2. Реальный прикладной модуль: подключение к production host

Не подключайте проект из `src/Demos` к production host. Для реального аналога
модуля сначала используйте production-имя и расположение, например
`src/Modules/DMP.Modules.WorkshopCatalog/DMP.Modules.WorkshopCatalog.csproj` и
пространство имён `DMP.Modules.WorkshopCatalog`.

Добавьте ссылку из production host на проект реального модуля:

```powershell
dotnet add src\Hosts\DMP.Platform.Api\DMP.Platform.Api.csproj reference `
  src\Modules\DMP.Modules.WorkshopCatalog\DMP.Modules.WorkshopCatalog.csproj
```

В
`src/Hosts/DMP.Platform.Api/Composition/PlatformApiCompositionExtensions.cs`
добавьте `using`:

```csharp
using DMP.Modules.WorkshopCatalog;
```

И зарегистрируйте модуль в `AddPlatformApi` рядом с остальными прикладными
модулями:

```csharp
services.AddWorkshopCatalogModule(configuration);
services.AddGeneralMasterDataModule(configuration);
services.AddPlantStructureModule(configuration);
```

Если модуль имеет собственный `*DatabaseInitializer`, добавьте его вызов в
`InitializePlatformApiAsync` рядом с инициализаторами остальных прикладных
модулей production host. Например:

```csharp
await WorkshopCatalogDatabaseInitializer.InitializeAsync(services, cancellationToken);
```

Проверка:

```powershell
dotnet build src\Hosts\DMP.Platform.Api\DMP.Platform.Api.csproj --no-restore
```

Эталон подключения production-модулей:
`src/Hosts/DMP.Platform.Api/Composition/PlatformApiCompositionExtensions.cs` и
`src/Hosts/DMP.Platform.Api/DMP.Platform.Api.csproj`.

## 14. Шаг 10. Запустить выбранный host

Если локальные настройки ещё не созданы:

```powershell
.\scripts\Initialize-LocalSettings.ps1
```

Скрипт создаёт локальные настройки и для `DMP.Platform.DemoApi`, и для
`DMP.Platform.Api`.

### 14.1. Запуск DemoApi для учебного модуля

```powershell
dotnet run --project src\Hosts\DMP.Platform.DemoApi\DMP.Platform.DemoApi.csproj --launch-profile http
```

Что происходит:

- DemoApi стартует только в `Development`;
- адрес по профилю `http`: `http://127.0.0.1:5090`;
- подключаются API платформы, ProductDefinition и ваш новый demo-модуль;
- при наличии строки подключения к SQL Server используются миграции и схема
  модуля, иначе возможен режим in-memory для локальных проверок.

### 14.2. Запуск production host для реального прикладного модуля

```powershell
dotnet run --project src\Hosts\DMP.Platform.Api\DMP.Platform.Api.csproj --launch-profile http
```

При локальном запуске профиль `http` также использует
`http://127.0.0.1:5090`, но состав приложения соответствует production host:
подключаются production-модули из `src/Modules`, а проекты из `src/Demos` в
него не входят. Оба host используют один порт, поэтому одновременно запускать
их с профилем `http` нельзя.

### 14.3. Проверка Runtime API

Следующие запросы выполняйте после запуска выбранного host:

```powershell
$tenantId = [guid]::NewGuid()
$userId = [guid]::NewGuid()

Invoke-RestMethod `
  -Method Get `
  -Uri "http://127.0.0.1:5090/api/runtime/entry-point" `
  -Headers @{
    "X-Tenant-Id" = $tenantId
    "X-User-Id" = $userId
  }
```

Для списка учебного `Workshop`:

```powershell
Invoke-RestMethod `
  -Method Post `
  -Uri "http://127.0.0.1:5090/api/runtime/objects/Workshop/list" `
  -ContentType "application/json" `
  -Headers @{
    "X-Tenant-Id" = $tenantId
    "X-User-Id" = $userId
  } `
  -Body (@{
    moduleCode = "Demo.WorkshopCatalog"
    objectTypeCode = "Workshop"
    datasetCode = "Workshop_ListDataset"
    page = 1
    pageSize = 25
  } | ConvertTo-Json -Depth 10)
```

Ожидаемый результат: API возвращает валидный runtime-ответ, даже если список
пока пустой.

Для реального прикладного модуля выполните такой же запрос к production host,
заменив `moduleCode`, `objectTypeCode` и `datasetCode` на стабильные коды этого
модуля. Коды `Demo.WorkshopCatalog` нельзя переносить в production-модуль.

Подробнее о заголовках и маршрутах: [контракты Object Runtime](../03_platform/03_object_runtime/03_contracts.md).

## 15. Шаг 11. Добавить минимальные проверки

Создайте тестовый класс по образцу ProductDefinition:

```powershell
New-Item -ItemType Directory -Force tests\DMP.Platform.IntegrationTests\WorkshopCatalog | Out-Null
```

Минимальная идея теста:

```csharp
using DMP.DemoModules.WorkshopCatalog;
using DMP.DemoModules.WorkshopCatalog.Configuration.Registration;
using DMP.DemoModules.WorkshopCatalog.Runtime.Contracts;
using DMP.Platform.IntegrationTests.Infrastructure;

namespace DMP.Platform.IntegrationTests.Runtime;

[Trait(TestCategories.Name, TestCategories.DemoModuleInternal)]
public sealed class WorkshopCatalogContractTests
{
    [Fact]
    public void Build_WorkshopContract_ReturnsStableObjectShape()
    {
        var definition = new WorkshopObjectContract().Build();

        Assert.Equal(WorkshopCatalogCodes.ModuleCode, definition.ModuleCode);
        Assert.Equal(WorkshopCatalogCodes.Workshop.ObjectTypeCode, definition.ObjectTypeCode);
        Assert.Contains(definition.Members, x => x.Code == WorkshopCatalogCodes.Workshop.Fields.Code);
        Assert.Contains(definition.Members, x => x.Code == WorkshopCatalogCodes.Workshop.Fields.Name);
        Assert.Contains(definition.Datasets, x => x.Code == WorkshopCatalogCodes.Workshop.ListDatasetCode);
    }

    [Fact]
    public void Registration_PublishesWorkshopObject()
    {
        var registration = new WorkshopCatalogModuleRegistration().Describe();

        Assert.Equal(WorkshopCatalogCodes.ModuleCode, registration.ModuleCode);
        Assert.Contains(registration.ObjectTypes, x => x.Code == WorkshopCatalogCodes.Workshop.ObjectTypeCode);
    }
}
```

Команда для запуска только ваших проверок:

```powershell
dotnet test tests\DMP.Platform.IntegrationTests\DMP.Platform.IntegrationTests.csproj `
  --no-restore `
  --filter "FullyQualifiedName~WorkshopCatalogContractTests"
```

Команда для сверки с эталонными ProductDefinition-проверками:

```powershell
dotnet test tests\DMP.Platform.IntegrationTests\DMP.Platform.IntegrationTests.csproj `
  --no-restore `
  --filter "FullyQualifiedName~DemoProductDefinitionBusinessObjectContractIntegrationTests|FullyQualifiedName~DemoProductDefinitionIdentityContractTests"
```

Важно: не запускайте полный набор интеграционных тестов во время разработки без
отдельной договорённости. Тесты, которые подключаются к SQL Server через Windows
Authentication/SSPI, нужно запускать вне песочницы под Windows-учётной записью
разработчика.

## 16. Как понять, что всё сделано правильно

Минимальный чек-лист:

| Проверка | Команда или место | Ожидаемый результат |
|---|---|---|
| Проект собирается | `dotnet build src\Demos\DMP.DemoModules.WorkshopCatalog\DMP.DemoModules.WorkshopCatalog.csproj --no-restore` | Ошибок компиляции нет |
| DemoApi собирается | `dotnet build src\Hosts\DMP.Platform.DemoApi\DMP.Platform.DemoApi.csproj --no-restore` | Ошибок компиляции нет |
| Production host с реальным модулем собирается | `dotnet build src\Hosts\DMP.Platform.Api\DMP.Platform.Api.csproj --no-restore` | Ошибок компиляции нет, production host не ссылается на проекты из `src/Demos` |
| Contract строится | `new WorkshopObjectContract().Build()` в тесте | Есть module code, object type, members и datasets |
| Registration видит объект | `new WorkshopCatalogModuleRegistration().Describe()` | `Workshop` есть в `ObjectTypes` |
| Runtime endpoint отвечает | `POST /api/runtime/objects/Workshop/list` | Возвращается runtime list response |

Если менялись baseline configuration или миграции, отдельно зафиксируйте:

- нужна ли пересборка или переиздание baseline;
- нужно ли добавить EF migration;
- какие локальные БД надо пересоздать или обновить.

## 17. Что добавлять после первого объекта

Быстрый старт показывает только минимальный вертикальный срез. Он не является
каталогом всех возможностей ядра платформы. Расширенные сценарии уже
представлены в `DMP.DemoModules.ProductDefinition`; если нужный шаблон
реализации есть в демо-модуле, разработчик должен использовать его как эталон,
а не дорабатывать ядро или общий рантайм фронтенда в задаче своего модуля.

### 17.1. Как выбирать следующий шаг

| Ситуация | Действие |
|---|---|
| Сценарий описан в этом быстром старте | Копируйте шаги и адаптируйте имена модуля, `object type`, `fields` и `baseline`. |
| Сценария нет в быстром старте, но он есть в `ProductDefinition` | Используйте демо-модуль как эталонный шаблон и перенесите только нужную часть. |
| Сценарий описан в `docs-new`, но не показан в демо-модуле | Проверьте статус реализации, тесты и владельца платформенной области до разработки. |
| Сценария нет ни в быстром старте, ни в демо-модуле, ни в подтверждённой платформенной документации | Оформите заявку на платформенную возможность по правилам раздела 4. |

### 17.2. Карта возможностей ProductDefinition

| Возможность | Где смотреть в демо-модуле | Дополнительная документация |
|---|---|---|
| Базовый catalog object с общими полями | `Objects/Nomenclature/Runtime/NomenclatureObjectContract.cs`, `.BaseObject<CommonCatalogObjectObjectContract>()` | `docs-new/03_platform/03_object_runtime/03_contracts.md` |
| Управление удалением, архивом и catalog policy | `NomenclatureObjectContract.Management(...)`, `NomenclatureHardDeleteGuard.cs` | `docs-new/03_platform/03_object_runtime/04_runtime.md`, `docs-new/03_platform/03_object_runtime/05_security_and_audit.md` |
| Создание на основании существующего объекта | `NomenclatureObjectContract.CreateFromExisting(...)`, `NomenclatureAdditionalUnitObjectContract.CreateFromExisting(...)` | `docs-new/03_platform/03_object_runtime/04_runtime.md` |
| Обязательные поля и значения по умолчанию | `NomenclatureObjectContract.GenerateDefaults(...)`, `NomenclatureDefaultsProvider.cs` | `docs-new/03_platform/03_object_runtime/03_contracts.md` |
| ValueSet-поля | `NomenclatureObjectContract.ValueSetMember(...)`, `NomenclatureBaselineConfiguration.ValueSet(...)` | `docs-new/03_platform/06_value_sets/00_platform_overview.md` |
| SystemEnum и flags enum | `NomenclatureBaselineConfiguration.SystemEnum(...)`, `SystemEnumFromEnum<NomenclatureLifecycleKind>(...)` | `docs-new/03_platform/02_configuration/artifact_types/system_enum.md` |
| TimeAmount-поля | `NomenclatureObjectContract.TimeAmountMember(...)` | `docs-new/03_platform/07_settings/00_platform_overview.md` |
| `Reference`-поле на объект | `NomenclatureObjectContract.ReferenceMember(...)` для `Analog` | `docs-new/03_platform/03_object_runtime/03_contracts.md` |
| Lookup с предметным поставщиком ограничений выбора | `NomenclatureSupplySourceSelectionProvider.cs`, поле `PreferredSupplySource` | `docs-new/03_platform/03_object_runtime/04_runtime.md` |
| Агрегатная коллекция `WithOwner` | `NomenclatureObjectContract.AggregateCollection(...)`, `NomenclatureAdditionalUnitObjectContract.cs` | `docs-new/03_platform/03_object_runtime/04_runtime.md` |
| Отдельно сохраняемая коллекция `Separate` | `NomenclatureObjectContract.AssociationCollection(...)`, `NomenclatureRelationObjectContract.cs` | `docs-new/03_platform/03_object_runtime/04_runtime.md` |
| Action с обработчиком команды | `UpdateNomenclatureExternalIdCommand*`, `SendNomenclatureNotificationCommand*` | `docs-new/03_platform/03_object_runtime/03_contracts.md`, `docs-new/03_platform/02_configuration/artifact_types/action.md` |
| File как параметр action во вложенной коллекции | `Objects/NomenclatureAttachment/Runtime/NomenclatureAttachmentObjectContract.cs`, `Objects/NomenclatureAttachment/Configuration/NomenclatureAttachmentBaselineConfiguration.cs`, action `NomenclatureAttachment_UploadFiles` | `docs-new/03_platform/02_configuration/artifact_types/view.md`, `docs-new/03_platform/03_object_runtime/04_runtime.md`, `docs-new/03_platform/13_content_storage/06_user_experience.md` |
| File как реквизит объекта в карточке и inline-редакторе | `Objects/NomenclatureAdditionalUnit/Runtime/NomenclatureAdditionalUnitObjectContract.cs`, `Objects/NomenclatureAdditionalUnit/Configuration/NomenclatureAdditionalUnitBaselineConfiguration.cs`, `NomenclatureAdditionalUnit.File` | `docs-new/03_platform/13_content_storage/06_user_experience.md`, `docs-new/03_platform/12_frontend_platform/04_runtime.md` |
| Action, изменяющий сам объект через `mutation planner` | `UpdateNomenclatureExternalIdMutationPlanner.cs` | `docs-new/03_platform/03_object_runtime/04_runtime.md` |
| Групповой Action | `BulkSetNomenclatureCategoryCodeCommandHandler.cs`, `BulkSetNomenclatureIsPurchasedCommandHandler.cs` | `docs-new/03_platform/03_object_runtime/03_contracts.md` |
| Validator | `NomenclatureValidator.cs`, validators связанных объектов | `docs-new/03_platform/03_object_runtime/04_runtime.md` |
| Lifecycle handler | `NomenclatureLifecycleHandler.cs` | `docs-new/03_platform/03_object_runtime/04_runtime.md`, `docs-new/03_platform/03_object_runtime/05_security_and_audit.md` |
| Привязка состояния Workflow | `NomenclatureObjectContract.Stateful(...)`, блок workflow в `NomenclatureBaselineConfiguration.cs` | `docs-new/03_platform/04_workflow/00_platform_overview.md` |
| Расширяемые значения объекта | `NomenclatureObjectContract.SupportsExtensions(...)`, `NomenclatureExtensionValueStore.cs` | `docs-new/03_platform/03_object_runtime/03_contracts.md` |
| Abstract reference-only type | `NomenclatureSupplySourceObjectContract.AbstractReferenceOnly()` | `docs-new/03_platform/03_object_runtime/02_architecture.md` |
| TPH-наследники в общей таблице | `InternalProductionSourceObjectContract.StoredInBaseTable(...)`, `ExternalSupplierSourceObjectContract.StoredInBaseTable(...)` | `docs-new/03_platform/03_object_runtime/02_architecture.md` |
| Иерархия объекта | `InternalProductionSourceObjectContract.Hierarchy(...)` | `docs-new/03_platform/03_object_runtime/04_runtime.md` |
| Представления `list`/`detail`/`lookup` и `layout` | `NomenclatureBaselineConfiguration.View(...)`, `NomenclatureAdditionalUnitBaselineConfiguration.cs` | `docs-new/03_platform/12_frontend_platform/00_platform_overview.md` |
| `MemberBehavior` и локальные UI-реакции | `NomenclatureBaselineConfiguration.MemberBehavior(...)` | `docs-new/03_platform/02_configuration/artifact_types/object_type.md` |
| Reports and outputs | `NomenclatureBaselineConfiguration.Report(...)`, `.Output(...)` | `docs-new/03_platform/11_reporting_output/00_platform_overview.md` |
| Navigation menu | `DemoProductDefinitionNavigationBaselineConfiguration.cs` | `docs-new/03_platform/02_configuration/artifact_types/menu.md` |
| Settings | `ProductDefinitionSettingsConfiguration.cs`, `ProductDefinitionSettings.cs` | `docs-new/03_platform/07_settings/00_platform_overview.md` |
| Permissions and roles | `ProductDefinitionSecurityConfiguration.cs` | `docs-new/03_platform/01_tenant_and_security/00_platform_overview.md` |
| Регрессионные тесты runtime-контрактов | `DemoProductDefinitionBusinessObjectContractIntegrationTests.cs` | `tests/DMP.Platform.IntegrationTests/DemoProductDefinition/README.md` |

Главное правило: сначала объявите структурную runtime-модель в
`BusinessObjectContract<T>`, затем добавляйте `presentation`, `localization`,
`views`, `workflow`, `actions` и `menu` в `baseline`.

Не переносите всю `Nomenclature` целиком в новый модуль. Берите из неё только
тот шаблон реализации, который нужен текущему `object type`, и добавляйте
собственные тесты на стабильность контракта.

## 18. Частые ошибки

| Ошибка | Как правильно |
|---|---|
| Писать отдельный CRUD controller для стандартного объекта | Использовать Runtime API и `BusinessObjectContract<T>` |
| Дублировать поля объекта в `baseline` вручную | Брать shell из `.FromBusinessObject<TEntity, TContract>(...)` |
| Хранить бизнес-логику в DbContext или repository | Держать её в `entity`, `validator`, `lifecycle handler` или `command handler` |
| Использовать строковые литералы кодов по всему модулю | Завести `*Codes.cs` и ссылаться на него |
| Смешивать `ActionCode` и `workflow command` | `Action` описывает объектную операцию, `workflow command` относится к машине состояний |
| Считать `View` структурой объекта | `View` описывает отображение; структура объекта живёт в contract |
| Считать отсутствие сценария в быстром старте отсутствием платформенной возможности | Проверить `ProductDefinition`, профильную документацию в `docs-new` и тесты |
| Исправлять нехватку платформы в коде своего модуля | Оформить платформенную задачу с универсальным контрактом и критериями приёмки |
| Жёстко привязывать общий рантайм фронтенда к одному `ObjectTypeCode` | Расширить декларативный контракт фронтенд-рантайма |
| Лезть в таблицы чужого модуля | Использовать `reference`, `lookup`, публичный контракт или `projection` |
| Запускать полный набор интеграционных тестов для маленькой правки | Запускать только затронутые тестовые классы |

## 19. Минимальная карта источников

- `src/Demos/DMP.DemoModules.ProductDefinition/README.md` - назначение и запуск демо-модуля.
- `src/Hosts/DMP.Platform.DemoApi/README.md` - запуск хоста разработки.
- `src/Hosts/DMP.Platform.Api/Composition/PlatformApiCompositionExtensions.cs` - подключение production-модулей к production host.
- `src/Hosts/DMP.Platform.Api/DMP.Platform.Api.csproj` - ссылки production host на проекты прикладных модулей.
- `src/Demos/DMP.DemoModules.ProductDefinition/DemoProductDefinitionDependencyInjection.cs` - DI и регистрация descriptor.
- `src/Demos/DMP.DemoModules.ProductDefinition/Configuration/Registration/DemoProductDefinitionModuleRegistration.cs` - манифест модуля.
- `src/Demos/DMP.DemoModules.ProductDefinition/Objects/Nomenclature/Runtime/NomenclatureObjectContract.cs` - самый полный контракт объекта.
- `src/Demos/DMP.DemoModules.ProductDefinition/Objects/NomenclatureAdditionalUnit/Runtime/NomenclatureAdditionalUnitObjectContract.cs` - компактный контракт объекта для строки коллекции.
- `src/Demos/DMP.DemoModules.ProductDefinition/Objects/Nomenclature/Configuration/NomenclatureBaselineConfiguration.cs` - baseline с views, actions, workflow, value sets, reports и outputs.
- `tests/DMP.Platform.IntegrationTests/DemoProductDefinition/README.md` - категории и правила запуска тестов демо-модуля.
- `docs-new/03_platform/03_object_runtime/00_platform_overview.md` - обзор Object Runtime.
- `docs-new/03_platform/03_object_runtime/03_contracts.md` - HTTP- и C#-контракты Runtime API.
- `docs-new/03_platform/02_configuration/artifact_types/object_type.md` - схема `ObjectType`.
- `docs-new/03_platform/12_frontend_platform/00_platform_overview.md` - граница Frontend Platform.
- `plans/033_document_management_content_capability_backlog.md` - пример результата архитектурной проработки платформенной возможности после выявленной потребности модуля.
- `docs/other_tech/dmp-module-object-runtime.md` - исторический и целевой большой дизайн-документ.

## 20. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.2 | 2026-09-03 16:04 +03:00 | Олег Юрьев (@axelprosoft) | 2. Карта возможностей ProductDefinition | docs: document content capability and document management module | [3b55ce83](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/3b55ce83518e455f26704720e6dbf40ba8db6170) |
| 1.1 | 2026-09-03 15:27 +03:00 | Воронкова Вероника (@VeronikaV2121) | Назначение; Что получится в конце; Шаг 9. Подключить модуль к DemoApi; Шаг 9. Подключить модуль к host; 1. Учебный модуль: подключение к DemoApi; Шаг 10. Запустить DemoApi; 2. Реальный прикладной модуль: подключение к production host; Шаг 10. Запустить выбранный host; 1. Запуск DemoApi для учебного модуля; 2. Запуск production host для реального прикладного модуля; 3. Проверка Runtime API; Как понять, что всё сделано правильно; Минимальная карта источников | chore: нормализовать служебный статус quickstart | [586f0b75](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/586f0b75e1d1b3440140b056fb1a3611012da23a) |
| 1.0 | 2026-09-03 11:01 +03:00 | Олег Юрьев (@axelprosoft) | YAML-шапка: review_status, status | Утверждение после merge PR #60: быстрый старт для разработчика прикладного модуля - драфт версия | [PR #60](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/60) |
| 0.1 | 2026-09-03 11:00 +03:00 | Олег Юрьев (@axelprosoft) | Создание документа | быстрый старт для разработчика прикладного модуля - драфт версия | [36a598cd](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/36a598cd6a49700611395940e964d18ddc1a4440) |
