# Chart: Общая прибыль за период

---
type: chart
name: Общая прибыль за период
superset_chart_id: 61
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Big number показывает сумму прибыли по заказам за выбранный период.

## Superset

- Chart ID: `61`
- Visualization type: `big_number`
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=61`
- X-axis: `date_order_added`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `SUM(Прибыль по заказу)` | `SUM(profit)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_order_added` | `Last quarter` |

## Агрегация в чарте

Чарт суммирует `profit` за выбранный период.

