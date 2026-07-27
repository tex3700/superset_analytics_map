# Chart: Метрика по пользователям достигшим цели ymugdV2

---
type: chart
name: Метрика по пользователям достигшим цели ymugdV2
superset_chart_id: 105
dataset: ya_metrica_users_goals_detailed_v2
dashboards:
  - Поведенческий из Яндекс.Метрики
purpose: comparison
---

## Назначение

Raw table для проверки order-level attribution в [ya_metrica_users_goals_detailed_v2](../datasets/ya_metrica_users_goals_detailed_v2.md).

Важно: chart показывает не всех пользователей Метрики, достигших целей, а только строки v2, которые были связаны с заказом и дедуплицированы до уровня `shop_order_id`.

## Superset

- Chart ID: `105`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.ya_metrica_users_goals_detailed_v2` (id `29`)
- Dashboard: `12`
- URL: `http://81.200.152.123:8088/explore/?slice_id=105`

## Используемые поля

Основные поля: `stat_date`, `client_id`, `first_traffic_source`, `first_utm_source`, `first_utm_medium`, `last_sign_utm_source`, `last_sign_utm_medium`, `goal_*_reaches`, `shop_order_id`, `order_status`, `product_quantity`, `total_price`, `total_price_with_quantity`, `shipping_value`, `total_value`, `crm_task_id`, `crm_task_status`, `attribution_goal`, `attribution_priority`.

## Фильтры и сортировка

| Поле | Значение |
|---|---|
| `stat_date` | `Last quarter` |
| sort | `stat_date DESC` |
| row limit | `1000` |

## Агрегация

Нет. Chart выводит raw rows из virtual dataset.

