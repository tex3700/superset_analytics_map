# ADB Table: analytics_data_quality_daily

---
type: table
name: analytics_data_quality_daily
database: ADB
schema: adb
source_system: ADB quality checks
loaded_by:
  - n8n:analytics_data_quality
used_by_datasets:
  - analytics_data_quality_daily
---

## Назначение

Таблица хранит ежедневные результаты проверок качества аналитики. Сейчас проверки считаются за rolling window 90 дней и записываются через n8n workflow `Analytics data quality`.

Заполнение выполняется вызовом хранимой процедуры:

```sql
CALL adb.refresh_analytics_data_quality_daily();
```

## Grain

```text
1 check_date + 1 check_name
```

## Поля

| Поле | Описание |
|---|---|
| `id` | Технический ID строки |
| `check_date` | Дата проверки |
| `check_name` | Техническое имя проверки |
| `check_group` | Группа проверки |
| `metric_value` | Числовое значение |
| `metric_text` | Текстовое значение |
| `status` | `ok`, `warning`, `info` |
| `comment` | Комментарий |
| `created_at` | Время вставки/обновления |

## Обновление

- Процесс: [Analytics data quality](../ingestion/analytics-data-quality.md)
- Частота: ежедневно по расписанию n8n
- Метод: `REPLACE INTO` внутри procedure `refresh_analytics_data_quality_daily`
- Окно расчета: последние 90 дней

## Payment v2 checks

После подключения payment measurement v2 процедура также пишет проверки:

| Check name | Смысл |
|---|---|
| `payment_outbox_trusted_event_rows` | Количество trusted event rows и orders в `mid_analytics_payment_event_outbox` |
| `payment_outbox_untrusted_event_rows` | Количество untrusted event rows и orders; канонический target `564 / 188` |
| `payment_gross_paid_trusted_orders` | Trusted gross paid на каноническом grain `order_id + payment_id` |
| `payment_gross_paid_untrusted_orders` | Untrusted gross paid, сохраняемый в финансовом факте |
| `payment_missing_purchase_paid_orders` | Канонические оплаты без sibling `purchase_paid` |
| `payment_amount_mismatch_orders` | Расхождение суммы между `order_paid_v2` и sibling events |
| `payment_currency_mismatch_orders` | Расхождение валюты между `order_paid_v2` и sibling events |
| `payment_postfix_sent_missing_client_id` | Post-fix sent trusted events с пустым `client_id` |
| `payment_postfix_new_client_id_reuse_clusters` | Post-fix новые reuse-кластеры ClientID |
