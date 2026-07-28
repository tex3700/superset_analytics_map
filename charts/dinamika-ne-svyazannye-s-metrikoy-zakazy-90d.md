# Chart: Динамика - Не связанные с метрикой заказы 90d

---
type: chart
name: Динамика - Не связанные с метрикой заказы 90d
superset_chart_id: 114
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Bar chart показывает динамику `unmatched_orders_90d`.

## Superset

- Chart ID: `114`
- Visualization type: `echarts_timeseries_bar`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=114`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `check_date` |
| time grain | `P1D` |
| metric | `MAX(metric_value)` |
| filter | `check_name = unmatched_orders_90d` |
| temporal filter | `No filter` |

