# Ingestion: Payment measurement v2

---
type: ingestion
name: payment_measurement_v2
tool: php_cron
schedule: daily 06:30 Europe/Moscow
source:
  - madeindream.analytics_payment_event_outbox
  - madeindream.analytics_payment_reconciliation
writes_to:
  - adb.mid_analytics_payment_event_outbox
  - adb.mid_analytics_payment_reconciliation
---

## Назначение

Процесс переносит payment measurement v2 из production MID/madeindream в ADB, чтобы Superset строил datasets только из ADB.

Статус на 2026-08-14:

- DDL ADB-зеркал выполнен;
- обновленный PHP cron `adb.php` загружен на сервер;
- cron `adb.php` запущен по расписанию, ADB-зеркала заполнены;
- ADB views `payment_events_v2`, `order_payments_v2`, `payment_campaign_attribution_v2` созданы;
- procedure `adb.refresh_analytics_data_quality_daily` пересоздана;
- Superset datasets созданы: `payment_events_v2` id `34`, `order_payments_v2` id `35`, `payment_campaign_attribution_v2` id `36`.

## Техническая информация

- Код: PHP cron `adb.php` в production CRM
- Класс: `ControllerCronAdb`
- Функции:
  - `move_mid_payment_event_outbox`
  - `move_mid_payment_reconciliation`
- Расписание: вместе с cron `adb.php`, ежедневно в 06:30
- Лог: `adb.log`

## Источник данных

| Production table | ADB mirror |
|---|---|
| `madeindream.analytics_payment_event_outbox` | `adb.mid_analytics_payment_event_outbox` |
| `madeindream.analytics_payment_reconciliation` | `adb.mid_analytics_payment_reconciliation` |

## Режим записи

```text
TRUNCATE -> batch INSERT
```

Такой режим соответствует текущему стилю `adb.php`: большинство ADB-таблиц пересобирается полностью во время ежедневного cron.

## Payment v2 contract

- Канонический финансовый факт: первый `operator_verified order_paid_v2`.
- Grain финансовой витрины: `order_id + payment_id`.
- `purchase_paid` используется как контроль ecommerce-доставки, суммы и валюты.
- `attribution_trust` читается из outbox и не пересчитывается в Superset.
- `untrusted` оплаты остаются в gross paid, но исключаются из доказанных campaign CPA/ROAS/CAC.

## Контрольные production read-back итоги

Канонический read-back после ClientID fix:

| Trust | Event rows | Orders |
|---|---:|---:|
| `trusted` | `536` | `179` |
| `untrusted` | `564` | `188` |

Для `untrusted` ожидается распределение:

```text
order_paid      188 rows / 188 orders
order_paid_v2   188 rows / 188 orders
purchase_paid   188 rows / 188 orders
```

## Quality checks

Добавлены в `adb.refresh_analytics_data_quality_daily`:

- `payment_outbox_trusted_event_rows`
- `payment_outbox_untrusted_event_rows`
- `payment_gross_paid_trusted_orders`
- `payment_gross_paid_untrusted_orders`
- `payment_missing_purchase_paid_orders`
- `payment_amount_mismatch_orders`
- `payment_currency_mismatch_orders`
- `payment_postfix_sent_missing_client_id`
- `payment_postfix_new_client_id_reuse_clusters`
