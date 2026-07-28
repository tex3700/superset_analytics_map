# Dataset: ad_order_attribution_v2

---
type: dataset
name: ad_order_attribution_v2
superset_dataset_id: 31
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.orders
  - adb.orders_totals
  - adb.ya_metrica_users_goals_detailed
used_by_charts:
  - ne-agregirovannye-dannye-po-zakazam-s-reklamnyh-ploshchadok
---

## Краткое описание

Order-level virtual dataset для диагностики рекламной attribution. В отличие от агрегированного [ad_stats_and_roi_v2](./ad_stats_and_roi_v2.md), этот dataset сохраняет детализацию до уровня заказа.

Главный grain:

```text
1 строка = 1 shop_order_id
```

Dataset начинается от `adb.orders` и через `LEFT JOIN` присоединяет строки Метрики. Поэтому он показывает не только связанные заказы, но и `unmatched`.

## Superset

- Dataset ID: `31`
- Dataset name: `ad_order_attribution_v2`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/31`
- Owner: Дмитрий Пыланкин
- Main datetime column: `order_date_added`

## Логика связи

Связь с Метрикой строится по fallback-условию:

```text
orders.ym_uid = ya_metrica_users_goals_detailed.client_id
```

Дополнительно строка Метрики должна попадать в окно заказа:

```text
stat_date BETWEEN DATE(order_date_added) AND DATE(order_date_modified)
```

Если `order_date_modified` пустой:

```text
stat_date = DATE(order_date_added)
```

У строки Метрики должна быть хотя бы одна из целей:

```text
order_paid
add_to_cart
ecommerce
quick_order_registration
zakaz_oformlen
```

Если к заказу подходит несколько строк Метрики, выбирается одна через `ROW_NUMBER()` с приоритетом целей.

## Поля качества

| Поле | Значение / смысл |
|---|---|
| `join_quality` | `fallback_ym_uid_date_goal`, если заказ связан с Метрикой; `unmatched`, если подходящей строки Метрики нет |
| `attribution_status` | `matched`, `missing_ym_uid`, `no_metrika_goal_in_order_window` |
| `attribution_goal` | Цель Метрики, по которой выбран кандидат |
| `attribution_priority` | Приоритет цели: чем меньше число, тем сильнее attribution-сигнал |

`exact` пока не используется, потому что нет прямого ключа `order_id/payment_id/transaction_id` между заказом, оплатой и событием Метрики.

## Финансовые поля

| Поле | Описание |
|---|---|
| `order_total_value` | Итоговая сумма заказа из `orders_totals`, `code = total` |
| `attributed_revenue` | Выручка, которую можно использовать в рекламной attribution |
| `cost_value` | Себестоимость из `orders_totals`, `code = cost` |
| `shipping_value` | Доставка из `orders_totals`, `code = shipping` |

`attributed_revenue` заполняется только для неотмененных/невозвратных заказов и целей `order_paid` / `zakaz_oformlen`.

## Используется в чартах

| Chart ID | Чарт | Назначение |
|---:|---|---|
| `108` | [Не агрегированные данные по заказам с рекламных площадок](../charts/ne-agregirovannye-dannye-po-zakazam-s-reklamnyh-ploshchadok.md) | Raw table для проверки order-level attribution |

