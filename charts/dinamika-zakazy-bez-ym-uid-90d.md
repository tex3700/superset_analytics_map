# Chart: Динамика - Заказы без ym_uid 90d

---
type: chart
name: Динамика - Заказы без ym_uid 90d
superset_chart_id: 109
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Bar chart показывает динамику количества заказов без `ym_uid` за rolling window 90 дней.

## Superset

- Chart ID: `109`
- Visualization type: `echarts_timeseries_bar`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=109`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `check_date` |
| time grain | `P1D` |
| metric | `MAX(metric_value)` |
| filter | `check_name = orders_without_ym_uid_90d` |
| temporal filter | `No filter` |

