# ADB Tables

## Data quality tables

| Таблица | Назначение |
|---|---|
| [analytics_data_quality_daily](./analytics_data_quality_daily.md) | Ежедневные результаты проверок качества аналитики |
| [mid_analytics_payment_event_outbox](./mid_analytics_payment_event_outbox.md) | ADB-зеркало payment outbox из MID/madeindream |
| [mid_analytics_payment_reconciliation](./mid_analytics_payment_reconciliation.md) | ADB-зеркало payment reconciliation из MID/madeindream |
| [mid_analytics_refund_event](./mid_analytics_refund_event.md) | ADB-зеркало refund/cancel events из MID/madeindream |

В этой папке описываем физические таблицы ADB, которые используются несколькими Superset-датасетами.

Такие таблицы лучше документировать отдельно, чтобы:

- не дублировать одну и ту же upstream-логику в каждом датасете;
- явно фиксировать источник, режим обновления и grain таблицы;
- упростить AI-агенту построение lineage между ETL, ADB и Superset.

## Таблицы

| Таблица | Назначение |
|---|---|
| [orders](orders.md) | Заказы магазина с привязкой к Яндекс.Метрике и CRM |
| [order_products](order_products.md) | Товары в заказах |
| [orders_totals](orders_totals.md) | Финансовые строки итогов заказа |
