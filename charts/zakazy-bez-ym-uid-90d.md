# Chart: Заказы без ym_uid 90d

---
type: chart
name: Заказы без ym_uid 90d
superset_chart_id: 110
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Big number показывает последнее записанное значение проверки `orders_without_ym_uid_90d`.

## Superset

- Chart ID: `110`
- Visualization type: `big_number`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=110`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `check_date` |
| time grain | `P1D` |
| metric | `MAX(metric_value)` |
| metric label | `Заказы без ym_uid` |
| filter | `check_name = orders_without_ym_uid_90d` |
| aggregation | `LAST_VALUE` |

