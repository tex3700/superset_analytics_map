# Dataset: ad_stats_and_roi_v2

---
type: dataset
name: ad_stats_and_roi_v2
superset_dataset_id: 30
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.ad_spend_daily
  - adb.ya_metrica_users_goals_detailed
  - adb.orders
  - adb.orders_totals
created_for:
  - customer_objections_roi_cac_review
  - roi_revenue_overcount_fix
used_by_charts:
  - statistika-roi-yandex-direkt-adsar-v2
  - roi-po-ploshchadkam-adsar-v2
  - usrednennyy-roi-po-vsem-ploshchadkam-adsar-v2
  - vyruchka-s-ploshchadok-za-period-adsar-v2
  - reklamnye-rashody-za-period-adsar-v2
  - statisticheskie-dannye-po-reklamnym-ploshchadkam-adsar-v2
  - vyruchka-po-ploshchadkam-adsar-v2
---

## Краткое описание

Virtual dataset создан как исправленная версия [ad_stats_and_roi](./ad_stats_and_roi.md) для сравнения старого и нового расчета рекламной выручки / ROI.

Главное отличие: перед агрегацией выручки датасет дедуплицирует attribution до уровня заказа, чтобы один `shop_order_id` попадал в выручку не более одного раза.

Старая версия dataset `18` остается в Superset для сверки с новой методикой.

## Superset

- Dataset ID: `30`
- Dataset name: `ad_stats_and_roi_v2`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/30`
- Owner: Дмитрий Пыланкин
- Main datetime column: `stat_date`

## Источники данных

| Источник | Процесс загрузки | Таблицы ADB |
|---|---|---|
| Yandex Direct через Yandex Metrika API | `Advertisings stats` | `adb.ad_spend_daily` |
| Yandex Metrika API | `Yandex metrica stats UTM and Users` | `adb.ya_metrica_users_goals_detailed` |
| MID / CRM databases | PHP cron `adb.php` | [adb.orders](../tables/orders.md), [adb.orders_totals](../tables/orders_totals.md) |

## Grain

Одна строка финального датасета соответствует:

```text
1 день + 1 рекламная площадка / platform
```

Ожидаемая уникальность:

```text
stat_date + platform
```

Внутри SQL есть промежуточный order-level слой `attributed_orders`, где ожидаемая уникальность:

```text
shop_order_id
```

## Логика расчета

### Расходы

Расходы берутся из `adb.ad_spend_daily`:

```sql
SUM(ad_cost) GROUP BY stat_date, advertising
```

### Выручка

Выручка строится через fallback-связку:

```text
ya_metrica_users_goals_detailed.client_id = orders.ym_uid
```

Далее выбирается одна attribution-строка на заказ через:

```sql
ROW_NUMBER() OVER (
  PARTITION BY shop_order_id
  ORDER BY attribution_priority, stat_date DESC, ym_row_id DESC
)
```

Приоритет целей:

| Priority | Цель |
|---:|---|
| `1` | `order_paid` |
| `2` | `zakaz_oformlen` |
| `3` | `ecommerce` |
| `4` | `quick_order_registration` |
| `5` | `add_to_cart` |

Финансовая выручка учитывается только для выбранных целей `order_paid` и `zakaz_oformlen`, если статус заказа не отмена/возврат.

### Нормализация площадок

| Raw `last_sign_utm_source` | Platform |
|---|---|
| `ya`, `yandex` | `Yandex.Direct` |
| `admitad` | `Admitad` |
| `email` | `Email` |
| `instagram` | `Instagram` |
| `telegram` | `Telegram` |
| `null`, `undefined`, `none`, empty, SQL `NULL` | исключаются |
| другое значение | исходное значение |

### ROI

```text
roi_percent = ((revenue - spend) / spend) * 100
```

Если `spend = 0`, `roi_percent = NULL`.

Дополнительное поле:

| Поле | Значение |
|---|---|
| `attribution_model` | `fallback_ym_uid_dedup_order` |

## Почему создан v2

Проверка старого dataset `18` подтвердила переучет выручки:

```text
revenue_rows = 914
distinct_revenue_orders = 743
extra_rows_vs_orders = 171
attributed_revenue_sum = 10824988.41
distinct_order_revenue_sum = 9275182.08
possible_overcount = 1549806.33
```

`v2` исправляет именно переучет одного заказа. При этом модель остается fallback attribution, а не exact payment attribution.

## Используется в чартах

| Chart ID | Чарт | Как агрегирует данные |
|---:|---|---|
| `98` | [Статистика ROI Яндекс Директ adsarV2](../charts/statistika-roi-yandex-direkt-adsar-v2.md) | `MAX(spend)`, `MAX(revenue)`, `MAX(roi_percent)` для `Yandex.Direct` |
| `99` | [ROI по площадкам adsarV2](../charts/roi-po-ploshchadkam-adsar-v2.md) | `SUM(spend)`, `SUM(revenue)`, ROI через суммы |
| `100` | [Усредненный ROI по всем площадкам adsarV2](../charts/usrednennyy-roi-po-vsem-ploshchadkam-adsar-v2.md) | `AVG(roi_percent)` |
| `101` | [Выручка с площадок за период adsarV2](../charts/vyruchka-s-ploshchadok-za-period-adsar-v2.md) | `SUM(revenue)` |
| `102` | [Рекламные расходы за период adsarV2](../charts/reklamnye-rashody-za-period-adsar-v2.md) | `SUM(spend)` |
| `103` | [Статистические данные по рекламным площадкам adsarV2](../charts/statisticheskie-dannye-po-reklamnym-ploshchadkam-adsar-v2.md) | Raw table |
| `104` | [Выручка по площадкам adsarV2](../charts/vyruchka-po-ploshchadkam-adsar-v2.md) | `SUM(revenue)` по `platform` |

## Использование

Пока dataset предназначен для новых comparison charts рядом со старым [ad_stats_and_roi](./ad_stats_and_roi.md).

После сверки старых и новых графиков можно принять решение, какие dashboard charts переводить на `ad_stats_and_roi_v2`.
