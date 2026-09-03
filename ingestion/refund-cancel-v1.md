# Ingestion: Refund/cancel v1

---
type: ingestion
name: refund_cancel_v1
tool: php_cron
schedule: daily 06:30 Europe/Moscow
source:
  - madeindream.analytics_refund_event
writes_to:
  - adb.mid_analytics_refund_event
---

## Назначение

Процесс переносит refund/cancel source из production MID/madeindream в ADB, чтобы Superset мог строить net paid datasets только из ADB.

## Source contract

Одна source table:

```text
madeindream.analytics_refund_event
```

События:

| `event_name` | Финансовый смысл |
|---|---|
| `order_refund_v1` | Refund event; входит в net только при `financial_state='accepted'` |
| `order_cancelled_v1` | Cancel lifecycle event; `refund_amount=0`, `financial_state='non_financial'`, revenue не уменьшает |

Business key:

```text
event_name + order_id + payment_id + refund_id
```

## ADB mirror

```text
adb.mid_analytics_refund_event
```

Режим записи:

```text
TRUNCATE -> batch INSERT
```

## Net formula

Считается отдельно по `currency`:

```text
net_paid_amount = verified_gross_received_amount - confirmed_refund_amount
```

`confirmed_refund_amount` включает только:

```sql
event_name = 'order_refund_v1'
AND financial_state = 'accepted'
```

`rejected` и `non_financial` в net не входят.

Если accepted refund связан с исходной оплатой, но у оплаты `amount_quality='unknown'` или `base_currency`, `verified_gross_received_amount` остается пустым. Такая строка не исправляется через `refund_source_gross_paid_amount`, потому что это не verified received money по принятому amount contract. Вместо этого она помечается `refund_against_unverified_gross_flag=1` и `net_quality='refund_against_unverified_gross'`.

## Attribution inheritance

Refund наследует attribution bucket исходной оплаты из `payment_campaign_attribution_v2`.

Если `quality_flags='refund_without_campaign_source_payment'` или source payment не найден:

- refund входит в общий net;
- campaign attribution становится `unattributed`;
- refund не уменьшает proven campaign net.

## ADB views / Superset datasets

| View / dataset | Grain | Назначение |
|---|---|---|
| `payment_refund_events_v1` | `order_id + payment_id + refund_id` | Event-level refund/cancel diagnostics |
| `order_payment_net_v1` | `order_id + payment_id` | Payment-level gross/refund/net |
| `payment_campaign_net_attribution_v1` | `order_id + payment_id + net attribution decision` | Campaign net attribution |

## Quality checks

Добавляются в `adb.refresh_analytics_data_quality_daily`:

- `refund_event_rows`;
- `refund_duplicate_business_keys`;
- `refund_accepted_events`;
- `refund_orphan_refunds`;
- `refund_currency_mismatch`;
- `refund_amount_exceeds_gross`;
- `refund_against_unverified_gross`;
- `refund_negative_net_payments`;
- `refund_cancelled_after_paid_without_refund`;
- `refund_net_<currency>`.

## Статус

Backend refund/cancel source принят на production. Superset/ADB change-set подготовлен: нужно применить ADB mirror, views, procedure и создать Superset datasets.
