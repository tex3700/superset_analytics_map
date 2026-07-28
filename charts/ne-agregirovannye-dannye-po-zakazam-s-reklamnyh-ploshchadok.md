# Chart: Не агрегированные данные по заказам с рекламных площадок

---
type: chart
name: Не агрегированные данные по заказам с рекламных площадок
superset_chart_id: 108
dataset: ad_order_attribution_v2
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Raw table показывает order-level attribution: одна строка соответствует одному заказу из [ad_order_attribution_v2](../datasets/ad_order_attribution_v2.md).

## Superset

- Chart ID: `108`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.ad_order_attribution_v2` (id `31`)
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=108`

## Настройки

| Параметр | Значение |
|---|---|
| temporal filter | `order_date_added = Last quarter` |
| sort | `shop_order_id DESC` |
| row limit | `10000` |

## Используемые поля

Chart выводит основные поля заказа, Метрики, attribution-связи и финансов: `shop_order_id`, `order_date_added`, `order_status`, `ym_uid`, `client_id`, `platform`, `attribution_goal`, `join_quality`, `attribution_status`, `order_total_value`, `attributed_revenue`, `cost_value`, `shipping_value`.

