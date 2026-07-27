# Dataset: ya_metrica_users_goals_detailed_v2

---
type: dataset
name: ya_metrica_users_goals_detailed_v2
superset_dataset_id: 29
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
created_for:
  - customer_objections_roi_cac_review
  - deduplicated_order_attribution
used_by_charts:
  - metrika-po-polzovatelyam-dostigshim-tseli-ymugd-v2
  - dolya-trafika-po-istochniku-pervogo-vizita-ymugd-v2
  - vyruchka-po-istochnikam-ymugd-v2
---

## Краткое описание

Virtual dataset создан как исправленная версия [ya_metrica_users_goals_detailed](./ya_metrica_users_goals_detailed.md) для проверки и дальнейшего перехода на более устойчивую attribution-модель.

Главное отличие: датасет принудительно приводит результат к уровню `1 строка = 1 связанный заказ`, чтобы один `shop_order_id` не размножал выручку при связи Метрика -> orders.

Старая версия dataset `15` остается как диагностическая: она полезна для просмотра исходных строк Метрики и анализа того, где возникает one-to-many join.

## Superset

- Dataset ID: `29`
- Dataset name: `ya_metrica_users_goals_detailed_v2`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/29`
- Owner: Дмитрий Пыланкин
- Main datetime column: `stat_date`

## Источники данных

| Источник | Процесс загрузки | Таблицы ADB |
|---|---|---|
| Яндекс.Метрика API | `Yandex metrica stats UTM and Users` | `adb.ya_metrica_users_goals_detailed` |
| MID / CRM databases | PHP cron `adb.php` | [adb.orders](../tables/orders.md), [adb.order_products](../tables/order_products.md), [adb.orders_totals](../tables/orders_totals.md) |

## Grain

Одна строка датасета соответствует одному атрибутированному заказу:

```text
1 shop_order_id
```

Ожидаемая уникальность:

```text
shop_order_id
```

Если один заказ подходит к нескольким строкам Метрики, выбирается одна строка через `ROW_NUMBER()`.

## Логика выбора строки Метрики

Кандидаты строятся по той же fallback-связке, что и в dataset `15`:

```text
ya_metrica_users_goals_detailed.client_id = orders.ym_uid
```

Дополнительные условия:

- `stat_date` попадает в интервал `DATE(order.date_added)` -> `DATE(order.date_modified)`;
- если `date_modified` пустой, `stat_date = DATE(order.date_added)`;
- у строки Метрики есть хотя бы одна из целей: `order_paid`, `add_to_cart`, `ecommerce`, `quick_order_registration`, `zakaz_oformlen`.

Если для одного `shop_order_id` найдено несколько кандидатов, выбирается строка с приоритетом:

| Priority | Цель |
|---:|---|
| `1` | `order_paid` |
| `2` | `zakaz_oformlen` |
| `3` | `ecommerce` |
| `4` | `quick_order_registration` |
| `5` | `add_to_cart` |

При равном приоритете выбирается более поздняя `stat_date`, затем больший `ym_row_id`.

## Финансовая логика

`total_value` заполняется только если:

- статус заказа не входит в список отмен/возвратов;
- выбранная attribution-цель входит в `order_paid`, `zakaz_oformlen`.

Для целей `ecommerce`, `quick_order_registration`, `add_to_cart` заказ может остаться в датасете как диагностическая связь, но `total_value` будет `NULL`.

## Поля качества

| Поле | Назначение |
|---|---|
| `join_quality` | Сейчас всегда `fallback_ym_uid_date_goal`; показывает, что связь не exact, а fallback по `ym_uid`, датам и целям |
| `attribution_priority` | Приоритет цели, по которой выбрана строка Метрики |
| `attribution_goal` | Цель Метрики, которая использована как attribution-сигнал |
| `ym_row_id` | ID исходной строки `adb.ya_metrica_users_goals_detailed` |

## Почему создан v2

Проверка customer objections подтвердила, что dataset `15` размножает строки относительно уникальных заказов:

```text
joined_rows = 4514
distinct_orders = 3075
extra_rows_vs_orders = 1439
```

`v2` не делает attribution exact, но устраняет переучет одного заказа внутри downstream-расчетов.

## Используется в чартах

| Chart ID | Чарт | Назначение |
|---:|---|---|
| `105` | [Метрика по пользователям достигшим цели ymugdV2](../charts/metrika-po-polzovatelyam-dostigshim-tseli-ymugd-v2.md) | Диагностическая raw table по order-level attribution строкам |
| `106` | [Доля трафика по источнику первого визита ymugdV2](../charts/dolya-trafika-po-istochniku-pervogo-vizita-ymugd-v2.md) | Распределение атрибутированных заказов по `first_traffic_source` |
| `107` | [Выручка по источникам ymugdV2](../charts/vyruchka-po-istochnikam-ymugd-v2.md) | `SUM(total_value)` по дате и нормализованному источнику |

Важно: эти charts показывают не всю аудиторию Метрики, а только строки, которые прошли attribution-связку с заказом.

## Использование

Пока dataset предназначен для сверки и построения новых comparison charts.

Не заменяет старые чарты автоматически.
