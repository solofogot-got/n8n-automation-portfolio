# Процесс автоматизации мониторинга состояний товаров на сайте Eltex-co.ru

## Описание Workflow

**Eltex State Workflow** — автоматизация мониторинга состояний товаров на сайте eltex-co.ru для дилеров. Получает данные из API Eltex, сравнивает статусы товаров, логирует изменения в Supabase, уведомляет в Telegram при изменениях. Триггеры: Manual и Schedule (09:20).

## Цель

- Отслеживание статусов товаров Eltex.
- Автоматическое обновление БД Supabase.
- Уведомления в Telegram о смене статуса.

## Схема workflow
![Workflow](screen.png)


## Установка

1. Импортируйте `workflow.json` в n8n v2.4.4+.
2. Настройте Credentials.
3. Активируйте Schedule Trigger или Manual Execute.

## Требуемые API Ключи

Supabase
Telegram
Eltex API 


## Настройки Таблиц Supabase

### Таблица `eltex_razdely_tovary`


| Поле | Тип | Описание | Использование |
| :-- | :-- | :-- | :-- |
| `id` | number | PK | Фильтр/UPDATE |
| `product_id` | number | ID товара Eltex | Compare, Filter |
| `current_status` | string | Текущий статус (old/new) | Сравнение old≠new |
| `currient_state_state` | string | Состояние ('new') | Фильтр для обработки |
| `current_status_old`? | string | Старый статус | Set нода |
| `current_status_new` | string | Новый статус из API | Set/UPDATE |


### Таблица `eltex_history`

- **Операции**: Insert (Create row).
- **Поля для Insert**:


| Поле | Значение | Тип | Описание |
| :-- | :-- | :-- | :-- |
| `message` | `eltex-co.ru {{product_name}} {{product_url}} {{current_status_new}}` | string | Лог изменения |
| `created_at` | `now().format('yyyy-MM-dd')` | date | Дата логирования |
| `workflow` | `'eltex_state'` | string | Имя workflow |


**Rls/Политики**: Убедитесь, что Supabase Credentials имеют права SELECT/INSERT/UPDATE. Используйте Row Level Security для prod.[^1]

## Архитектура (Ключевые Узлы)

- **Триггеры**: ManualTrigger, ScheduleTrigger (09:20).
- **Логика**: SplitInBatches → IF (old ≠ new) → HTTP Eltex API → Set states → Supabase ops → Telegram.
- **Sticky Notes**: Указывают на lifecycle, API calls, Supabase updates.
