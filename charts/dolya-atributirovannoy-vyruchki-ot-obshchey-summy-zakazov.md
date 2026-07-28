# Chart: Доля атрибутированной выручки от общей суммы заказов

---
type: chart
name: Доля атрибутированной выручки от общей суммы заказов
superset_chart_id: 117
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Big number показывает долю атрибутированной выручки от общей суммы заказов за rolling window 90 дней.

## Superset

- Chart ID: `117`
- Visualization type: `big_number`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=117`

## Настройки

| Параметр | Значение |
|---|---|
| metric | `MAX(metric_value)` |
| metric label | `Attributed revenue share` |
| filter | `check_name = attributed_revenue_share_90d` |
| aggregation | `LAST_VALUE` |

