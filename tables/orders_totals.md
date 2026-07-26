# ADB Table: orders_totals

---
type: adb_table
name: orders_totals
schema: adb
database: ADB
updated_by:
  - php_cron:adb_cron_php
source_tables:
  - madeindream.order_total
  - madeindream.order_product
  - madeindream.product
used_by_datasets:
  - ya_metrica_users_goals_detailed
  - ad_stats_and_roi
  - mart_lid_orders
---

## Назначение

Таблица содержит финансовые строки итогов заказа: total, shipping, discounts, taxes, а также добавленную скриптом строку себестоимости `cost`.

## Источник и обновление

- Процесс: [ADB cron PHP](../ingestion/adb-cron-php.md)
- Функция в PHP: `move_orders_totals`
- Расписание: ежедневно в 06:30
- Режим обновления: `TRUNCATE -> batch INSERT`
- Основной источник: MID table `order_total`
- Дополнительная строка `cost`: расчет из `order_product` и `product.cost`

## Grain

Одна строка соответствует:

```text
1 финансовая строка заказа
```

Один заказ может иметь несколько строк с разными `code`.

## Поля

| Поле ADB | Источник / логика | Описание |
|---|---|---|
| `shop_order_id` | `order_total.order_id` | ID заказа магазина |
| `code` | `order_total.code` или расчетный `cost` | Тип финансовой строки |
| `title` | `order_total.title` | Название строки |
| `value` | `order_total.value` или расчет | Значение строки |
| `sort_order` | `order_total.sort_order` или `99` для `cost` | Порядок отображения |

## Дополнительная строка `cost`

После переноса `order_total` скрипт добавляет строку:

```text
code = cost
title = Себестоимость
value = SUM(product.cost * order_product.quantity)
sort_order = 99
```

Расчет выполняется по каждому заказу.

## Используется в датасетах

| Датасет | Роль |
|---|---|
| [ya_metrica_users_goals_detailed](../datasets/ya_metrica_users_goals_detailed.md) | Получение итоговой суммы заказа `total_value` |
| [ad_stats_and_roi](../datasets/ad_stats_and_roi.md) | Источник выручки, агрегируемой по рекламной площадке |
| [mart_lid_orders](../datasets/mart_lid_orders.md) | Источник `revenue`, `cost`, `shipping` и `profit` по заказу |

## Использование в virtual dataset

В `ya_metrica_users_goals_detailed` используется только строка `code = 'total'`:

```sql
SELECT
    shop_order_id,
    MAX(value) AS total_value
FROM orders_totals
WHERE code = 'total'
GROUP BY shop_order_id
```

## Особенности

- Таблица полностью пересобирается каждый день.
- `total_value` в датасете берется как `MAX(value)` по строкам `code = 'total'` на заказ.
- Расчетная строка `cost` может использоваться в других датасетах для прибыльности, но в `ya_metrica_users_goals_detailed` она не используется.
