# Ingestion: ADB cron PHP

---
type: ingestion
name: adb_cron_php
tool: php_cron
schedule: daily 06:30 Europe/Moscow
source:
  - mid_database
  - crm_database
writes_to:
  - adb.orders
  - adb.orders_utm
  - adb.mid_analytics_payment_event_outbox
  - adb.mid_analytics_payment_reconciliation
  - adb.orders_totals
  - adb.order_products
  - adb.customers
  - adb.customer_properties
  - adb.contact_events
  - adb.forms
  - adb.task_properties
  - adb.call_tracking
  - adb.mart_lids
  - adb.ym_abandoned_orders
  - adb.marketplace_products
---

## Назначение

PHP cron script обновляет часть таблиц ADB из внутренних баз MID и CRM.

Для текущего датасета `ya_metrica_users_goals_detailed` важны таблицы:

- [adb.orders](../tables/orders.md)
- [adb.order_products](../tables/order_products.md)
- [adb.orders_totals](../tables/orders_totals.md)

Они используются в virtual SQL датасета для связывания пользователей Яндекс.Метрики с заказами, товарами и суммами заказов.

## Техническая информация

- Код: production PHP cron `adb.php` в CRM
- Класс: `ControllerCronAdb`
- URL запуска: `https://crm.madeindream.com/admin/index.php?route=cron/adb`
- Расписание: ежедневно в 06:30
- Лог: `DIR_LOGS . 'adb.log'`
- Лимит времени выполнения: `2400` секунд
- Memory limit: `2G`

## Подключения

Скрипт подключается к трем базам:

| Логическое имя | Назначение | База |
|---|---|---|
| `mid_db` | интернет-магазин / MID | `madeindream` |
| `crm_db` | CRM | `crm_mid2` |
| `adb` | аналитическая база | `adb` |

Пароли и секреты не документируются.

## Основной порядок выполнения

```text
move_orders
move_orders_utm
move_mid_payment_event_outbox
move_mid_payment_reconciliation
move_orders_totals
move_order_products
move_mp_monitoring_prices
move_customers
move_customer_properties
add_contact_id_to_customer
move_contact_events
move_forms
move_task_properties
move_call_tracking
update_mart_lids
update_ym_abandoned_orders
update_marketplace_product_matches
```

## Таблицы, важные для Яндекс.Метрики

### [adb.orders](../tables/orders.md)

Функция: `move_orders`

Режим обновления:

```text
TRUNCATE -> batch INSERT
```

Источник:

- MID table `order`
- CRM task status из `crm_task` / `crm_task_status`
- CRM executor из `crm_task_grant` / `crm_user`

Ключевые поля для аналитики:

| Поле ADB | Значение |
|---|---|
| `shop_order_id` | ID заказа магазина |
| `ym_uid` | `_ym_uid`, извлеченный из cookie заказа |
| `date_added` | дата создания заказа |
| `date_modified` | дата изменения заказа |
| `order_status` | статус заказа |
| `crm_task_id` | связанная CRM-задача |
| `total` | сумма заказа |
| `crm_task_status` | статус задачи CRM |
| `manager` | исполнитель CRM-задачи |

### [adb.order_products](../tables/order_products.md)

Функция: `move_order_products`

Режим обновления:

```text
TRUNCATE -> batch INSERT
```

Источник:

- MID table `order_product`
- MID product/category mapping для категорий товаров

Использование в датасете `ya_metrica_users_goals_detailed`:

```sql
SELECT
    shop_order_id,
    SUM(quantity) AS product_quantity,
    SUM(price) AS total_price
FROM order_products
GROUP BY shop_order_id
```

### [adb.orders_totals](../tables/orders_totals.md)

Функция: `move_orders_totals`

Режим обновления:

```text
TRUNCATE -> batch INSERT
```

Источник:

- MID table `order_total`
- дополнительная строка `code = 'cost'` рассчитывается как `SUM(product.cost * quantity)` по заказу.

Использование в датасете `ya_metrica_users_goals_detailed`:

```sql
SELECT
    shop_order_id,
    MAX(value) AS total_value
FROM orders_totals
WHERE code = 'total'
GROUP BY shop_order_id
```

## Таблицы payment measurement v2

### [adb.mid_analytics_payment_event_outbox](../tables/mid_analytics_payment_event_outbox.md)

Функция: `move_mid_payment_event_outbox`

Режим обновления:

```text
TRUNCATE -> batch INSERT
```

Источник:

- MID table `analytics_payment_event_outbox`

Назначение:

- перенос `order_paid`, `order_paid_v2`, `purchase_paid`;
- сохранение `order_id`, `payment_id`, `paid_at`, `amount`, `currency`;
- сохранение `client_id`, `yclid`, UTM и `attribution_trust`;
- контроль доставки через `status`, `last_http_code`, `sent_at`.

### [adb.mid_analytics_payment_reconciliation](../tables/mid_analytics_payment_reconciliation.md)

Функция: `move_mid_payment_reconciliation`

Режим обновления:

```text
TRUNCATE -> batch INSERT
```

Источник:

- MID table `analytics_payment_reconciliation`

Назначение:

- контроль read-back и доставки payment-событий;
- флаги `purchase_seen`, `goal_readback_ok`, `classification`, `reconciled_status`;
- сверка суммы/валюты с outbox на уровне payment-v2 datasets.

## Особенности

- Перед основными операциями скрипт отключает `FOREIGN_KEY_CHECKS` и `UNIQUE_CHECKS`, после выполнения включает обратно.
- Большая часть таблиц пересобирается полностью через `TRUNCATE`.
- Этапы выполнения логируются в `adb.log` через `log_stage`.
- Для `orders.ym_uid` значение извлекается из `_ym_uid` внутри `ym_cookie`.
