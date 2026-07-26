# Chart: Новые и лояльные покупатели за период

---
type: chart
name: Новые и лояльные покупатели за период
superset_chart_id: 63
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Donut chart показывает долю новых и лояльных покупателей за период.

## Superset

- Chart ID: `63`
- Visualization type: `pie`
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=63`

## Группировка

```sql
CASE
    WHEN count_orders = 1 THEN 'Новые'
    ELSE 'Лояльные (больше одной покупки)'
END
```

## Метрика

| Метрика | Формула |
|---|---|
| `Покупатели` | `COUNT(DISTINCT customer_id)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_task_added` | `No filter` |
| `date_order_added` | `Last month` |

## Агрегация в чарте

Чарт считает уникальных покупателей по сегменту:

- `Новые`, если `count_orders = 1`;
- `Лояльные`, если у покупателя больше одного валидного заказа.

