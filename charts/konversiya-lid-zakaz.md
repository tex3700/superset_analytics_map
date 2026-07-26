# Chart: Конверсия (Лид -> Заказ)

---
type: chart
name: Конверсия (Лид -> Заказ)
superset_chart_id: 73
dataset: rel_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Big number показывает процентную конверсию валидных лидов в заказы новых клиентов.

## Superset

- Chart ID: `73`
- Visualization type: `big_number`
- Dataset: `adb.rel_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=73`
- X-axis: `date`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `Конверсия %` | `CASE WHEN SUM(count_lids) > 0 THEN SUM(count_orders) * 100.0 / SUM(count_lids) ELSE 0 END` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date` | `Last quarter` |

## Агрегация в чарте

Чарт суммирует дневные значения за период:

```text
SUM(count_orders) * 100 / SUM(count_lids)
```

Если лидов за период нет, возвращается `0`.

## Особенности

- Big number использует aggregation `LAST_VALUE` для временного ряда.
- Включены timestamp, trend line и сравнение с предыдущим периодом (`compare_lag = 1`).

