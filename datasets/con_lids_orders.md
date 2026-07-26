# Dataset: con_lids_orders

---
type: dataset
name: con_lids_orders
superset_dataset_id: 21
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.mart_lids
  - adb.orders
used_by_charts:
  - otnoshenie_zakazy_lidy
---

## Краткое описание

Virtual dataset считает дневное количество лидов и дневное количество заказов для расчета отношения `Заказы / Лиды`.

Лиды берутся из `adb.mart_lids`, заказы из `adb.orders`.

## Superset

- Dataset ID: `21`
- Dataset name: `con_lids_orders`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/21`
- Owner: Дмитрий Пыланкин
- Main datetime column: `date`

## SQL датасета

```sql
WITH lids AS (
    SELECT
        DATE(lid_date_added) AS date,
        COUNT(*) AS count_lids
    FROM adb.mart_lids
    WHERE DATE(task_date_added) BETWEEN DATE(lid_date_added) - INTERVAL 2 DAY
                                    AND DATE(lid_date_added) + INTERVAL 2 DAY
      AND project_id NOT IN (648, 667)
    GROUP BY DATE(lid_date_added)
),
orders AS (
    SELECT DATE(date_added) AS date, COUNT(*) AS count_orders
    FROM adb.orders o1
    WHERE shop_customer_id NOT IN (
        59522, 50043, 17536, 90224, 107545, 76752, 84811, 88147,
        111661, 95888, 26278, 77219, 106532
    )
      AND order_status NOT IN (
        'Отменен',
        'Отмена (дублирующийся заказ)',
        'Неудавшийся',
        'Возврат',
        'Возврат б\\у',
        'Возврат средств'
    )
      AND shop_customer_id IN (
          SELECT shop_customer_id
          FROM adb.orders
          WHERE shop_customer_id NOT IN (
              59522, 50043, 17536, 90224, 107545, 76752, 84811, 88147,
              111661, 95888, 26278, 77219, 106532
          )
            AND order_status NOT IN (
              'Отменен',
              'Отмена (дублирующийся заказ)',
              'Неудавшийся',
              'Возврат',
              'Возврат б\\у',
              'Возврат средств'
          )
          GROUP BY shop_customer_id
          HAVING COUNT(*) = 1
      )
    GROUP BY DATE(date_added)
)
SELECT
    date,
    SUM(count_lids) AS count_lids,
    SUM(count_orders) AS count_orders
FROM (
    SELECT
        l.date,
        l.count_lids,
        COALESCE(o.count_orders, 0) AS count_orders
    FROM lids l
    LEFT JOIN orders o ON l.date = o.date

    UNION ALL

    SELECT
        o.date,
        0 AS count_lids,
        o.count_orders
    FROM orders o
    WHERE NOT EXISTS (
        SELECT 1 FROM lids l WHERE l.date = o.date
    )
) combined
GROUP BY date
ORDER BY date DESC;
```

## Grain

Одна строка соответствует:

```text
1 день
```

Уникальность проверена в ADB:

```text
total_rows = 3807
distinct_dates = 3807
duplicate_date_rows = 0
```

## Поля

| Поле | Описание | Источник / формула |
|---|---|---|
| `date` | Дата агрегации | `DATE(lid_date_added)` или `DATE(orders.date_added)` |
| `count_lids` | Количество лидов | `COUNT(*)` из `mart_lids` по дате |
| `count_orders` | Количество заказов | `COUNT(*)` из `orders` по дате |

## Логика подсчета лидов

Лид попадает в расчет, если:

- `task_date_added` находится в окне `lid_date_added - 2 дня` -> `lid_date_added + 2 дня`;
- `project_id NOT IN (648, 667)`.

Исключенные проекты:

| `project_id` | Значение |
|---|---|
| `648` | Брошенные заказы РФ |
| `667` | Брошенные заказы KZ |

## Логика подсчета заказов

Заказы считаются по дате `orders.date_added`.

Исключаются:

- marketplace customer ID, описанные в [mart_lid_orders](../datasets/mart_lid_orders.md);
- отмененные/возвратные статусы заказа;
- покупатели, у которых больше одного валидного заказа.

Фильтр:

```text
GROUP BY shop_customer_id
HAVING COUNT(*) = 1
```

То есть `count_orders` здесь считается только по новым покупателям с единственным валидным заказом.

Это намеренная бизнес-логика: для отношения `Заказы / Лиды` важно считать именно новых лидов/новых клиентов. Если у клиента больше одного валидного заказа, он уже считается лояльным покупателем и не должен попадать в `count_orders` этого датасета.

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Отношение Заказы - Лиды](../charts/otnoshenie-zakazy-lidy.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `SUM(count_orders) / SUM(count_lids)` |


