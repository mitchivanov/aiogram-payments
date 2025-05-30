# Бот для управления подписками на каналы

Этот бот позволяет пользователям покупать подписки разных типов (Базовая и Премиум) и получать доступ к соответствующим каналам через защищенные персональные пригласительные ссылки.

## Установка

1. Установите зависимости:
```bash
pip install aiogram python-dotenv sqlalchemy
```

2. Настройте файл `.env`:
```
TELEGRAM_BOT_TOKEN=your_bot_token_here
PAYMENT_TOKEN=your_payment_token
PAYMENT_TEST_MODE=True  # или False для боевого режима

# Каналы для подписок (ID каналов)
BASIC_CHANNEL_ID=-100123456789  # Замените на ID вашего канала для базовой подписки
PREMIUM_CHANNEL_ID=-100987654321  # Замените на ID вашего канала для премиум подписки
```

## Настройка каналов

1. Создайте два канала в Telegram: один для базовой подписки, один для премиум
2. Добавьте вашего бота в каналы как администратора с правами:
   - Добавление участников
   - Создание приглашений
   - **Управление приглашениями** (необходимо для обработки запросов на вступление)
   - **Удаление сообщений** (рекомендуется)

3. Получите ID каналов. Для этого:
   - Перешлите сообщение из канала боту @username_to_id_bot или подобному
   - Запишите полученные ID в файл .env как BASIC_CHANNEL_ID и PREMIUM_CHANNEL_ID

## Механизм защиты ссылок-приглашений

В боте реализован механизм защиты ссылок-приглашений от использования неправильными пользователями:

1. При активации подписки создается персональная ссылка, которая:
   - Требует одобрения для вступления в канал
   - Содержит ID пользователя в названии для отслеживания
   - Действительна в течение 7 дней (можно изменить)

2. Когда пользователь переходит по ссылке и запрашивает вступление:
   - Бот проверяет ID пользователя, сравнивая его с ожидаемым ID для данной ссылки
   - Если ID совпадает, запрос автоматически одобряется
   - Если ID не совпадает, запрос отклоняется с уведомлением

Это гарантирует, что доступ к каналу получит только тот пользователь, который купил подписку.

## Настройка платежей

1. Для работы оплаты через Telegram Payments API необходимо:
   - Для тестирования используйте @BotFather и команду /mybots -> Выберите бота -> Payments -> ЮKassa Test
   - Для боевого режима подключите платежную систему через BotFather

2. Полученный токен платежной системы укажите в .env как PAYMENT_TOKEN
   - Для тестирования вы получите токен вида: 381764678:TEST:12345

## Запуск бота

```bash
python main.py
```

## Основные функции

- Выбор типа подписки (Базовая/Премиум)
- Выбор длительности подписки (30/90/180 дней)
- Оплата через Telegram Payments
- Автоматическая генерация защищенных пригласительных ссылок
- Защита от использования ссылок неправильными пользователями
- Автоматическое одобрение запросов на вступление для правильных пользователей
- Отслеживание статуса подписки
- Возможность продления/отмены подписки

## Структура кода

- `main.py` - основной файл бота с обработчиками команд
- `database.py` - описание схемы базы данных SQLite
- `subscription_service.py` - сервис работы с подписками
- `subscription_manager.py` - менеджер подписок для работы с БД
- `keyboards.py` - клавиатуры для взаимодействия с ботом

## Перед запуском в production

1. Убедитесь, что бот настроен как администратор в обоих каналах с необходимыми правами
2. Установите `PAYMENT_TEST_MODE=False` в файле .env
3. Замените тестовый платежный токен на боевой
4. Проверьте правильность ID каналов
5. Проверьте механизм запросов на вступление - они должны автоматически одобряться для правильных пользователей

## Тестирование

Для проверки корректности работы системы реализованы асинхронные тесты на базе pytest и pytest-asyncio.

### Запуск тестов

1. Установите dev-зависимости:
   ```bash
   pip install -r requirements.txt
   ```
2. Запустите тесты:
   ```bash
   pytest
   ```

### Структура тестов

- `tests/test_subscription_service.py` — тесты для SubscriptionService (создание и получение пользователя)
- `tests/test_subscription_manager.py` — тесты для SubscriptionManager (создание, продление подписки)
- `tests/test_subscription_service_retry.py` — тесты на ретраи (устойчивость к временным ошибкам)

Тесты используют отдельную in-memory БД или тестовую БД PostgreSQL. Для интеграционного тестирования с docker-compose используйте отдельный compose-файл с тестовой БД.

Все тесты асинхронные и покрывают основные сценарии работы сервиса подписок. 