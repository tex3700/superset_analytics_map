# Chart: Средний чек (AOV) по заказам за период

---
type: chart
name: Средний чек (AOV) по заказам за период
superset_chart_id: 70
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Big number показывает средний чек по заказам за выбранный период.

## Superset

- Chart ID: `70`
- Visualization type: `big_number`
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=70`
- X-axis: `date_order_added`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `AVG(Выручка по заказу)` | `AVG(revenue)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_task_added` | `No filter` |
| `date_order_added` | `Last quarter` |

## Агрегация в чарте

Чарт усредняет `revenue` по строкам заказов.

