# Chart: Общая выручка за период

---
type: chart
name: Общая выручка за период
superset_chart_id: 60
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Big number показывает сумму выручки по заказам за выбранный период.

## Superset

- Chart ID: `60`
- Visualization type: `big_number`
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=60`
- X-axis: `date_order_added`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `SUM(Выручка по заказу)` | `SUM(revenue)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_order_added` | `Last quarter` |

## Агрегация в чарте

Чарт суммирует `revenue` за выбранный период.

