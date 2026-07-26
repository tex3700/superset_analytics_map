# Chart: Аналитика лидов по заказам

---
type: chart
name: Аналитика лидов по заказам
superset_chart_id: 58
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Raw table с детализацией заказов, CRM-лидов, источников Метрики и финансовых показателей.

## Superset

- Chart ID: `58`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=58`

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_order_added` | `Last quarter` |

## Сортировка

```text
shop_order_id DESC
```

## Агрегация в чарте

Чарт не агрегирует данные. Показывает строки, подготовленные virtual dataset.

## Особенности

- Server pagination: enabled.
- Server page length: `50`.
- Row limit: `10000`.

