# ADB Table: mid_analytics_payment_reconciliation

---
type: table
name: mid_analytics_payment_reconciliation
database: ADB
schema: adb
source_system: MID / madeindream
loaded_by:
  - php_cron:adb_cron_php
used_by_datasets:
  - payment_events_v2
  - order_payments_v2
---

## Назначение

ADB-зеркало production-таблицы `madeindream.analytics_payment_reconciliation`.

Используется как слой сверки outbox-событий с доставкой в Метрику/ecommerce:

- HTTP-доставка цели;
- `goal_readback_ok`;
- `purchase_seen`;
- `classification`;
- `reconciled_status`;
- attempts и notes.

## Grain

```text
1 строка = 1 reconciliation_id
```

Также есть уникальность:

```text
event_name + order_id + payment_id
```

## Ключевые поля

| Поле | Описание |
|---|---|
| `event_name` | Тип события: `order_paid`, `order_paid_v2`, `purchase_paid` |
| `order_id` | ID заказа MID |
| `payment_id` | ID платежа |
| `event_id` | ID outbox-события |
| `confirmed_at` | Время подтверждения события |
| `sent_at` | Время отправки |
| `goal_http_code` | HTTP-код доставки |
| `goal_readback_ok` | Флаг read-back подтверждения цели |
| `purchase_seen` | Флаг наличия `purchase_paid` для пары `order_id + payment_id` |
| `amount` | Сумма в reconciliation |
| `currency` | Валюта |
| `classification` | Классификация reconciliation |
| `reconciled_status` | Статус сверки |
| `notes` | Текстовые пояснения |
| `adb_synced_at` | Время переноса строки в ADB |

## Обновление

- Процесс: [ADB cron PHP](../ingestion/adb-cron-php.md)
- Функция: `move_mid_payment_reconciliation`
- Режим: `TRUNCATE -> batch INSERT`
- Источник: `madeindream.analytics_payment_reconciliation`

## Использование

Таблица не создает финансовый факт сама по себе. Она добавляет quality-флаги к payment-v2 витринам:

- есть ли `purchase_paid`;
- совпадает ли сумма/валюта;
- доставлено ли событие;
- есть ли проблемы read-back.
