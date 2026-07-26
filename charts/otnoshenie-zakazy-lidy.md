# Chart: Отношение Заказы - Лиды

---
type: chart
name: Отношение Заказы - Лиды
superset_chart_id: 72
dataset: con_lids_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Big number показывает отношение количества заказов к количеству лидов за выбранный период.

## Superset

- Chart ID: `72`
- Visualization type: `big_number`
- Dataset: `adb.con_lids_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=72`
- X-axis: `date`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `Отношение Заказы/Лиды` | `CASE WHEN SUM(count_lids) > 0 THEN SUM(count_orders) / SUM(count_lids) ELSE 0 END` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date` | `Last quarter` |

## Агрегация в чарте

Чарт суммирует дневные значения за период:

```text
SUM(count_orders) / SUM(count_lids)
```

Если лидов за период нет, возвращается `0`.

## Особенности

- Big number использует aggregation `LAST_VALUE` для временного ряда.
- Включены timestamp, trend line и сравнение с предыдущим периодом (`compare_lag = 1`).

