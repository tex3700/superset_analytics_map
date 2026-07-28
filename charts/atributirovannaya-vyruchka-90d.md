# Chart: Атрибутированная выручка 90d

---
type: chart
name: Атрибутированная выручка 90d
superset_chart_id: 115
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Big number показывает атрибутированную выручку за rolling window 90 дней.

## Superset

- Chart ID: `115`
- Visualization type: `big_number`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=115`

## Настройки

| Параметр | Значение |
|---|---|
| metric | `MAX(metric_value)` |
| metric label | `Attributed revenue` |
| filter | `check_name = attributed_revenue_90d` |
| aggregation | `LAST_VALUE` |

