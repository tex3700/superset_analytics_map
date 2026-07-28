# Chart: Всего заказов 90d

---
type: chart
name: Всего заказов 90d
superset_chart_id: 111
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Big number показывает общее количество заказов за последние 90 дней.

## Superset

- Chart ID: `111`
- Visualization type: `big_number`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=111`

## Настройки

| Параметр | Значение |
|---|---|
| metric | `MAX(metric_value)` |
| metric label | `Всего заказов` |
| filter | `check_name = orders_total_90d` |
| aggregation | `LAST_VALUE` |

