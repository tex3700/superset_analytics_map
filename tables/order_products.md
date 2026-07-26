# ADB Table: order_products

---
type: adb_table
name: order_products
schema: adb
database: ADB
updated_by:
  - php_cron:adb_cron_php
source_tables:
  - madeindream.order_product
  - madeindream.product_to_category
  - madeindream.category
  - madeindream.category_description
used_by_datasets:
  - ya_metrica_users_goals_detailed
  - order_products_ym
  - ad_stats_and_roi
---

## Назначение

Таблица содержит товары в заказах интернет-магазина.

Для аналитики она используется как источник количества товаров и товарной суммы по заказу.

## Источник и обновление

- Процесс: [ADB cron PHP](../ingestion/adb-cron-php.md)
- Функция в PHP: `move_order_products`
- Расписание: ежедневно в 06:30
- Режим обновления: `TRUNCATE -> batch INSERT`
- Основной источник: MID table `order_product`
- Дополнительные источники категорий: `product_to_category`, `category`, `category_description`

## Grain

Одна строка соответствует:

```text
1 товарная строка заказа
```

## Поля

| Поле ADB | Источник / логика | Описание |
|---|---|---|
| `shop_order_id` | `order_product.order_id` | ID заказа магазина |
| `model` | `order_product.model` | Модель / артикул товара |
| `quantity` | `order_product.quantity` | Количество товара в строке заказа |
| `price` | `order_product.price` | Цена товара в строке заказа |
| `category` | построенный путь категории | Категория товара в формате иерархии |

## Логика категории

Скрипт строит путь категории по дереву `category.parent_id`.

Для каждого товара берется первая найденная категория из `product_to_category`, затем для нее строится путь:

```text
Категория > Подкатегория > ...
```

## Используется в датасетах

| Датасет | Роль |
|---|---|
| [ya_metrica_users_goals_detailed](../datasets/ya_metrica_users_goals_detailed.md) | Агрегация количества и суммы товаров по заказу |
| [order_products_ym](../datasets/order_products_ym.md) | Прямая витрина товарных строк заказов |
| [ad_stats_and_roi](../datasets/ad_stats_and_roi.md) | Агрегируется по заказу в CTE `order_products_agg`; поля не используются в финальной выборке и не влияют на расчет ROI |

## Использование в virtual dataset

В `ya_metrica_users_goals_detailed` таблица агрегируется до уровня заказа:

```sql
SELECT
    shop_order_id,
    SUM(quantity) AS product_quantity,
    SUM(price) AS total_price
FROM order_products
GROUP BY shop_order_id
```

## Особенности

- Таблица полностью пересобирается каждый день.
- `total_price` в virtual dataset считается как `SUM(price)`, без умножения на `quantity`. Если `price` хранится как цена единицы, это нужно учитывать при интерпретации.
