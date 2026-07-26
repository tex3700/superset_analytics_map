# Chart: Среднее время от регистрации лида до первого заказа (в часах)

---
type: chart
name: Среднее время от регистрации лида до первого заказа (в часах)
superset_chart_id: 65
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Big number показывает среднее количество часов от первого касания лида до первого заказа.

## Superset

- Chart ID: `65`
- Visualization type: `big_number`
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=65`
- X-axis: `date_order_added`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `AVG(hours_to_first_order_numeric)` | `AVG(hours_to_first_order_numeric)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_task_added` | `No filter` |
| `hours_to_first_order_numeric` | `< 8760` |
| `date_order_added` | `Last quarter` |

## Агрегация в чарте

Чарт усредняет числовое поле часов до первого заказа. Фильтр `< 8760` отсекает значения больше одного года.

