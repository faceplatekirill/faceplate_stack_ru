# Развёртывание Faceplate + OpenClaw (Docker Compose)

## Структура проекта

В итоге, директория должна выглядеть следующим образом:

```
faceplate_stack/
├── docker-compose.yml
├── .env
├── faceplate/
│   ├── DB/
│   └── logs/
└── openclaw/
    ├── config/
    └── workspace/
```
Приведём последовательность шагов:

---

## Шаг 1. Клонирование OpenClaw

Создайте директорию и клонируйте репозиторий:

```bash
mkdir -p ~/Documents/faceplate_stack/openclaw
cd ~/Documents/faceplate_stack/openclaw
```

Клонируйте конфиг-файлы и скиллы Openclaw:

```bash
git clone https://github.com/faceplatekirill/fp_openclaw.git .
```

После клонирования в папке `openclaw` появятся все необходимые файлы (`config`, `workspace` и др.).

---

## Шаг 2. Подготовка директорий

Создайте структуру директорий для SCADA:

```bash
mkdir -p faceplate/DB
mkdir -p faceplate/logs
```

**Примечание:**
> - `faceplate/DB` — хранилище базы данных SCADA
> - `faceplate/logs` — журналы приложения
> - `openclaw/config` — файлы конфигурации OpenClaw
> - `openclaw/workspace` — рабочая среда (агенты, память, инструменты)

---

## Шаг 3. Docker Compose

Файл `docker-compose.yml` уже настроен для запуска двух сервисов:

### Faceplate

- Основное приложение (бэкенд + UI + БД)
- Открытые порты:
  - `9000`
  - `8000`
  - `7000`
- Постоянные тома:
  - База данных → `faceplate/DB`
  - Журналы логов → `faceplate/logs`

### OpenClaw

- Агент для SCADA
- Открытый порт:
  - `18789`
- Постоянные тома:
  - Конфигурация → `openclaw/config`
  - Рабочая среда → `openclaw/workspace`

---

Файл `.env` содержит дополнительные параметры для Docker Compose.

## Шаг 4. Запуск стека

**Запуск всего стека в фоновом режиме:**

```bash
docker compose up -d
```

**Просмотр журналов:**

```bash
docker compose logs -f
```

**Остановка стека:**

```bash
docker compose down
```

---

## Проверка сервисов

После запуска сервисы доступны по следующим адресам:

| Сервис             | URL                                                              |
|--------------------|------------------------------------------------------------------|
| Faceplate Studio   | [http://localhost:9000/fp/studio](http://localhost:9000/fp/studio) |
| Faceplate Runtime  | [http://localhost:9000/fp/runtime](http://localhost:9000/fp/runtime) |
| OpenClaw           | [http://localhost:18789](http://localhost:18789)                 |

---

**Учётные данные Faceplate по умолчанию**

Логин пользователя:
```bash
system
```

Пароль:
```bash
111111
```

---

## Рабочая среда OpenClaw

Директория `openclaw/workspace` содержит состояние системы:

| Путь / Файл        | Описание                                      |
|--------------------|-----------------------------------------------|
| `AGENTS.md`        | Определения агентов                           |
| `MEMORY.md`        | Конфигурация памяти                           |
| `memory/`          | Постоянное хранилище памяти                   |
| `TOOLS.md`         | Определения инструментов                      |
| `skills/`          | Пользовательские навыки                       |
| `extensions/`      | Расширения и плагины                          |
| `PROJECT_KB`       | База знаний проекта                           |
| `KNOWLEDGE_BASE`   | Общая база знаний                             |

> **Не удаляйте эту папку при перезапуске** — в ней хранится постоянное состояние агента.

## Настройка OpenClaw

Процесс настройки OpenClaw совместно с Faceplate описан здесь: https://github.com/faceplatekirill/fp_openclaw_configs

---

## Пересборка и перезапуск

Если образы или конфигурации изменились:

```bash
docker compose down
docker compose up -d --build
```

---

## Устранение неполадок

**1. Порт уже занят**

```bash
sudo lsof -i :9000
```

**2. Отказано в доступе к директориям**

```bash
sudo chown -R $USER:$USER .
```

**3. Контейнер не запускается**

```bash
docker compose logs faceplate
docker compose logs openclaw
```

**4. Интерфейс OpenClaw требует Gateway Token**

Вставьте токен из файла `openclaw/config/openclaw.json` в поле **Gateway Token** интерфейса OpenClaw.

**5. Интерфейс OpenClaw требует утвердить сессию**

Если интерфейс OpenClaw требует утвердить сессию, необходимо сделать следующее:

```bash
docker exec -it faceplate_openclaw_platform /bin/bash
```

Вы окажетесь внутри контейнера. Выполните команду:

```bash
openclaw devices list
```

Вы получите список ожидающих запросов. Скопируйте из списка Request ID (далее по тексту — Request_ID)  и выполните:

```bash
openclaw devices approve Request_ID
```

Теперь вы можете использовать интерфейс OpenClaw в браузере.



---

## Итог

Этот стек запускает два взаимосвязанных сервиса:

- **`faceplate`** — бэкенд, UI и база данных
- **`openclaw`** — система ИИ-агентов

---

## Следующий шаг

У вас развёрнут пустой проект Faceplate. Заполнить его тестовыми SCADA-данными можно по инструкции:
https://github.com/faceplatekirill/fp_demo_project_light_ru

## Документация Faceplate

Документацию и информацию по Faceplate можно найти здесь:
https://github.com/faceplate-docs/faceplate/tree/dev/docs/ru
