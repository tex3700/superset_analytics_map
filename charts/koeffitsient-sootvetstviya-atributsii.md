# Chart: Коэффициент соответствия атрибуции

---
type: chart
name: Коэффициент соответствия атрибуции
superset_chart_id: 113
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Big number показывает `attribution_match_rate_90d`: долю заказов за последние 90 дней, которые удалось связать с Метрикой.

Формула в процедуре:

```text
matched_orders_90d / orders_total_90d * 100
```

## Superset

- Chart ID: `113`
- Visualization type: `big_number`
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=113`

## Настройки

| Параметр | Значение |
|---|---|
| metric | `MAX(metric_value)` |
| metric label | `Attribution match rate` |
| filter | `check_name = attribution_match_rate_90d` |
| aggregation | `LAST_VALUE` |

