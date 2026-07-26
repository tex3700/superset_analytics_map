# Chart: Источники по брошенным заказам

---
type: chart
name: Источники по брошенным заказам
superset_chart_id: 77
dataset: source_abadoned_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Donut/pie chart показывает распределение брошенных заказов по источникам.

## Superset

- Chart ID: `77`
- Visualization type: `pie`
- Dataset: `adb.source_abadoned_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=77`

## Группировка

В чарте используется SQL expression:

```sql
CASE
    WHEN utm_source IS NULL OR utm_source = '' OR utm_source = 'null | null' THEN 'Не определен'
    ELSE utm_source
END
```

## Метрика

| Метрика | Формула |
|---|---|
| `count` | `COUNT(*)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `lid_date_added` | `No filter` |
| `date_task_added` | `Last month` |

## Агрегация в чарте

Чарт считает количество брошенных заказов по нормализованному источнику.

