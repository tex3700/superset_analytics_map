# Dataset: analytics_data_quality_daily

---
type: dataset
name: analytics_data_quality_daily
superset_dataset_id: 32
superset_type: physical
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.analytics_data_quality_daily
  - n8n:analytics_data_quality
used_by_dashboards:
  - kachestvo-dannyh-analitiki
---

## Краткое описание

Physical dataset Superset над таблицей [adb.analytics_data_quality_daily](../tables/analytics_data_quality_daily.md).

Dataset хранит ежедневные результаты диагностических проверок качества аналитики за rolling window 90 дней. Таблица заполняется n8n workflow `Analytics data quality`, который вызывает MySQL procedure `adb.refresh_analytics_data_quality_daily`.

## Superset

- Dataset ID: `32`
- Dataset name: `analytics_data_quality_daily`
- Dataset type: physical
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/32`
- Owner: Дмитрий Пыланкин
- Main datetime column: `check_date`

## Grain

Одна строка соответствует:

```text
1 дата проверки + 1 check_name
```

Ожидаемая уникальность в таблице:

```text
check_date + check_name
```

## Поля

| Поле | Описание |
|---|---|
| `check_date` | Дата, за которую записан результат проверки |
| `check_name` | Техническое имя проверки |
| `check_group` | Группа проверки: например `orders`, `attribution`, `revenue` |
| `metric_value` | Числовое значение проверки |
| `metric_text` | Текстовое значение, если нужно |
| `status` | Статус проверки: `ok`, `warning`, `info` |
| `comment` | Человекочитаемый комментарий |
| `created_at` | Время записи результата |

## Основные checks

| check_name | Смысл |
|---|---|
| `orders_total_90d` | Всего заказов за последние 90 дней |
| `orders_without_ym_uid_90d` | Заказы без `ym_uid` за последние 90 дней |
| `unmatched_orders_90d` | Заказы без связи с Метрикой |
| `matched_orders_90d` | Заказы со связью Метрика -> заказ |
| `attribution_match_rate_90d` | Доля заказов, связанных с Метрикой |
| `attributed_revenue_90d` | Атрибутированная выручка |
| `orders_total_revenue_90d` | Общая сумма заказов |
| `attributed_revenue_share_90d` | Доля атрибутированной выручки от общей суммы заказов |

## Используется в чартах

Dataset используется на dashboard [Качество данных аналитики](../dashboards/kachestvo-dannyh-analitiki.md), chart IDs `109`-`118`.

