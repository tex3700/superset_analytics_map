# Chart: Динамика выручка и прибыль по дням

---
type: chart
name: Динамика выручка и прибыль по дням
superset_chart_id: 62
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Line chart показывает дневную динамику выручки и прибыли.

## Superset

- Chart ID: `62`
- Visualization type: `echarts_timeseries_line`
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=62`
- X-axis: `date_order_added`
- Time grain: `P1D`

## Метрики

| Метрика | Формула |
|---|---|
| `Прибыль` | `SUM(profit)` |
| `Выручка` | `SUM(revenue)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_task_added` | `No filter` |
| `date_order_added` | `Last month` |

## Агрегация в чарте

Чарт суммирует `profit` и `revenue` по дням заказа.

