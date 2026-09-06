---
id: DOC-07-99-02
title: 'Локальное развертывание DMP Platform через Docker Compose'
type: operation
status: approved
version: '1.0'
owner: '@A-Zhigalin'
reviewers:
- '@axelprosoft'
- '@A-Zhigalin'
scope: operations
module: operations
holder: '@axelprosoft'
created_at: 2026-08-27 11:08
created_by: '@A-Zhigalin'
updated_at: 2026-08-27 11:39
last_modified_by: '@A-Zhigalin'
last_reviewed: null
review_status: approved
supersedes: []
source: upstream
---

# Локальное развертывание DMP Platform через Docker Compose

## 1. Назначение документа

Документ описывает настройку, запуск, проверку и остановку локального экземпляра DMP Platform через Docker Compose.

Compose запускает:

- SQL Server 2022;
- production host `DMP.Platform.Api` без демонстрационного backend-модуля;
- frontend-приложения Studio, Admin и Runtime;
- входной Nginx, который публикует приложения и API на одном HTTP-порту.

## 2. Область действия

Инструкция предназначена для локальной разработки и первичной проверки Docker-образов. Конфигурация может использоваться как основа тестового стенда после замены паролей и секретов.

Инструкция не описывает CI/CD, HTTPS, резервное копирование и промышленную эксплуатацию SQL Server.

## 3. Предварительные требования

На рабочей станции должны быть установлены:

- Docker Desktop с поддержкой Linux containers;
- Docker Compose v2;
- Git checkout репозитория DMP Platform Foundation.

Docker daemon должен иметь доступ к Docker Hub, `mcr.microsoft.com` и `registry.npmjs.org`. При первом запуске загружаются базовые образы и пакеты, поэтому операция может занять несколько минут.

Все команды выполняются из корня репозитория, в котором находится `compose.yaml`.

## 4. Настройка окружения

### 4.1 Настройки по умолчанию

Compose поддерживает следующие переменные:

| Переменная | Значение по умолчанию | Назначение |
|---|---|---|
| `DMP_HTTP_PORT` | `8090` | HTTP-порт входного Nginx на локальной машине. |
| `MSSQL_HOST_PORT` | `1443` | Порт SQL Server, опубликованный на локальной машине. |
| `MSSQL_SA_PASSWORD` | `DmpLocal_SA_2026!` | Пароль системной учетной записи `sa`. |
| `DMP_ADMIN_PASSWORD` | `ChangeMe123!` | Начальный пароль администратора платформы. |
| `DMP_PREVIEW_SIGNING_SECRET` | `dmp-local-preview-signing-secret-change-me` | Секрет подписи preview-запросов Report Service. |
| `DMP_API_BASE_URL` | пустая строка | Публичный базовый URL API для frontend. |

Пустой `DMP_API_BASE_URL` включает рекомендуемый режим same-origin: браузер обращается к `/api`, а входной Nginx перенаправляет запросы в backend-контейнер. Значение внедряется при запуске frontend-контейнеров; пересборка образа для его изменения не требуется.

### 4.2 Локальный файл `.env`

Для переопределения настроек необходимо создать `.env` рядом с `compose.yaml`:

```dotenv
DMP_HTTP_PORT=8090
MSSQL_HOST_PORT=1443
MSSQL_SA_PASSWORD=DmpLocal_SA_2026!
DMP_ADMIN_PASSWORD=ChangeMe123!
DMP_PREVIEW_SIGNING_SECRET=dmp-local-preview-signing-secret-change-me
DMP_API_BASE_URL=
```

Пароли и секреты по умолчанию допустимы только для локальной разработки. Для общего тестового стенда должны использоваться уникальные значения, переданные защищенным способом.

Если порт `8090` занят или зарезервирован операционной системой, следует выбрать свободный порт, например:

```dotenv
DMP_HTTP_PORT=8091
```

Контейнер SQL Server по умолчанию доступен на локальном порту `1443`, чтобы не конфликтовать с установленным на машине SQL Server, обычно использующим `1433`. При необходимости следует изменить только внешний порт:

```dotenv
MSSQL_HOST_PORT=14330
```

Внутреннее подключение backend к SQL Server при этом остается на `sqlserver:1433`.

Для подключения к контейнерной базе через SQL Server Management Studio при настройках по умолчанию используются:

```text
Server name: localhost,1443
Authentication: SQL Server Authentication
Login: sa
Password: значение MSSQL_SA_PASSWORD
Trust server certificate: enabled
Database: DMP.Platform
```

## 5. Первый запуск

Собрать образы и запустить контейнеры в фоновом режиме:

```powershell
docker compose up --build -d
```

Проверить состояние сервисов:

```powershell
docker compose ps
```

SQL Server сначала переходит в состояние `health: starting`. После готовности SQL Server запускается backend, который создает базу `DMP.Platform`, применяет миграции и выполняет начальную инициализацию платформы.

Проверить процесс инициализации backend:

```powershell
docker compose logs -f backend
```

Выход из просмотра логов выполняется сочетанием `Ctrl+C`; контейнеры при этом продолжают работать.

## 6. Адреса приложений

При стандартном `DMP_HTTP_PORT=8090` используются следующие адреса:

| Компонент | Адрес |
|---|---|
| Studio | <http://localhost:8090/studio/> |
| Admin | <http://localhost:8090/admin/> |
| Runtime | <http://localhost:8090/runtime/> |
| Проверка входного Nginx | <http://localhost:8090/health> |
| Проверка backend | <http://localhost:8090/api-health> |

Если задан другой `DMP_HTTP_PORT`, число `8090` в адресах заменяется выбранным значением.

Корневой адрес <http://localhost:8090/> перенаправляет пользователя в Studio.

## 7. Начальная учетная запись

После первичной инициализации используется учетная запись:

```text
Логин:  platform-root\global.admin
Пароль: значение DMP_ADMIN_PASSWORD, по умолчанию ChangeMe123!
```

Пароль из `.env` используется при создании начальной учетной записи. Изменение переменной после инициализации существующей базы не меняет уже сохраненный пароль.

## 8. Повторный запуск и пересборка

Запустить ранее созданные контейнеры:

```powershell
docker compose start
```

Применить изменения исходного кода и пересобрать образы:

```powershell
docker compose up --build -d
```

Перезапустить сервисы без пересборки:

```powershell
docker compose restart
```

Изменение runtime-переменных frontend требует пересоздания контейнеров, но не пересборки образов:

```powershell
docker compose up -d --force-recreate studio admin runtime gateway
```

## 9. Остановка и удаление

Остановить и удалить контейнеры и сеть Compose, сохранив данные SQL Server:

```powershell
docker compose down
```

Данные базы хранятся в named volume `dmp-platform_sqlserver-data` и будут использованы при следующем запуске.

Полностью удалить локальную базу и начать с чистого состояния:

```powershell
docker compose down --volumes
```

Команда с `--volumes` безвозвратно удаляет данные SQL Server из этого Compose-проекта. Перед ее выполнением необходимо убедиться, что локальные данные больше не нужны.

## 10. Диагностика

### 10.1 Состояние и логи

```powershell
docker compose ps
docker compose logs --tail 200
docker compose logs --tail 200 sqlserver
docker compose logs --tail 200 backend
docker compose logs --tail 200 gateway
```

### 10.2 Порт недоступен

Ошибка `ports are not available` означает, что опубликованный порт занят или зарезервирован. Необходимо изменить `DMP_HTTP_PORT` или `MSSQL_HOST_PORT` в `.env`, затем повторить:

```powershell
docker compose up -d
```

### 10.3 Ошибки загрузки образов или npm-пакетов

Ошибки `EOF`, `TLS handshake timeout`, `EAI_AGAIN` и `failed to fetch anonymous token` обычно означают временную недоступность registry или DNS из Docker daemon. После восстановления сети следует повторить:

```powershell
docker compose up --build -d
```

Docker повторно использует уже загруженные слои.

### 10.4 Backend не запускается

Проверить готовность SQL Server и сообщения backend:

```powershell
docker compose ps sqlserver backend
docker compose logs --tail 200 sqlserver
docker compose logs --tail 200 backend
```

Если пароль `MSSQL_SA_PASSWORD` был изменен после создания SQL volume, новое значение может не совпадать с паролем существующего экземпляра SQL Server. Необходимо вернуть исходный пароль либо, если данные не нужны, удалить volume по правилам раздела 9 и создать базу заново.

## 11. Связанные документы

- [Стратегия ведения документации](../00_governance/00_documentation_strategy.md).
- [Корневой README проекта](../../README.md).
- [Docker Compose](../../compose.yaml).
- [Dockerfile backend](../../Dockerfile.backend).
- [Dockerfile frontend](../../src/Frontend/Dockerfile).

## 12. История изменений

| Версия | Дата и время | Автор | Раздел | Изменение | Коммит/PR |
|---|---|---|---|---|---|
| 1.0 | 2026-08-27 11:39 +04:00 | A-Zhigalin (@A-Zhigalin) | YAML-шапка: review_status, status | Утверждение после merge PR #49: Feature/docker | [PR #49](https://github.com/axelprosoft/DMP.PlatformFoundation/pull/49) |
| 0.1 | 2026-08-27 11:37 +04:00 | A-Zhigalin (@A-Zhigalin) | Создание документа | Настройка работы в виде контейнера docker | [420397ac](https://github.com/axelprosoft/DMP.PlatformFoundation/commit/420397ac94ef2407ee41971c5a025e5617cd21c2) |
