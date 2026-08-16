# Ingestion: Analytics data quality

---
type: ingestion
name: Analytics data quality
tool: n8n
target_tables:
  - adb.analytics_data_quality_daily
depends_on:
  - adb.orders
  - adb.orders_totals
  - adb.ya_metrica_users_goals_detailed
  - superset_dataset:ad_order_attribution_v2
---

## Назначение

n8n workflow `Analytics data quality` ежедневно обновляет таблицу качества данных [adb.analytics_data_quality_daily](../tables/analytics_data_quality_daily.md).

Workflow вызывает MySQL procedure:

```sql
CALL adb.refresh_analytics_data_quality_daily();
```

## Окно расчета

Проверки считаются за последние 90 дней:

```sql
CURDATE() - INTERVAL 90 DAY
```

Такое окно выбрано потому, что данные аналитики собираются недавно, а в `orders` есть более старая история, которая не отражает качество текущей attribution-модели.

## Логика workflow

```text
Schedule Trigger
  -> MySQL: CALL adb.refresh_analytics_data_quality_daily()
  -> MySQL: read today's checks
```

В процедуре результаты записываются в `analytics_data_quality_daily` через `REPLACE INTO`, поэтому за одну дату и один `check_name` хранится актуальное значение.

## Checks

| check_name | Смысл |
|---|---|
| `orders_total_90d` | Всего заказов за последние 90 дней |
| `orders_without_ym_uid_90d` | Заказы без `ym_uid` |
| `unmatched_orders_90d` | Заказы без подходящей строки Метрики |
| `matched_orders_90d` | Заказы, связанные с Метрикой |
| `attribution_match_rate_90d` | Доля связанных заказов |
| `attributed_revenue_90d` | Атрибутированная выручка |
| `orders_total_revenue_90d` | Общая сумма заказов |
| `attributed_revenue_share_90d` | Доля атрибутированной выручки |

## Payment v2 checks

После подключения payment measurement v2 эта же процедура пишет проверки по ADB-зеркалам payment outbox/reconciliation:

| check_name | Смысл |
|---|---|
| `payment_outbox_trusted_event_rows` | Trusted event rows и distinct orders; контроль production read-back `536 / 179` |
| `payment_outbox_untrusted_event_rows` | Untrusted event rows и distinct orders; контроль production read-back `564 / 188` |
| `payment_gross_paid_trusted_orders` | Количество trusted оплат на grain `order_id + payment_id` |
| `payment_gross_paid_untrusted_orders` | Количество untrusted оплат, сохраняемых в общем финансовом факте |
| `payment_gross_paid_trusted_rub/kzt/byn` | Trusted gross paid отдельно по валютам |
| `payment_gross_paid_untrusted_rub/kzt/byn` | Untrusted gross paid отдельно по валютам |
| `payment_missing_purchase_paid_orders` | Канонические `order_paid_v2` без sibling `purchase_paid` |
| `payment_amount_mismatch_orders` | Расхождения суммы между `order_paid_v2` и sibling events |
| `payment_currency_mismatch_orders` | Расхождения валюты между `order_paid_v2` и sibling events |
| `payment_postfix_sent_missing_client_id` | Post-fix sent trusted events с пустым `client_id` |
| `payment_postfix_new_client_id_reuse_clusters` | Post-fix новые reuse-кластеры ClientID |

## Используется

- Superset physical dataset: [analytics_data_quality_daily](../datasets/analytics_data_quality_daily.md), id `32`
- Dashboard: [Качество данных аналитики](../dashboards/kachestvo-dannyh-analitiki.md), id `16`
