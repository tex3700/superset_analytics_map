# Dataset: ad_stats_and_roi

---
type: dataset
name: ad_stats_and_roi
superset_dataset_id: 18
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.ad_spend_daily
  - adb.ya_metrica_users_goals_detailed
  - adb.orders
  - adb.order_products
  - adb.orders_totals
used_by_charts:
  - statisticheskie_dannye_po_reklamnym_ploshchadkam
  - roi_po_ploshchadkam
  - vyruchka_po_ploshchadkam
  - reklamnye_rashody_za_period
  - vyruchka_s_ploshchadok_za_period
  - usrednennyy_roi_po_vsem_ploshchadkam
  - statistika_roi_yandex_direkt
---

## Краткое описание

Virtual dataset собирает расходы, выручку и ROI по рекламным площадкам.

Расходы берутся из `adb.ad_spend_daily`. Выручка рассчитывается через связку данных Яндекс.Метрики, заказов и итогов заказов:

```text
ya_metrica_users_goals_detailed
  -> orders
  -> orders_totals
```

Статус после разбора customer objections: dataset оставлен для сверки со старой методикой. Для сравнения создан [ad_stats_and_roi_v2](./ad_stats_and_roi_v2.md), id `30`, где выручка дедуплицируется на уровне `shop_order_id`.

## Superset

- Dataset ID: `18`
- Dataset name: `ad_stats_and_roi`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/18`
- Owner: Дмитрий Пыланкин
- Main datetime column: `stat_date`

## Источники данных

| Источник | Процесс загрузки | Таблицы ADB |
|---|---|---|
| Yandex Direct через Yandex Metrika API | `Advertisings stats` | `adb.ad_spend_daily` |
| Yandex Metrika API | `Yandex metrica stats UTM and Users` | `adb.ya_metrica_users_goals_detailed` |
| MID / CRM databases | `ADB cron PHP` | `adb.orders`, `adb.order_products`, `adb.orders_totals` |

## SQL датасета

```sql
WITH
spend_data AS (
    SELECT
        stat_date,
        advertising AS platform,
        SUM(ad_cost) AS total_spend
    FROM adb.ad_spend_daily
    GROUP BY stat_date, advertising
),
order_products_agg AS (
    SELECT
        shop_order_id,
        SUM(quantity) AS product_quantity,
        SUM(price) AS total_price
    FROM order_products
    GROUP BY shop_order_id
),
orders_totals_agg AS (
    SELECT
        shop_order_id,
        MAX(value) AS total_value
    FROM orders_totals
    WHERE code = 'total'
    GROUP BY shop_order_id
),
revenue_base AS (
    SELECT
        y.stat_date,
        y.last_sign_utm_source AS utm_source_raw,
        ot.total_value
    FROM ya_metrica_users_goals_detailed y
    LEFT JOIN orders o
        ON y.client_id = o.ym_uid
        AND (
            (o.date_modified IS NOT NULL
             AND y.stat_date BETWEEN DATE(o.date_added) AND DATE(o.date_modified))
            OR
            (o.date_modified IS NULL
             AND y.stat_date = DATE(o.date_added))
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
        AND o.order_status NOT IN ('Отменен', 'Отмена (дублирующийся заказ)', 'Неудавшийся', 'Возврат', 'Возврат б\у', 'Возврат средств')
    WHERE ot.total_value IS NOT NULL
),
revenue_agg AS (
    SELECT
        stat_date,
        CASE
            WHEN LOWER(utm_source_raw) IN ('ya', 'yandex') THEN 'Yandex.Direct'
            WHEN LOWER(utm_source_raw) IN ('admitad') THEN 'Admitad'
            WHEN LOWER(utm_source_raw) IN ('email') THEN 'Email'
            WHEN LOWER(utm_source_raw) IN ('instagram') THEN 'Instagram'
            WHEN LOWER(utm_source_raw) IN ('telegram') THEN 'Telegram'
            WHEN LOWER(utm_source_raw) IN ('null', 'undefined', 'none', '') OR utm_source_raw IS NULL THEN NULL
            ELSE utm_source_raw
        END AS platform,
        SUM(total_value) AS total_revenue
    FROM revenue_base
    WHERE utm_source_raw IS NOT NULL
      AND TRIM(utm_source_raw) != ''
    GROUP BY stat_date, platform
),
combined AS (
    SELECT
        s.stat_date,
        s.platform,
        s.total_spend AS spend,
        COALESCE(r.total_revenue, 0) AS revenue
    FROM spend_data s
    LEFT JOIN revenue_agg r
        ON s.stat_date = r.stat_date AND s.platform = r.platform
    UNION
    SELECT
        r.stat_date,
        r.platform,
        COALESCE(s.total_spend, 0) AS spend,
        r.total_revenue AS revenue
    FROM revenue_agg r
    LEFT JOIN spend_data s
        ON r.stat_date = s.stat_date AND r.platform = s.platform
    WHERE s.stat_date IS NULL
)
SELECT
    stat_date,
    platform,
    ROUND(spend, 2) AS spend,
    ROUND(revenue, 2) AS revenue,
    CASE
        WHEN spend = 0 THEN NULL
        ELSE ROUND(((revenue - spend) / spend) * 100, 2)
    END AS roi_percent
FROM combined
WHERE platform IS NOT NULL
ORDER BY stat_date DESC, revenue;
```

## Grain

Одна строка датасета соответствует:

```text
1 день + 1 рекламная площадка / platform
```

Ожидаемая уникальность:

```text
stat_date + platform
```

## Поля

| Поле | Тип | Описание | Источник / формула |
|---|---|---|---|
| `stat_date` | DATE | Дата статистики | `spend_data.stat_date` или `revenue_agg.stat_date` |
| `platform` | VARCHAR | Рекламная площадка | `ad_spend_daily.advertising` или нормализованный `last_sign_utm_source` |
| `spend` | DOUBLE | Расход на рекламу | `SUM(ad_spend_daily.ad_cost)` |
| `revenue` | DECIMAL | Выручка от рекламы | `SUM(orders_totals.value)` после атрибуции к источнику |
| `roi_percent` | DOUBLE | ROI, % | `((revenue - spend) / spend) * 100`; если `spend = 0`, то `NULL` |

## Метрики в датасете

В Superset определена только стандартная метрика:

| Метрика | Формула | Описание |
|---|---|---|
| `count` | `COUNT(*)` | Количество строк |

## Агрегации внутри датасета

Dataset выполняет несколько уровней агрегации:

1. Расходы: `SUM(ad_cost) GROUP BY stat_date, advertising`.
2. Выручка: `MAX(orders_totals.value)` по заказу, затем `SUM(total_value)` по `stat_date + platform`.
3. ROI на уровне строки: `((revenue - spend) / spend) * 100`.
4. Объединение расходов и выручки: эмуляция `FULL OUTER JOIN` через `LEFT JOIN + UNION`.

## Неиспользуемая CTE `order_products_agg`

В SQL есть CTE `order_products_agg` и `LEFT JOIN` к ней, но поля `product_quantity` и `total_price` не используются в финальной выборке и не влияют на расчет `spend`, `revenue` или `roi_percent`.

Текущая договоренность: оставляем эту часть SQL как есть. CTE была оставлена как заготовка для будущих товарных метрик по рекламным площадкам.

## Атрибуция выручки

Заказ связывается с пользователем Метрики по логике:

```text
ya_metrica_users_goals_detailed.client_id = orders.ym_uid
```

Дополнительно дата события Метрики должна попадать в период заказа:

```text
stat_date BETWEEN DATE(order.date_added) AND DATE(order.date_modified)
```

Если `date_modified` пустой:

```text
stat_date = DATE(order.date_added)
```

Выручка учитывается только для пользователей, у которых есть достижения целей заказа/покупки, и только если статус заказа не входит в список отмен/возвратов.

## Нормализация площадок

| Raw `last_sign_utm_source` | Platform |
|---|---|
| `ya`, `yandex` | `Yandex.Direct` |
| `admitad` | `Admitad` |
| `email` | `Email` |
| `instagram` | `Instagram` |
| `telegram` | `Telegram` |
| `null`, `undefined`, `none`, empty, SQL `NULL` | исключаются |
| другое значение | исходное значение |

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Статистические данные по рекламным площадкам](../charts/statisticheskie-dannye-po-reklamnym-ploshchadkam.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | Raw table, без агрегации |
| [ROI по площадкам](../charts/roi-po-ploshchadkam.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | `SUM(spend)`, `SUM(revenue)`, ROI через суммы |
| [Выручка по площадкам](../charts/vyruchka-po-ploshchadkam.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | `SUM(revenue)` по `platform` |
| [Рекламные расходы за период](../charts/reklamnye-rashody-za-period.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | `SUM(spend)` |
| [Выручка с площадок за период](../charts/vyruchka-s-ploshchadok-za-period.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | `SUM(revenue)` |
| [Усредненный ROI по всем площадкам](../charts/usrednennyy-roi-po-vsem-ploshchadkam.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | `AVG(roi_percent)`, нужен именно средний процент |
| [Статистика ROI Яндекс Директ](../charts/statistika-roi-yandex-direkt.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | `MAX(spend)`, `MAX(revenue)`, `MAX(roi_percent)` для `Yandex.Direct` |

## Проверки качества

- В датасете должна быть не более одной строки на `stat_date + platform`.
- `spend` и `revenue` не должны быть отрицательными.
- `roi_percent` должен быть `NULL`, если `spend = 0`.
- Для площадок с выручкой, но без расходов, ROI в датасете будет `NULL`.
