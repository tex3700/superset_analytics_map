# Dataset: source_abadoned_orders

---
type: dataset
name: source_abadoned_orders
superset_dataset_id: 23
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.mart_lids
  - adb.ym_abandoned_orders
  - adb.orders
  - adb.ya_metrica_users_goals_detailed
used_by_charts:
  - broshennye_zakazy_s_privyazkoy_k_istochniku
  - istochniki_po_broshennym_zakazam
---

## Краткое описание

Virtual dataset показывает брошенные заказы из CRM-проектов `648` и `667` с привязкой к источнику Яндекс.Метрики.

Источники `ym_uid`:

1. `adb.ym_abandoned_orders`;
2. fallback: `adb.orders`.

После получения `ym_uid` датасет берет последнюю строку `ya_metrica_users_goals_detailed` с `stat_date <= date_task_added` и строит поле `utm_source`.

## Superset

- Dataset ID: `23`
- Dataset name: `source_abadoned_orders`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/23`
- Owner: Дмитрий Пыланкин
- Main datetime column: `lid_date_added`

## SQL датасета

```sql
SELECT
  base.contact_id,
  base.task_id,
  base.lid_date_added AS date_lid_added,
  base.task_date_added AS date_task_added,
  base.project_id,
  base.project_name,
  base.task_status,
  base.task_property,
  base.lid_email,
  base.lid_phone,
  base.ym_uid,
  ymugd.stat_date,
  ymugd.client_id,
  ymugd.previous_visit_date,
  ymugd.first_visit_date,
  CASE
    WHEN ymugd.last_sign_utm_source IS NOT NULL AND ymugd.last_sign_utm_source != 'null'
      THEN CONCAT(ymugd.last_sign_utm_source, ' | ', ymugd.last_sign_utm_medium)
    WHEN ymugd.first_utm_source IS NOT NULL AND ymugd.first_utm_source != 'null'
      THEN CONCAT(ymugd.first_utm_source, ' | ', ymugd.first_utm_medium)
    ELSE ymugd.first_traffic_source
  END AS utm_source
FROM (
  SELECT
    ml.contact_id,
    ml.task_id,
    ml.lid_date_added,
    ml.task_date_added,
    ml.project_id,
    ml.project_name,
    ml.task_status,
    ml.task_property,
    ml.lid_email,
    ml.lid_phone,
    COALESCE(yao.ym_uid, o.ym_uid) AS ym_uid
  FROM adb.mart_lids ml
  LEFT JOIN adb.ym_abandoned_orders yao
    ON ml.task_id = yao.crm_task_id
    AND ml.lid_phone = yao.phone
  LEFT JOIN adb.orders o
    ON ml.task_id = o.crm_task_id
    AND ml.lid_phone = o.telephone
  WHERE ml.project_id IN (648, 667)
) AS base
LEFT JOIN adb.ya_metrica_users_goals_detailed ymugd
  ON ymugd.client_id = base.ym_uid
  AND ymugd.stat_date = (
    SELECT MAX(ymugd2.stat_date)
    FROM adb.ya_metrica_users_goals_detailed ymugd2
    WHERE ymugd2.client_id = base.ym_uid
      AND ymugd2.stat_date <= base.task_date_added
  )
ORDER BY base.contact_id DESC;
```

## Grain

Ожидаемый grain:

```text
1 CRM contact + 1 CRM task брошенного заказа
```

Базовая часть датасета проверена в ADB:

```text
total_rows = 4601
distinct_contact_task = 4601
duplicate_contact_task_rows = 0
```

## Логика проектов

Датасет берет только лиды из проектов:

| `project_id` | Значение |
|---|---|
| `648` | Брошенные заказы РФ |
| `667` | Брошенные заказы KZ |

## Логика источника

Приоритет определения `utm_source`:

1. Если есть `last_sign_utm_source` и он не равен строке `null`, используется:

```text
last_sign_utm_source | last_sign_utm_medium
```

2. Иначе, если есть `first_utm_source` и он не равен строке `null`, используется:

```text
first_utm_source | first_utm_medium
```

3. Иначе используется:

```text
first_traffic_source
```

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Брошенные заказы с привязкой к источнику](../charts/broshennye-zakazy-s-privyazkoy-k-istochniku.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | Raw table |
| [Источники по брошенным заказам](../charts/istochniki-po-broshennym-zakazam.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `COUNT(*)` по нормализованному `utm_source` |

## Примечание

- Название с опечаткой `source_abadoned_orders` оставляем как в Superset.
