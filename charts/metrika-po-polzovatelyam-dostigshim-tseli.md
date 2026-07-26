# Chart: Метрика по пользователям достигшим цели

---
type: chart
name: Метрика по пользователям достигшим цели
superset_chart_id: 43
dataset: ya_metrica_users_goals_detailed
dashboards:
  - Поведенческий из Яндекс.Метрики
---

## Назначение

Табличный чарт показывает детальные строки пользователей Яндекс.Метрики, достигших целей, вместе с привязанными заказами и суммами, если заказ удалось сопоставить по `client_id = ym_uid`.

## Superset

- Chart ID: `43`
- Visualization type: `table`
- Dataset: `adb.ya_metrica_users_goals_detailed`
- Dashboard ID: `12`
- URL: `http://81.200.152.123:8088/explore/?slice_id=43`

## Используемые поля

Чарт работает в `query_mode = raw`.

Основные поля:

```text
stat_date
client_id
first_traffic_source
first_visit_date
first_utm_source
first_utm_medium
start_url
previous_visit_date
last_sign_utm_source
visits
pageviews
page_depth
avg_visit_duration_sec
goal_*_reaches
shop_order_id
order_status
product_quantity
total_price
total_value
```

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last quarter` |

## Агрегация в чарте

Нет. Чарт выводит raw rows из virtual dataset.

## Особенности

- Лимит строк: `1000`.
- Серверная пагинация включена, размер страницы `50`.
- Сортировка: `stat_date` по убыванию.

