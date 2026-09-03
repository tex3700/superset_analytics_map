# ADB Table: mid_analytics_refund_event

---
type: table
name: mid_analytics_refund_event
database: ADB
schema: adb
source_system: MID / madeindream
loaded_by:
  - php_cron:adb_cron_php
used_by_datasets:
  - payment_refund_events_v1
  - order_payment_net_v1
  - payment_campaign_net_attribution_v1
---

## Назначение

ADB-зеркало production-таблицы `madeindream.analytics_refund_event`.

Таблица содержит refund/cancel events для payment-v2. Superset читает только ADB, поэтому production refund source переносится в `adb.mid_analytics_refund_event`.

## Grain

```text
1 строка = 1 event_id
business grain = event_name + order_id + payment_id + refund_id
```

## События

| `event_name` | Смысл | Финансово уменьшает revenue |
|---|---|---|
| `order_refund_v1` | Подтвержденный или отклоненный возврат | Да, только если `financial_state='accepted'` |
| `order_cancelled_v1` | Отмена заказа | Нет |

Cancel не создает refund-строку. Для cancel `refund_amount=0`, `financial_state='non_financial'`.

## Ключевые поля

| Поле | Описание |
|---|---|
| `event_id` | Уникальное событие |
| `event_name` | `order_refund_v1` / `order_cancelled_v1` |
| `order_id` | ID заказа MID |
| `payment_id` | ID исходной оплаты |
| `refund_id` | Стабильный ключ возврата/отмены |
| `paid_event_id` | ID исходного payment event, если backend смог его указать |
| `refunded_at` | Время события refund/cancel |
| `refund_amount` | Сумма возврата; для cancel `0` |
| `gross_paid_amount` | Gross paid сумма исходной оплаты по backend refund source |
| `currency` | Валюта возврата, без FX |
| `refund_type` | `full` / `partial` |
| `reason_code` | Код причины без персональных данных |
| `source` | Источник события |
| `operator_id` | ID оператора |
| `operator_username` | Username оператора |
| `order_status_id` | Статус заказа на момент события |
| `paid_before_cancel` | Был ли заказ оплачен до cancel |
| `attribution_trust` | Trust исходной оплаты для трассировки |
| `financial_state` | `accepted`, `rejected`, `non_financial` |
| `quality_flags` | Quality-флаги backend |
| `payload_hash` | Hash payload для дедупликации/аудита |
| `status` | Технический статус события |
| `updated_at` | Время обновления source row |
| `adb_synced_at` | Время переноса строки в ADB |

## Правила использования

- В net входит только `event_name='order_refund_v1' AND financial_state='accepted'`.
- `rejected` и `non_financial` не уменьшают revenue.
- `quality_flags='refund_without_campaign_source_payment'` входит в общий net, но не атрибутируется на кампанию.
- `quality_flags='orphan_refund'` должен идти с `financial_state='rejected'`, поэтому не входит в net formula.
- RUB/KZT/BYN нельзя суммировать без документированного FX.
