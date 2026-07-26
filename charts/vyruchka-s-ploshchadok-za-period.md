# Chart: Выручка с площадок за период

---
type: chart
name: Выручка с площадок за период
superset_chart_id: 53
dataset: ad_stats_and_roi
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Big number показывает сумму выручки, атрибутированной рекламным площадкам, за выбранный период.

## Superset

- Chart ID: `53`
- Visualization type: `big_number`
- Dataset: `adb.ad_stats_and_roi`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=53`
- X-axis: `stat_date`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `SUM(Выручка от рекламы)` | `SUM(revenue)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last quarter` |

## Агрегация в чарте

Чарт суммирует `revenue` за выбранный период.

## Особенности

- Big number использует aggregation `LAST_VALUE` для временного ряда.
- Включены timestamp, trend line и сравнение с предыдущим периодом (`compare_lag = 1`).

