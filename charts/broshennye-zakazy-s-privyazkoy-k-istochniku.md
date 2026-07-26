# Chart: Брошенные заказы с привязкой к источнику

---
type: chart
name: Брошенные заказы с привязкой к источнику
superset_chart_id: 76
dataset: source_abadoned_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Raw table показывает брошенные заказы с найденным источником Яндекс.Метрики.

## Superset

- Chart ID: `76`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.source_abadoned_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=76`

## Фильтры

| Фильтр | Значение |
|---|---|
| `lid_date_added` | `No filter` |
| `date_task_added` | `Last quarter` |

## Агрегация в чарте

Чарт не агрегирует данные. Показывает строки virtual dataset.

