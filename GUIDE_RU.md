# 📖 Руководство по AAEmu (на русском языке)

> **AAEmu** — серверное программное обеспечение для игры ArcheAge, написанное на платформе .NET C#.  
> Форк оригинального проекта [AAEmu](https://github.com/AAEmu/AAEmu).

---

## 📑 Оглавление

1. [📦 Установка и запуск](#-установка-и-запуск)
   - [Требования](#требования)
   - [Клонирование репозитория](#клонирование-репозитория)
   - [Настройка базы данных](#настройка-базы-данных)
   - [Настройка конфигурации](#настройка-конфигурации)
   - [User Secrets (dotnet user-secrets)](#user-secrets-dotnet-user-secrets)
   - [Запуск через dotnet run](#запуск-через-dotnet-run)
   - [Запуск через Docker](#запуск-через-docker)
   - [Описание портов](#описание-портов)
2. [✅ Список выполненных доработок](#-список-выполненных-доработок)
   - [Игровые системы](#игровые-системы)
   - [Менеджеры](#менеджеры)
   - [Инфраструктура](#инфраструктура)
3. [🔧 Что нужно сделать для полноценной игры](#-что-нужно-сделать-для-полноценной-игры)
4. [💡 Идеи по реализации системы Компаньона](#-идеи-по-реализации-системы-компаньона)
   - [Концепция](#концепция)
   - [Архитектура](#архитектура)
   - [Поведение AI Follow](#поведение-ai-follow)
   - [Интеграция в пати](#интеграция-в-пати)
   - [Роль саппорта](#роль-саппорта)
   - [Сетевые пакеты](#сетевые-пакеты)
   - [База данных](#база-данных)
   - [Примеры кода](#примеры-кода)

---

## 📦 Установка и запуск

### Требования

| Компонент       | Версия / Требование                                   |
| --------------- | ----------------------------------------------------- |
| **ОС**          | Windows 10/11, Ubuntu 20.04+, macOS 12+               |
| **.NET SDK**    | 10.0.0+ (см. [`global.json`](./global.json))          |
| **MySQL**       | 8.0.36+ (или через Docker)                            |
| **Docker**      | 20.10+ (опционально, для запуска через `docker-compose`) |
| **RAM**         | Минимум 4 ГБ (рекомендуется 8 ГБ)                    |

> **Важно:** Версия .NET SDK задана в файле [`global.json`](./global.json):
> ```json
> {
>   "sdk": {
>     "version": "10.0.0",
>     "rollForward": "latestMajor",
>     "allowPrerelease": false
>   }
> }
> ```

---

### Клонирование репозитория

```bash
git clone https://github.com/l3nsa/aaemu.git
cd aaemu
git checkout develop
```

---

### Настройка базы данных

Проект использует две базы данных MySQL:

| База данных    | Назначение                            |
| -------------- | ------------------------------------- |
| `aaemu_login`  | Аккаунты, сессии, список игровых серверов |
| `aaemu_game`   | Все игровые данные (персонажи, предметы, мир) |

**Шаги для создания баз данных вручную:**

```sql
CREATE DATABASE aaemu_login CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE aaemu_game  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**Применение SQL-миграций:**

```bash
mysql -u root -p aaemu_login < ./SQL/aaemu_login.sql
mysql -u root -p aaemu_game  < ./SQL/aaemu_game.sql
```

---

### Настройка конфигурации

#### `AAEmu.Login` — `Config.Local.json`

Создайте файл `AAEmu.Login/Config.Local.json` (он переопределяет `Config.json`):

```json
{
  "SecretKey": "ваш_секретный_ключ",
  "AutoAccount": true,
  "InternalNetwork": {
    "Host": "*",
    "Port": 1234
  },
  "Network": {
    "Host": "*",
    "Port": 1237,
    "NumConnections": 10
  },
  "Connections": {
    "MySQLProvider": {
      "Database": "aaemu_login",
      "Host": "localhost",
      "Password": "ваш_пароль",
      "Port": "3306",
      "User": "root"
    }
  },
  "GameServers": [
    {
      "ID": 1,
      "Name": "AAEmu.Game",
      "Host": "127.0.0.1",
      "Port": 1239
    }
  ]
}
```

#### `AAEmu.Game` — `Config.Local.json`

Создайте файл `AAEmu.Game/Config.Local.json`:

```json
{
  "Id": 1,
  "SecretKey": "ваш_секретный_ключ",
  "Network": {
    "Host": "*",
    "Port": 1239,
    "NumConnections": 10
  },
  "StreamNetwork": {
    "Host": "*",
    "Port": 1250
  },
  "WebApiNetwork": {
    "Host": "*",
    "Port": 1280
  },
  "LoginNetwork": {
    "Host": "127.0.0.1",
    "Port": 1234
  },
  "Connections": {
    "MySQLProvider": {
      "Host": "localhost",
      "Port": "3306",
      "User": "root",
      "Password": "ваш_пароль",
      "Database": "aaemu_game"
    }
  }
}
```

---

### User Secrets (dotnet user-secrets)

Вместо хранения паролей в `Config.Local.json` можно использовать механизм **User Secrets** (.NET):

```bash
# Для AAEmu.Login
cd AAEmu.Login
dotnet user-secrets init
dotnet user-secrets set "Connections:MySQLProvider:Password" "ваш_пароль"
dotnet user-secrets set "SecretKey" "ваш_секретный_ключ"

# Для AAEmu.Game
cd ../AAEmu.Game
dotnet user-secrets init
dotnet user-secrets set "Connections:MySQLProvider:Password" "ваш_пароль"
dotnet user-secrets set "SecretKey" "ваш_секретный_ключ"
```

User Secrets хранятся локально и **не попадают в репозиторий**.

---

### Запуск через `dotnet run`

**Терминал 1 — Login-сервер:**

```bash
cd AAEmu.Login
dotnet run
```

**Терминал 2 — Game-сервер:**

```bash
cd AAEmu.Game
dotnet run
```

> Убедитесь, что Login-сервер запущен **до** запуска Game-сервера.

---

### Запуск через Docker

Проект включает готовый [`docker-compose.yaml`](./docker-compose.yaml).

**1. Создайте `.env` на основе примера:**

```bash
cp .env.example .env
```

**2. Отредактируйте `.env`:**

```env
PROJECT_VERSION_PREFIX=0.3.0.0
PROJECT_VERSION_SUFFIX=alpha

BUILD_CONFIGURATION=Debug
BUILD_FRAMEWORK=net10.0
BUILD_RUNTIME=linux-musl-x64

DB_USER=root
DB_PASSWORD=ваш_пароль
```

**3. Запустите контейнеры:**

```bash
# Стандартный запуск
docker-compose up -d

# С горячей перезагрузкой (для разработки)
docker-compose watch
```

**4. Доступные сервисы после запуска:**

| Сервис     | URL / Порт                  |
| ---------- | --------------------------- |
| MySQL      | `localhost:3306`            |
| Adminer    | http://localhost:8080       |
| Login      | `localhost:1237`            |
| Game       | `localhost:1239`, `1250`    |

**5. Остановка:**

```bash
docker-compose down
```

---

### Описание портов

| Порт   | Протокол | Сервис               | Описание                                              |
| ------ | -------- | -------------------- | ----------------------------------------------------- |
| `1237` | TCP      | Login (внешний)      | Подключение игровых клиентов к Login-серверу          |
| `1234` | TCP      | Login (внутренний)   | Внутренняя связь Login ↔ Game (внутри сети)          |
| `1239` | TCP      | Game                 | Подключение игровых клиентов к Game-серверу           |
| `1250` | TCP      | Stream               | Потоковые данные (карта мира, синхронизация объектов) |
| `1280` | HTTP     | WebAPI               | REST API для администрирования сервера                |
| `3306` | TCP      | MySQL                | База данных                                           |
| `8080` | HTTP     | Adminer (Docker)     | Веб-интерфейс для управления БД                       |

---

## ✅ Список выполненных доработок

### Игровые системы

| Система              | Статус             | Описание                                                     |
| -------------------- | ------------------ | ------------------------------------------------------------ |
| **Персонажи**        | ✅ Реализовано     | Создание, загрузка, сохранение, смерть, воскрешение          |
| **Предметы**         | ✅ Реализовано     | Инвентарь, экипировка, выкидывание, передача, срок действия  |
| **Навыки (Skills)**  | ✅ Реализовано     | Активация скиллов, кулдауны, скиллбуки                       |
| **Баффы (Buffs)**    | ✅ Реализовано     | Наложение, снятие, стаки, длительность, эффекты              |
| **Квесты**           | ✅ Реализовано     | Выдача, прогресс, сдача, награды, триггеры                   |
| **Мир / Зоны**       | ✅ Реализовано     | Загрузка мира, зоны, спавн объектов, движение персонажей     |
| **Крафт**            | ✅ Реализовано     | Рецепты, очереди крафта, компоненты                          |
| **Аукцион**          | ✅ Базово          | Выставление, поиск, покупка лотов                            |
| **Почта**            | ✅ Реализовано     | Отправка, получение, вложения                                |
| **Торговля**         | ✅ Базово          | Сделки между игроками                                        |
| **Порталы**          | ✅ Реализовано     | Телепортация по порталам                                     |
| **Дуэль**            | ✅ Реализовано     | Вызов на дуэль, логика боя                                   |
| **Дома (Housing)**   | ✅ Реализовано     | Строительство, размещение мебели                             |
| **Питомцы (Mates)**  | ✅ Частично        | Базовый менеджер Mate                                        |
| **Рабы (Slaves)**    | ✅ Частично        | Корабли, осадные орудия — базовая реализация                 |
| **Dungeon (Indun)**  | ✅ Базово          | Базовый менеджер подземелий                                  |
| **Налоги**           | ✅ Реализовано     | `TaxationsManager` — расчёт и списание налогов               |
| **Семьи**            | ✅ Реализовано     | Создание семьи, члены семьи                                  |
| **Экспедиции**       | ✅ Реализовано     | Гильдии / экспедиции                                         |

---

### Менеджеры

| Менеджер                 | Статус         | Описание                                              |
| ------------------------ | -------------- | ----------------------------------------------------- |
| `AIManager`              | ✅ Реализован  | Базовый AI для NPC                                    |
| `AiPathsManager`         | ✅ Реализован  | Пути патрулирования NPC                               |
| `TeamManager`            | ✅ Базово      | Система партий                                        |
| `SlaveManager`           | ✅ Частично    | Корабли и осадные машины                              |
| `MateManager`            | ✅ Частично    | Питомцы / компаньоны                                  |
| `HousingManager`         | ✅ Реализован  | Дома и обстановка                                     |
| `AuctionManager`         | ✅ Реализован  | Аукцион                                               |
| `CraftManager`           | ✅ Реализован  | Крафт                                                 |
| `ExpeditionManager`      | ✅ Реализован  | Экспедиции (гильдии)                                  |
| `FamilyManager`          | ✅ Реализован  | Семьи                                                 |
| `MailManager`            | ✅ Реализован  | Почта                                                 |
| `TradeManager`           | ✅ Реализован  | Торговля                                              |
| `PortalManager`          | ✅ Реализован  | Порталы и телепортация                                |
| `DuelManager`            | ✅ Реализован  | Дуэли                                                 |
| `CrimeManager`           | ✅ Реализован  | Система преступлений / злодеяний                      |
| `GimmickManager`         | ✅ Реализован  | Гиммики (интерактивные объекты мира)                  |
| `WorldManager`           | ✅ Реализован  | Управление игровым миром                              |
| `ZoneManager`            | ✅ Реализован  | Зоны мира                                             |
| `SpawnManager`           | ✅ Реализован  | Спавн NPC и объектов                                  |
| `ItemManager`            | ✅ Реализован  | Предметы                                              |
| `SkillManager`           | ✅ Реализован  | Навыки                                                |
| `QuestManager`           | ✅ Реализован  | Квесты                                                |
| `CharacterManager`       | ✅ Реализован  | Персонажи                                             |
| `ExperienceManager`      | ✅ Реализован  | Опыт и уровни                                         |
| `NpcManager`             | ✅ Реализован  | NPC                                                   |
| `DoodadManager`          | ✅ Реализован  | Интерактивные объекты (Doodad)                        |
| `FactionManager`         | ✅ Реализован  | Фракции                                               |
| `IndunManager`           | ✅ Базово      | Подземелья                                            |
| `InstantGameManager`     | ✅ Базово      | Мгновенные игры                                       |
| `ShipyardManager`        | ✅ Реализован  | Верфи (постройка кораблей)                            |
| `TransferManager`        | ✅ Реализован  | Транспортные маршруты                                 |
| `GameScheduleManager`    | ✅ Реализован  | Расписание игровых событий                            |
| `TaxationsManager`       | ✅ Реализован  | Налоги                                                |
| `SaveManager`            | ✅ Реализован  | Сохранение данных                                     |
| `TimeManager`            | ✅ Реализован  | Внутриигровое время                                   |

---

### Инфраструктура

| Компонент          | Статус        | Описание                                                               |
| ------------------ | ------------- | ---------------------------------------------------------------------- |
| **Тесты**          | ✅ 958 тестов | Покрытие ~60-70%; цель — 80%                                           |
| **Docker**         | ✅ Готово     | `docker-compose.yaml` с MySQL, Adminer, Login, Game                    |
| **CI/CD**          | ✅ Настроено  | Travis CI (`.travis.yml`) и GitHub Actions (`.github/`)                |
| **WebAPI**         | ✅ Реализован | REST API на порту 1280 для управления сервером                         |
| **Aspire**         | ✅ Настроен   | `AAEmu.Aspire.AppHost` для оркестрации микросервисов                  |
| **User Secrets**   | ✅ Поддержка  | Безопасное хранение секретов для локальной разработки                  |
| **Логирование**    | ✅ Реализовано| Структурированное логирование через Microsoft.Extensions.Logging       |

---

## 🔧 Что нужно сделать для полноценной игры

Ниже перечислены незавершённые или требующие расширения системы:

### 🤖 AI и NPC

- [ ] **Полноценный AI NPC** — поведение, патрулирование, агрессия, стейт-машина (FSM)
  - Агрессивный NPC: обнаружение игрока в радиусе, атака, возврат на точку
  - Патруль: следование по пути через `AiPathsManager`
  - Реакция на смерть, побег при низком HP
- [ ] **Социальный AI** — NPC реагируют на события (помогают соратникам, убегают от опасности)

### 🐾 Компаньоны и питомцы

- [ ] **Полноценная система Mate/Companion** — см. раздел [💡 Идеи по реализации](#-идеи-по-реализации-системы-компаньона)
  - Follow-логика (следование за игроком)
  - Боевые действия компаньона
  - Интеграция в партию через `TeamManager`

### 👥 Партии и рейды

- [ ] **Расширение `TeamManager`** — поддержка NPC в пати
- [ ] **Рейды** — группы из нескольких пати
- [ ] **Авторазбивка лута** — автоматическое распределение наград

### 🏰 Подземелья

- [ ] **`IndunManager`** — расширить базовую реализацию:
  - Создание инстансов (копий) подземелья для каждой группы
  - Боссы с фазами
  - Таймеры и сброс подземелий
  - Уникальные правила для каждого данжа

### ⚡ Мгновенные игры

- [ ] **`InstantGameManager`** — завершить реализацию:
  - Arenas (PvP-арены)
  - Guild vs Guild
  - Турниры

### 🚢 Морская торговля и трейдпаки

- [ ] **Трейдпаки (Tradepacks)** — полноценная торговля через море:
  - Создание трейдпаков из ресурсов
  - Ценообразование в зависимости от маршрута и спроса
  - Интеграция с экономикой мира

### 🌍 Мировые события

- [ ] **Арх Эйдж / Мировые события** — события по расписанию:
  - Кровавые ворота (Crimson Rift)
  - Война за Галкион (Halcyona War)
  - Вторжение демонов
  - Авторандомные события через `GameScheduleManager`

### ⚔️ PvP-системы

- [ ] **Расширение `CrimeManager`**:
  - Очки злодеяний, суд, тюрьма
  - PvP-зоны и флаги
- [ ] **Crimson Rift** — боевое событие с командами
- [ ] **Halcyona War** — масштабный PvP-конфликт

### ⛵ Морские бои

- [ ] **Расширение `SlaveManager`** — морские бои:
  - Пушки на кораблях
  - Абордаж
  - Потопление и ремонт
  - Различные типы кораблей

### 🏠 Жильё и налоги

- [ ] **Полноценная система налогов** — просрочка уплаты → конфискация дома
- [ ] **Замки и крепости** — осада, захват, управление замком

### 🎒 Инвентарь

- [ ] **Полноценная система инвентаря**:
  - Система сортировки
  - Банковские слоты
  - Инвентарь склада (warehouse)
  - Инвентарь животных / транспорта

### 💰 Экономика

- [ ] **Аукцион** — полноценная реализация с историей цен
- [ ] **Касса** — казна гильдии/экспедиции
- [ ] **Налоги** — полный цикл начисления и взыскания
- [ ] **Рынок** — NPC-торговцы с динамическими ценами

### 🧪 Тесты и качество кода

- [ ] **Увеличить покрытие тестами до 80%** (сейчас ~60-70%):
  - Добавить тесты для `SlaveManager`, `MateManager`, `HousingManager`
  - Расширить тесты для `TeamManager`, `AuctionManager`
  - Интеграционные тесты для базы данных

---

## 💡 Идеи по реализации системы Компаньона

### Концепция

**Компаньон** — это NPC-союзник игрока, который:

- 🏃 Следует за игроком
- ⚔️ Участвует в бою (атакует врагов / поддерживает игрока)
- 💊 Выполняет роль саппорта (хил, баффы, воскрешение)
- 👥 Отображается в составе партии через `TeamManager`
- 📈 Получает опыт вместе с игроком

> В ArcheAge аналогом является система **Mate** (питомец/компаньон), частично реализованная в `MateManager.cs`.

---

### Архитектура

Предлагаемая архитектура основана на существующих паттернах проекта:

```
AAEmu.Game/
├── Core/
│   ├── Managers/
│   │   ├── CompanionManager.cs        ← новый менеджер
│   │   └── ICompanionManager.cs       ← интерфейс менеджера
│   └── Models/
│       └── Game/
│           ├── Units/
│           │   └── Companion.cs        ← модель компаньона
│           └── AI/
│               └── States/
│                   ├── CompanionFollowState.cs
│                   ├── CompanionCombatState.cs
│                   └── CompanionIdleState.cs
├── Network/
│   └── Packets/
│       └── Game/
│           ├── S2C/
│           │   ├── SCCompanionSpawnPacket.cs
│           │   └── SCCompanionFollowPacket.cs
│           └── C2S/
│               └── CSCompanionCommandPacket.cs
└── SQL/
    └── companions_migration.sql
```

---

### Поведение AI Follow

Компаньон реализует **конечный автомат (FSM)** с четырьмя состояниями, используя существующий `AIManager`:

```
┌─────────────────────────────────────────────────────────────┐
│                    AI Компаньона (FSM)                      │
│                                                             │
│  ┌──────────┐    игрок    ┌──────────┐                     │
│  │  Idle    │ ──движется──▶  Follow  │                     │
│  │(стоит)   │◀── стоит ───│(следует) │                     │
│  └──────────┘             └─────┬────┘                     │
│                                 │ враг                      │
│                                 │ рядом                     │
│  ┌──────────┐             ┌─────▼────┐                     │
│  │Teleport  │             │  Combat  │                     │
│  │(>20 метр)│             │  (бой)   │                     │
│  └──────────┘             └──────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

| Состояние     | Условие перехода                         | Действие                                        |
| ------------- | ---------------------------------------- | ----------------------------------------------- |
| `Idle`        | Игрок стоит на месте                     | Стоит рядом с игроком                           |
| `Follow`      | Игрок движется                           | Следует за игроком на дистанции 2-3 метра       |
| `Combat`      | Враг атакует игрока или входит в радиус  | Атакует врага / кастует поддержку на игрока     |
| `Teleport`    | Расстояние до игрока > 20 метров         | Телепортируется к игроку                        |

---

### Интеграция в пати

Для поддержки компаньонов в партии необходимо расширить `TeamManager.cs`:

- **Добавить тип участника** `TeamMemberType.Companion`
- **Компаньон в UI партии**: получать пакет `SCTeamMemberPacket` с данными компаньона
- **Опыт**: компаньон получает опыт в той же доле, что и игрок-владелец
- **Смерть компаньона**: компаньон «отключается» до следующего воскрешения

---

### Роль саппорта

Компаньон выполняет следующие автоматические действия:

| Триггер                        | Действие компаньона                              |
| ------------------------------ | ------------------------------------------------ |
| HP игрока < 50%                | Кастует скилл лечения на игрока                 |
| Вход игрока в бой              | Накладывает бафф на игрока (усиление атаки/защиты) |
| Смерть игрока (если есть скилл)| Воскрешает игрока                               |
| Враг атакует игрока            | Кастует дебафф на врага (замедление / ослабление) |

---

### Сетевые пакеты

Новые пакеты для системы компаньонов (по аналогии с существующими пакетами в `AAEmu.Game/Core/Network`):

| Пакет                        | Направление | Описание                                          |
| ---------------------------- | ----------- | ------------------------------------------------- |
| `SCCompanionSpawnPacket`     | S → C       | Спавн компаньона в мире                           |
| `SCCompanionFollowPacket`    | S → C       | Обновление позиции компаньона (следование)        |
| `SCCompanionDespawnPacket`   | S → C       | Деспавн компаньона                                |
| `CSCompanionCommandPacket`   | C → S       | Команда игрока компаньону (атаковать/стоять/вернуться) |
| `SCCompanionStatusPacket`    | S → C       | Обновление HP/MP компаньона                       |

---

### База данных

**Таблица `companions`** — данные компаньона игрока:

```sql
CREATE TABLE `companions` (
    `id`          BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `owner_id`    INT UNSIGNED NOT NULL COMMENT 'ID персонажа-владельца',
    `template_id` INT UNSIGNED NOT NULL COMMENT 'ID шаблона NPC компаньона',
    `name`        VARCHAR(64)  NOT NULL DEFAULT '',
    `level`       TINYINT UNSIGNED NOT NULL DEFAULT 1,
    `experience`  INT UNSIGNED NOT NULL DEFAULT 0,
    `hp`          INT UNSIGNED NOT NULL DEFAULT 0,
    `mp`          INT UNSIGNED NOT NULL DEFAULT 0,
    `created_at`  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at`  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    KEY `idx_owner_id` (`owner_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Компаньоны игроков';
```

**Таблица `companion_skills`** — навыки компаньона:

```sql
CREATE TABLE `companion_skills` (
    `id`           BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `companion_id` BIGINT UNSIGNED NOT NULL COMMENT 'ID компаньона',
    `skill_id`     INT UNSIGNED NOT NULL COMMENT 'ID навыка',
    `level`        TINYINT UNSIGNED NOT NULL DEFAULT 1,
    PRIMARY KEY (`id`),
    KEY `idx_companion_id` (`companion_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Навыки компаньонов';
```

Файлы миграций размещаются в папке `SQL/`.

---

### Примеры кода

#### `Companion.cs` — модель компаньона

```csharp
namespace AAEmu.Game.Core.Models.Game.Units;

/// <summary>
/// Представляет компаньона (NPC-союзника) игрока.
/// Наследует от Npc и добавляет логику следования и поддержки.
/// </summary>
public class Companion : Npc
{
    /// <summary>ID персонажа-владельца.</summary>
    public uint OwnerId { get; set; }

    /// <summary>Текущее состояние AI компаньона.</summary>
    public CompanionAiState AiState { get; set; } = CompanionAiState.Idle;

    /// <summary>Дистанция следования за игроком (в метрах).</summary>
    public float FollowDistance { get; set; } = 2.5f;

    /// <summary>Дистанция телепортации к игроку (в метрах).</summary>
    public float TeleportDistance { get; set; } = 20f;

    public Companion() : base()
    {
    }
}

/// <summary>Возможные состояния AI компаньона.</summary>
public enum CompanionAiState
{
    Idle,
    Follow,
    Combat,
    Teleport
}
```

#### `ICompanionManager.cs` — интерфейс менеджера

```csharp
namespace AAEmu.Game.Core.Managers;

/// <summary>Интерфейс менеджера компаньонов.</summary>
public interface ICompanionManager
{
    /// <summary>Вызвать компаньона для персонажа.</summary>
    bool SummonCompanion(uint characterId, uint templateId);

    /// <summary>Отозвать компаньона персонажа.</summary>
    void DismissCompanion(uint characterId);

    /// <summary>Получить активного компаньона персонажа.</summary>
    Companion? GetActiveCompanion(uint characterId);

    /// <summary>Обновить AI всех активных компаньонов (вызывается каждый тик).</summary>
    void Tick(TimeSpan delta);
}
```

#### `CompanionManager.cs` — скелет менеджера

```csharp
using NLog;

namespace AAEmu.Game.Core.Managers;

/// <summary>
/// Менеджер компаньонов игроков.
/// По аналогии с MateManager.cs управляет жизненным циклом компаньонов.
/// </summary>
public class CompanionManager : ICompanionManager
{
    private static readonly Logger Logger = LogManager.GetCurrentClassLogger();

    // Активные компаньоны: ключ — ID персонажа-владельца
    private readonly ConcurrentDictionary<uint, Companion> _activeCompanions = new();

    private readonly IWorldManager _worldManager;
    private readonly IAIManager _aiManager;

    public CompanionManager(IWorldManager worldManager, IAIManager aiManager)
    {
        _worldManager = worldManager;
        _aiManager    = aiManager;
    }

    /// <inheritdoc/>
    public bool SummonCompanion(uint characterId, uint templateId)
    {
        var character = _worldManager.GetCharacterById(characterId);
        if (character is null)
        {
            Logger.Warn("SummonCompanion: персонаж {0} не найден", characterId);
            return false;
        }

        if (_activeCompanions.ContainsKey(characterId))
        {
            Logger.Debug("SummonCompanion: у персонажа {0} уже есть компаньон", characterId);
            return false;
        }

        var companion = new Companion
        {
            OwnerId    = characterId,
            TemplateId = templateId,
            Position   = character.Position.Clone(),
            AiState    = CompanionAiState.Follow
        };

        _activeCompanions[characterId] = companion;

        // Спавним в мире
        _worldManager.SpawnUnit(companion);

        Logger.Info("Компаньон {0} вызван для персонажа {1}", templateId, characterId);
        return true;
    }

    /// <inheritdoc/>
    public void DismissCompanion(uint characterId)
    {
        if (_activeCompanions.TryRemove(characterId, out var companion))
        {
            _worldManager.DespawnUnit(companion);
            Logger.Info("Компаньон персонажа {0} отозван", characterId);
        }
    }

    /// <inheritdoc/>
    public Companion? GetActiveCompanion(uint characterId)
        => _activeCompanions.GetValueOrDefault(characterId);

    /// <inheritdoc/>
    public void Tick(TimeSpan delta)
    {
        foreach (var (ownerId, companion) in _activeCompanions)
        {
            var owner = _worldManager.GetCharacterById(ownerId);
            if (owner is null)
                continue;

            UpdateCompanionAi(companion, owner, delta);
        }
    }

    private void UpdateCompanionAi(Companion companion, Character owner, TimeSpan delta)
    {
        var distanceToOwner = companion.Position.DistanceTo(owner.Position);

        // Переход в Teleport если слишком далеко
        if (distanceToOwner > companion.TeleportDistance)
        {
            companion.AiState = CompanionAiState.Teleport;
            companion.Position = owner.Position.Clone();
            return;
        }

        companion.AiState = distanceToOwner > companion.FollowDistance
            ? CompanionAiState.Follow
            : CompanionAiState.Idle;

        // Проверка угрозы (враги рядом)
        if (_aiManager.HasNearbyEnemies(companion.Position, 10f))
            companion.AiState = CompanionAiState.Combat;
    }
}
```

#### Состояние AI `CompanionFollowState.cs`

```csharp
namespace AAEmu.Game.Core.Models.Game.AI.States;

/// <summary>
/// Состояние AI компаньона: следование за игроком.
/// Использует существующую инфраструктуру AIManager и AiPathsManager.
/// </summary>
public class CompanionFollowState : IAiState
{
    private readonly Companion _companion;
    private readonly float _followDistance;

    public CompanionFollowState(Companion companion, float followDistance = 2.5f)
    {
        _companion     = companion;
        _followDistance = followDistance;
    }

    public void Enter() { /* инициализация */ }

    public void Tick(TimeSpan delta)
    {
        var owner = // TODO: получить владельца через _worldManager.GetCharacterById(_companion.OwnerId)
            (Character?)null;
        if (owner is null) return;

        var dist = _companion.Position.DistanceTo(owner.Position);
        if (dist <= _followDistance) return;

        // Вычисляем направление и двигаемся
        var direction = (owner.Position - _companion.Position).Normalized();
        _companion.Position += direction * (float)(delta.TotalSeconds * _companion.MoveSpeed);
    }

    public void Exit() { /* очистка */ }
}
```

---

> 💬 **Вопросы и предложения** приветствуются через Issues и Pull Requests!  
> 📄 Лицензия: см. [LICENSE](./LICENSE), [LICENSE.GPL](./LICENSE.GPL), [LICENSE.MIT](./LICENSE.MIT)
