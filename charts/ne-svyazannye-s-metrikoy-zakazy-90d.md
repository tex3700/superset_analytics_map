# Chart: Не связанные с метрикой заказы 90d

---
type: chart
name: Не связанные с метрикой заказы 90d
superset_chart_id: 112
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Big number показывает количество заказов за последние 90 дней, которые не удалось связать с Метрикой.

## Superset

- Chart ID: `112`
- Visualization type: `big_number`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=112`

## Настройки

| Параметр | Значение |
|---|---|
| metric | `MAX(metric_value)` |
| metric label | `Unmatched orders` |
| filter | `check_name = unmatched_orders_90d` |
| aggregation | `LAST_VALUE` |

