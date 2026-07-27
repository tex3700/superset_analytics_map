# Dataset: ya_metrica_users_goals_detailed

---
type: dataset
name: ya_metrica_users_goals_detailed
superset_dataset_id: 15
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.ya_metrica_users_goals_detailed
  - adb.orders
  - adb.order_products
  - adb.orders_totals
  - n8n:yandex_metrica_stats_utm_and_users
  - php_cron:adb_cron_php
used_by_charts:
  - metrika-po-polzovatelyam-dostigshim-tseli
  - vyruchka-po-istochnikam
  - dolya-trafika-po-istochniku-pervogo-vizita
used_by_datasets:
  - ad_stats_and_roi
  - mart_lid_orders
  - source_abadoned_orders
---

## Краткое описание

Virtual dataset Superset, который объединяет детализацию пользователей Яндекс.Метрики, достигших целей, с заказами интернет-магазина.

Основная таблица `adb.ya_metrica_users_goals_detailed` заполняется n8n workflow `Yandex metrica stats UTM and Users`.

Заказы и товарные суммы подтягиваются из таблиц ADB, которые ежедневно пересобирает PHP cron `adb.php`.

Статус после разбора customer objections: dataset оставлен как диагностический. Для order-level attribution без размножения заказов создан [ya_metrica_users_goals_detailed_v2](./ya_metrica_users_goals_detailed_v2.md), id `29`.

## Superset

- Dataset ID: `15`
- Dataset name: `ya_metrica_users_goals_detailed`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/15`
- Owner: Дмитрий Пыланкин
- Main datetime column: `stat_date`

## Источники данных

| Источник | Процесс загрузки | Таблицы ADB |
|---|---|---|
| Яндекс.Метрика API | `Yandex metrica stats UTM and Users` | `adb.ya_metrica_users_goals_detailed` |
| MID / CRM databases | PHP cron `adb.php` | [adb.orders](../tables/orders.md), [adb.order_products](../tables/order_products.md), [adb.orders_totals](../tables/orders_totals.md) |

## SQL датасета

```sql
WITH
-- Агрегируем количество и стоимость товаров по каждому заказу
order_products_agg AS (
    SELECT
        shop_order_id,
        SUM(quantity) AS product_quantity,
        SUM(price)   AS total_price
    FROM order_products
    GROUP BY shop_order_id
),
-- Для каждого заказа берём максимальное значение value при code = 'total'
orders_totals_agg AS (
    SELECT
        shop_order_id,
        MAX(value) AS total_value
    FROM orders_totals
    WHERE code = 'total'
    GROUP BY shop_order_id
)

SELECT
    y.*,
    o.shop_order_id,
    o.order_status,
    op.product_quantity,
    op.total_price,
    ot.total_value
FROM ya_metrica_users_goals_detailed y
LEFT JOIN orders o
    ON y.client_id = o.ym_uid
    AND (
        (
            o.date_modified IS NOT NULL
            AND y.stat_date BETWEEN DATE(o.date_added) AND DATE(o.date_modified)
        )
        OR
        (
            o.date_modified IS NULL
            AND y.stat_date = DATE(o.date_added)
        )
    )
    AND (
        y.goal_order_paid_reaches > 0
        OR y.goal_add_to_cart_reaches > 0
        OR y.goal_ecommerce_reaches > 0
        OR y.goal_quick_order_registration_reaches > 0
        OR y.goal_zakaz_oformlen_reaches > 0
    )
LEFT JOIN order_products_agg op
    ON o.shop_order_id = op.shop_order_id
    AND (
        y.goal_order_paid_reaches > 0
        OR y.goal_zakaz_oformlen_reaches > 0
    )
LEFT JOIN orders_totals_agg ot
    ON o.shop_order_id = ot.shop_order_id
    AND (
        y.goal_order_paid_reaches > 0
        OR y.goal_zakaz_oformlen_reaches > 0
    )
    AND o.order_status NOT IN (
        'Отменен',
        'Отмена (дублирующийся заказ)',
        'Неудавшийся',
        'Возврат',
        'Возврат б\\у',
        'Возврат средств'
    )
ORDER BY y.stat_date DESC, o.shop_order_id;
```

## Grain

Базовая таблица `adb.ya_metrica_users_goals_detailed` содержит строки на уровне:

```text
1 дата + 1 client_id + параметры первого/последнего источника + достигнутые цели
```

Физическая таблица имеет следующие ключи и индексы:

```sql
PRIMARY KEY (`id`) USING BTREE,
INDEX `idx_start_url` (`start_url`(255)) USING BTREE,
INDEX `idx_ym_client_statdate` (`client_id`, `stat_date`) USING BTREE
```

После virtual join одна строка может получить данные заказа, если:

- `client_id = orders.ym_uid`;
- дата события Метрики попадает в интервал заказа `date_added` -> `date_modified`, либо равна `date_added`, если `date_modified` пустой;
- у пользователя есть достижение одной из заказных целей.

## Агрегации внутри датасета

В virtual SQL есть две предагрегации:

| CTE | Grain | Формула |
|---|---|---|
| `order_products_agg` | `shop_order_id` | `SUM(quantity) AS product_quantity`, `SUM(price) AS total_price` |
| `orders_totals_agg` | `shop_order_id` | `MAX(value) AS total_value` для `code = 'total'` |

## Логика присоединения заказов

`orders` присоединяется только для строк, где достигнута хотя бы одна цель:

```text
order_paid
add_to_cart
ecommerce
quick_order_registration
zakaz_oformlen
```

Связка:

```text
client_id = orders.ym_uid
```

подтверждена как основная логика атрибуции заказа к пользователю Яндекс.Метрики.

Товарные и финансовые агрегаты присоединяются только для целей:

```text
order_paid
zakaz_oformlen
```

`total_value` дополнительно исключает отмененные/возвратные статусы заказа.
Список исключенных статусов проверялся по таблице статусов заказа и считается полным на момент документирования.

## Поля

| Поле | Описание |
|---|---|
| `stat_date` | дата события / статистики Метрики |
| `client_id` | ClientID пользователя Яндекс.Метрики |
| `first_traffic_source` | источник первого визита |
| `first_visit_date` | дата первого визита |
| `first_utm_source` | UTM source первого визита |
| `first_utm_medium` | UTM medium первого визита |
| `start_url` | страница входа |
| `previous_visit_date` | дата предыдущего визита |
| `last_sign_utm_source` | UTM source последнего значимого перехода |
| `last_sign_utm_medium` | UTM medium последнего значимого перехода |
| `visits` | количество визитов |
| `pageviews` | просмотры страниц |
| `page_depth` | глубина просмотра |
| `avg_visit_duration_sec` | средняя длительность визита |
| `goal_*_reaches` | достижения целей |
| `shop_order_id` | связанный ID заказа магазина |
| `order_status` | статус заказа |
| `product_quantity` | количество товаров в заказе |
| `total_price` | сумма товаров в заказе по `order_products` |
| `total_value` | итог заказа из `orders_totals`, `code = total` |

## Обновление данных

### Метрика

- Процесс: n8n workflow `Yandex metrica stats UTM and Users`
- Расписание: ежедневно в 04:00 Europe/Moscow
- Регулярный период загрузки: вчерашний день
- Перед вставкой workflow удаляет данные за период `date1` -> `date2` из `adb.ya_metrica_users_goals_detailed`, затем вставляет актуальные строки.
- Историческая первичная загрузка была выполнена вручную с `2025-09-01`; более ранние данные Яндекс.Метрика не вернула.

### Заказы

- Процесс: PHP cron `adb.php`
- URL: `https://crm.madeindream.com/admin/index.php?route=cron/adb`
- Расписание: ежедневно в 06:30
- Таблицы [orders](../tables/orders.md), [order_products](../tables/order_products.md), [orders_totals](../tables/orders_totals.md) пересобираются через `TRUNCATE -> INSERT`.

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Метрика по пользователям достигшим цели](../charts/metrika-po-polzovatelyam-dostigshim-tseli.md) | [Поведенческий из Яндекс.Метрики](../dashboards/povedencheskiy-iz-yandex-metriki.md) | Raw table |
| [Выручка по источникам](../charts/vyruchka-po-istochnikam.md) | [Поведенческий из Яндекс.Метрики](../dashboards/povedencheskiy-iz-yandex-metriki.md), Детализация по рекламным платформам | Pivot: `SUM(total_value)` по дате и нормализованному источнику |
| [Доля трафика по источнику первого визита](../charts/dolya-trafika-po-istochniku-pervogo-vizita.md) | [Поведенческий из Яндекс.Метрики](../dashboards/povedencheskiy-iz-yandex-metriki.md) | `COUNT(first_traffic_source)` по `first_traffic_source` |

## Используется в других датасетах

| Датасет | Роль |
|---|---|
| [ad_stats_and_roi](../datasets/ad_stats_and_roi.md) | Источник атрибуции выручки к рекламным площадкам через `last_sign_utm_source` и связь с заказами |
| [mart_lid_orders](../datasets/mart_lid_orders.md) | Источник UTM/traffic source для задачи, покупателя и лида |
| [source_abadoned_orders](../datasets/source_abadoned_orders.md) | Источник последнего доступного визита и UTM перед задачей брошенного заказа |

## Вопросы

- Если в MID появятся новые статусы отмен/возвратов, нужно перепроверить список исключений для `total_value`.
