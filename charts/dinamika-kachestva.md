# Chart: Динамика качества

---
type: chart
name: Динамика качества
superset_chart_id: 116
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Line chart показывает динамику всех quality checks по датам.

## Superset

- Chart ID: `116`
- Visualization type: `echarts_timeseries_line`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=116`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `check_date` |
| time grain | `P1D` |
| metric | `MAX(metric_value)` |
| group by | `check_name` |
| temporal filter | `No filter` |

