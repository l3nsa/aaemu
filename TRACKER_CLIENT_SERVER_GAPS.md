# 📋 Трекер: Реализация недостающих систем клиента ArcheAge 1.2

> Этот файл служит для отслеживания прогресса реализации недостающих игровых систем,
> которые ожидает клиент ArcheAge 1.2 (`r208022`), но отсутствуют или частично реализованы на сервере AAEmu.
>
> Подробный анализ: [`CLIENT_SERVER_GAP_ANALYSIS.md`](./CLIENT_SERVER_GAP_ANALYSIS.md)

---

## 🔴 Критический приоритет

### Обработка пакетов торговли (8 пакетов)
- [ ] `CSCanStartTradePacket` — проверка возможности торговли
- [ ] `CSCannotStartTradePacket` — отказ от торговли
- [ ] `CSStartTradePacket` — начало сделки
- [ ] `CSCancelTradePacket` — отмена сделки
- [ ] `CSPutupTradeItemPacket` — добавление предмета
- [ ] `CSPutupTradeMoneyPacket` — добавление денег
- [ ] `CSTakedownTradeItemPacket` — удаление предмета
- [ ] `CSTradeLockPacket` / `CSTradeOkPacket` — подтверждение

### Обработка пакетов аукциона (6 пакетов)
- [ ] `CSAuctionPostPacket` — выставление лота
- [ ] `CSAuctionSearchPacket` — поиск на аукционе
- [ ] `CSBidAuctionPacket` — ставка на лот
- [ ] `CSCancelAuctionPacket` — отмена лота
- [ ] `CSAuctionLowestPricePacket` — минимальная цена
- [ ] `CSAuctionMyBidListPacket` — список моих ставок

### Обработка пакетов партий и рейдов (15 пакетов)
- [ ] `CSInviteToTeamPacket` — приглашение в группу
- [ ] `CSInviteAreaToTeamPacket` — приглашение из области
- [ ] `CSReplyToJoinTeamPacket` — ответ на приглашение
- [ ] `CSKickTeamMemberPacket` — исключение из группы
- [ ] `CSLeaveTeamPacket` — выход из группы
- [ ] `CSDismissTeamPacket` — роспуск группы
- [ ] `CSMakeTeamOwnerPacket` — передача лидерства
- [ ] `CSMoveTeamMemberPacket` — перемещение по слотам
- [ ] `CSSetTeamMemberRolePacket` — назначение роли
- [ ] `CSSetTeamOfficerPacket` — назначение офицера
- [ ] `CSConvertToRaidTeamPacket` — конвертация в рейд
- [ ] `CSChangeLootingRulePacket` — правила лута
- [ ] `CSAskRiskyTeamActionPacket` — опасные действия
- [ ] `CSRollDicePacket` — бросок кубиков
- [ ] `CSLootDicePacket` — лут через кубики

### Обработка пакетов гильдий/экспедиций (12 пакетов)
- [ ] `CSCreateExpeditionPacket` — создание гильдии
- [ ] `CSDismissExpeditionPacket` — роспуск гильдии
- [ ] `CSInviteToExpeditionPacket` — приглашение
- [ ] `CSKickFromExpeditionPacket` — исключение
- [ ] `CSLeaveExpeditionPacket` — выход из гильдии
- [ ] `CSChangeExpeditionMemberRolePacket` — смена роли
- [ ] `CSChangeExpeditionOwnerPacket` — передача лидерства
- [ ] `CSChangeExpeditionRolePolicyPacket` — политика ролей
- [ ] `CSChangeExpeditionSponsorPacket` — спонсорство
- [ ] `CSRenameExpeditionPacket` — переименование
- [ ] `CSReplyExpeditionInvitationPacket` — ответ на приглашение

### Доработка эффектов навыков (~110 эффектов с TODO)
- [ ] Базовые боевые эффекты (KnockBack, Projectile и др.)
- [ ] Эффекты зачарования (GradeEnchant, PhysicalEnchant и др.)
- [ ] Эффекты транспорта (SpawnSlave, DestroyAndSpawnSlave)
- [ ] Эффекты телепортации (TeleportToSiegeHq, SavePortal)
- [ ] Эффекты крафта (ItemRefurbishment, ItemSmelting, ItemSocketing)
- [ ] Эффекты трансформации (GenderTransfer, Skinize)
- [ ] Гильдейские эффекты (ExpeditionSummon, AddExpeditionExp)

### Система питомцев/маунтов (9 пакетов)
- [ ] `CSChangeMateEquipmentPacket` — экипировка маунта
- [ ] `CSChangeMateNamePacket` — переименование
- [ ] `CSChangeMateTargetPacket` — смена цели
- [ ] `CSChangeMateUserStatePacket` — состояние управления
- [ ] `CSMountMatePacket` — посадка
- [ ] `CSUnMountMatePacket` — спешивание
- [ ] `CSRemoveMatePacket` — удаление
- [ ] `CSRepairPetItemsPacket` — ремонт экипировки
- [ ] AI для следования за игроком и боя

---

## 🟠 Высокий приоритет

### Система правосудия (9+ пакетов)
- [ ] `CSJuryEndTestimonyPacket` — завершение показаний
- [ ] `CSJurySummonedPacket` — вызов присяжных
- [ ] `CSJuryVerdictPacket` — вынесение вердикта
- [ ] `CSReplyImprisonOrTrialPacket` — выбор: тюрьма или суд
- [ ] `CSReplyInviteJuryPacket` — ответ на приглашение в жюри
- [ ] `CSJoinTrialAudiencePacket` — аудитория суда
- [ ] `CSLeaveTrialAudiencePacket` — выход из аудитории
- [ ] `CSSkipFinalStatementPacket` — пропуск последнего слова
- [ ] `CSCancelTrialPacket` — отмена суда
- [ ] Менеджер суда и тюрьмы

### Жильё — расширенные функции (12 пакетов + 25 TODO)
- [ ] `CSBuyHousePacket` — покупка
- [ ] `CSCreateHousePacket` — постройка
- [ ] `CSDecorateHousePacket` — декорирование
- [ ] `CSChangeHouseNamePacket` — переименование
- [ ] `CSChangeHousePermissionPacket` — права доступа
- [ ] `CSConstructHouseTaxPacket` / `CSRequestHouseTaxPacket` — налоги
- [ ] `CSSellHousePacket` / `CSSellHouseCancelPacket` — продажа
- [ ] Конфискация при неуплате налогов
- [ ] Система замков и крепостей

### Корабли и рабы (9 пакетов)
- [ ] `CSSpawnSlavePacket` — вызов
- [ ] `CSDespawnSlavePacket` — деспавн
- [ ] `CSDestroySlavePacket` — уничтожение
- [ ] `CSChangeSlaveEquipmentPacket` — экипировка
- [ ] `CSChangeSlaveNamePacket` — переименование
- [ ] `CSBindSlavePacket` — привязка
- [ ] `CSRepairSlaveItemsPacket` — ремонт
- [ ] Кастомизация кораблей
- [ ] Морские бои (пушки, абордаж, ремонт)

### Почтовая система (3 незавершённых пакета)
- [ ] `CSDeleteMailPacket` — удаление
- [ ] `CSListMailContinuePacket` — продолжение списка
- [ ] `CSReturnMailPacket` — возврат письма

### Торговля трейдпаками
- [ ] Создание трейдпаков из ресурсов
- [ ] Маршруты и ценообразование
- [ ] Интеграция с экономикой

### Семьи (6 пакетов)
- [ ] `CSFamilyInviteMemberPacket` — приглашение
- [ ] `CSFamilyKickPacket` — исключение
- [ ] `CSFamilyLeavePacket` — выход
- [ ] `CSFamilyChangeOwnerPacket` — смена главы
- [ ] `CSFamilyChangeTitlePacket` — смена титула
- [ ] `CSFamilyReplyInvitationPacket` — ответ

---

## 🟡 Средний приоритет

### Арены / PvP-турниры (5 пакетов)
- [ ] `CSApplyToInstantGamePacket`
- [ ] `CSJoinInstantGamePacket`
- [ ] `CSLeaveInstantGamePacket`
- [ ] `CSCancelInstantGamePacket`
- [ ] `CSEnteredInstantGameWorldPacket`

### Салон красоты (3 пакета)
- [ ] `CSBeautyshopDataPacket`
- [ ] `CSEnterBeautySalonPacket`
- [ ] `CSExitBeautySalonPacket`

### Редактирование персонажа
- [ ] `CSEditCharacterPacket` — помечен как `NOT IMPLEMENTED`

### Осады и замки
- [ ] Полный цикл осады
- [ ] Управление территориями
- [ ] Налоги доминионов
- [ ] PvP-флаги

### Мировые события
- [ ] Crimson Rift
- [ ] Halcyona War
- [ ] Вторжение демонов
- [ ] Интеграция с `GameScheduleManager`

### Фракции и миграция (5 пакетов)
- [ ] `CSFactionDeclareHostilePacket`
- [ ] `CSFactionImmigrateToOriginPacket`
- [ ] `CSFactionImmigrationInvitePacket`
- [ ] `CSFactionImmigrationInviteReplyPacket`
- [ ] `CSFactionKickToOriginPacket`

### Крафт — расширенная система
- [ ] Зачарование
- [ ] Сокеты (луна-камни)
- [ ] Перековка
- [ ] Улучшение предметов

---

## 🟢 Низкий приоритет

### Музыкальная система (4 пакета)
- [ ] `CSEndMusicPacket`
- [ ] `CSSaveUserMusicNotes`
- [ ] `CSSendUserMusicPacket`
- [ ] `CSRequestMusicNotesPacket`

### Башенная защита
- [ ] Реализация мини-игры Tower Defense

### Погода и окружающая среда
- [ ] `WeatherManager` — менеджер погоды
- [ ] Цикл погоды по зонам
- [ ] Влияние на геймплей

### Расширенный AI NPC
- [ ] Социальное поведение
- [ ] Патрулирование по маршрутам
- [ ] Реакция на события

### Безопасность аккаунта
- [ ] `CSSetupSecondPassword`
- [ ] `CSRequestSecondPasswordKeyTablesPacket`

### Тесты
- [ ] Увеличить покрытие тестами до 80%
- [ ] Тесты для `SlaveManager`
- [ ] Тесты для `MateManager`
- [ ] Тесты для `HousingManager`
- [ ] Интеграционные тесты для БД

---

## 📊 Прогресс

| Категория | Прогресс |
|-----------|----------|
| 🔴 Критический | 0 / 7 задач |
| 🟠 Высокий | 0 / 6 задач |
| 🟡 Средний | 0 / 7 задач |
| 🟢 Низкий | 0 / 5 задач |
| **Итого** | **0 / 25 задач** |

---

> Обновляйте этот файл по мере завершения задач.
> Подробности по каждому пункту — в [`CLIENT_SERVER_GAP_ANALYSIS.md`](./CLIENT_SERVER_GAP_ANALYSIS.md).
