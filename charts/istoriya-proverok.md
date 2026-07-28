# Chart: История проверок

---
type: chart
name: История проверок
superset_chart_id: 118
dataset: analytics_data_quality_daily
dashboards:
  - Качество данных аналитики
---

## Назначение

Raw table показывает историю всех quality checks.

## Superset

- Chart ID: `118`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.analytics_data_quality_daily` (id `32`)
- Dashboard: `16`
- URL: `http://81.200.152.123:8088/explore/?slice_id=118`

## Используемые поля

```text
check_date
created_at
check_name
check_group
metric_value
metric_text
status
comment
```

## Настройки

| Параметр | Значение |
|---|---|
| temporal filter | `check_date = No filter` |
| row limit | `10000` |
| server page length | `50` |

