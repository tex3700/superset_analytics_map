# Dataset: rel_lid_orders

---
type: dataset
name: rel_lid_orders
superset_dataset_id: 22
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.mart_lids
  - adb.customers
  - adb.orders
used_by_charts:
  - konversiya_lid_zakaz
---

## Краткое описание

Virtual dataset считает дневное количество валидных лидов и дневное количество заказов новых клиентов, у которых есть валидный лид.

Используется для расчета конверсии:

```text
Лид -> Заказ
```

## Superset

- Dataset ID: `22`
- Dataset name: `rel_lid_orders`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/22`
- Owner: Дмитрий Пыланкин
- Main datetime column: `date`

## SQL датасета

```sql
WITH valid_leads AS (
    SELECT
        DATE(ml.lid_date_added) AS lead_date,
        c.shop_customer_id
    FROM adb.mart_lids ml
    JOIN adb.customers c ON ml.contact_id = c.contact_id
    WHERE ml.project_id NOT IN (648, 667)
      AND DATE(ml.task_date_added) BETWEEN DATE(ml.lid_date_added) - INTERVAL 2 DAY
                                      AND DATE(ml.lid_date_added) + INTERVAL 2 DAY
),
valid_orders AS (
    SELECT
        shop_customer_id,
        date_added
    FROM adb.orders
    WHERE shop_customer_id NOT IN (
        59522, 50043, 17536, 90224, 107545, 76752, 84811, 88147,
        111661, 95888, 26278, 77219, 106532
    )
    AND order_status NOT IN (
        'Отменен', 'Отмена (дублирующийся заказ)', 'Неудавшийся',
        'Возврат', 'Возврат б\\у', 'Возврат средств'
    )
),
new_customers AS (
    SELECT shop_customer_id
    FROM valid_orders
    GROUP BY shop_customer_id
    HAVING COUNT(*) = 1
),
filtered_orders AS (
    SELECT vo.date_added
    FROM valid_orders vo
    WHERE vo.shop_customer_id IN (SELECT shop_customer_id FROM new_customers)
      AND vo.shop_customer_id IN (SELECT shop_customer_id FROM valid_leads)
),
lids_agg AS (
    SELECT lead_date AS date, COUNT(*) AS count_lids
    FROM valid_leads
    GROUP BY lead_date
),
orders_agg AS (
    SELECT DATE(date_added) AS date, COUNT(*) AS count_orders
    FROM filtered_orders
    GROUP BY DATE(date_added)
)
SELECT
    date,
    SUM(count_lids) AS count_lids,
    SUM(count_orders) AS count_orders
FROM (
    SELECT date, count_lids, 0 AS count_orders FROM lids_agg
    UNION ALL
    SELECT date, 0 AS count_lids, count_orders FROM orders_agg
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
total_rows = 1732
distinct_dates = 1732
duplicate_date_rows = 0
```

## Поля

| Поле | Описание | Источник / формула |
|---|---|---|
| `date` | Дата агрегации | дата лида или заказа |
| `count_lids` | Количество валидных лидов | `COUNT(*)` из `valid_leads` |
| `count_orders` | Количество заказов новых клиентов с валидным лидом | `COUNT(*)` из `filtered_orders` |

## Логика валидного лида

Лид попадает в `valid_leads`, если:

- `mart_lids.contact_id` связан с `customers.contact_id`;
- `project_id NOT IN (648, 667)`;
- `task_date_added` находится в окне `lid_date_added - 2 дня` -> `lid_date_added + 2 дня`.

Исключенные проекты:

| `project_id` | Значение |
|---|---|
| `648` | Брошенные заказы РФ |
| `667` | Брошенные заказы KZ |

## Логика заказа

Заказ попадает в `count_orders`, если:

- заказ не относится к marketplace customer ID;
- статус заказа не отмена/возврат;
- покупатель имеет ровно один валидный заказ;
- покупатель есть среди валидных лидов.

Фильтр новых клиентов:

```text
GROUP BY shop_customer_id
HAVING COUNT(*) = 1
```

## Отличие от `con_lids_orders`

`con_lids_orders` считает заказы новых покупателей независимо от наличия валидного лида.

`rel_lid_orders` дополнительно требует связь:

```text
orders.shop_customer_id IN valid_leads.shop_customer_id
```

По SQL-логике отличие именно в фильтрации заказов через наличие валидного лида. По бизнес-интерпретации основное видимое различие проявляется в итоговом расчете показателя: этот датасет используется для конверсии "Лид -> Заказ".

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Конверсия (Лид -> Заказ)](../charts/konversiya-lid-zakaz.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `SUM(count_orders) * 100 / SUM(count_lids)` |

