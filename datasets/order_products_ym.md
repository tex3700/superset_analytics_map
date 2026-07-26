# Dataset: order_products_ym

---
type: dataset
name: order_products_ym
superset_dataset_id: 16
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.order_products
  - php_cron:adb_cron_php
used_by_charts:
  - tovary-v-zakaze
---

## Краткое описание

Virtual dataset Superset, который показывает товарные строки заказов из таблицы [adb.order_products](../tables/order_products.md).

Несмотря на название `order_products_ym`, датасет не загружается из Яндекс.Метрики напрямую. Он используется рядом с метрикой для анализа товаров в заказах.

## Superset

- Dataset ID: `16`
- Dataset name: `order_products_ym`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/16`
- Owner: Дмитрий Пыланкин
- Main datetime column: не задана

## Источники данных

| Источник | Процесс загрузки | Таблицы ADB |
|---|---|---|
| MID database | PHP cron `adb.php` | [adb.order_products](../tables/order_products.md) |

## SQL датасета

```sql
SELECT *
FROM adb.order_products
```

## Grain

Одна строка соответствует:

```text
1 товарная строка заказа
```

Это тот же grain, что у физической таблицы [adb.order_products](../tables/order_products.md).

## Поля

| Поле | Тип | Описание |
|---|---|---|
| `order_products_id` | LONG | Технический ID строки |
| `shop_order_id` | LONG | ID заказа магазина |
| `model` | VAR_STRING | Товар / модель |
| `quantity` | LONG | Количество |
| `price` | NEWDECIMAL | Цена |
| `category` | VAR_STRING | Категория |

## Агрегации внутри датасета

Нет. Датасет является прямым `SELECT *` из `adb.order_products`.

## Обновление данных

- Процесс: PHP cron `adb.php`
- URL: `https://crm.madeindream.com/admin/index.php?route=cron/adb`
- Расписание: ежедневно в 06:30
- Функция: `move_order_products`
- Режим обновления: `TRUNCATE -> batch INSERT`

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Товары в заказе](../charts/tovary-v-zakaze.md) | [Поведенческий из Яндекс.Метрики](../dashboards/povedencheskiy-iz-yandex-metriki.md), Детализация по продажам и воронке лидов | Raw table |

## Особенности

- В датасете нет temporal column, поэтому временная фильтрация по дате заказа на уровне этого датасета невозможна без join с заказами.
- Поле `price` берется из `order_product.price`. В текущем чарте оно отображается как есть.

