# Chart: Количество заказов за период

---
type: chart
name: Количество заказов за период
superset_chart_id: 64
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Big number показывает количество заказов за выбранный период.

## Superset

- Chart ID: `64`
- Visualization type: `big_number`
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=64`
- X-axis: `date_order_added`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `count` | `COUNT(*)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_task_added` | `No filter` |
| `date_order_added` | `Last quarter` |

## Агрегация в чарте

Так как grain датасета ожидается `1 строка = 1 заказ`, `COUNT(*)` интерпретируется как количество заказов.

