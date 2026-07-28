# Dashboard: Качество данных аналитики

---
type: dashboard
name: Качество данных аналитики
superset_dashboard_id: 16
datasets:
  - analytics_data_quality_daily
charts:
  - dinamika-zakazy-bez-ym-uid-90d
  - zakazy-bez-ym-uid-90d
  - vsego-zakazov-90d
  - ne-svyazannye-s-metrikoy-zakazy-90d
  - koeffitsient-sootvetstviya-atributsii
  - dinamika-ne-svyazannye-s-metrikoy-zakazy-90d
  - atributirovannaya-vyruchka-90d
  - dinamika-kachestva
  - dolya-atributirovannoy-vyruchki-ot-obshchey-summy-zakazov
  - istoriya-proverok
---

## Назначение

Dashboard показывает состояние качества данных аналитики: полноту `ym_uid`, долю заказов, связанных с Метрикой, количество unmatched-заказов и контроль выручки за последние 90 дней.

## Superset

- Dashboard ID: `16`
- Dashboard title: `Качество данных аналитики`
- URL: `http://81.200.152.123:8088/superset/dashboard/16/`
- Owner: Дмитрий Пыланкин
- Published: yes

## Источник данных

Все charts используют physical dataset [analytics_data_quality_daily](../datasets/analytics_data_quality_daily.md), id `32`.

Таблица заполняется workflow [Analytics data quality](../ingestion/analytics-data-quality.md) через procedure `adb.refresh_analytics_data_quality_daily`.

## Чарты

| Chart ID | Чарт | Тип | Проверка |
|---:|---|---|---|
| `109` | [Динамика - Заказы без ym_uid 90d](../charts/dinamika-zakazy-bez-ym-uid-90d.md) | bar | `orders_without_ym_uid_90d` |
| `110` | [Заказы без ym_uid 90d](../charts/zakazy-bez-ym-uid-90d.md) | big number | `orders_without_ym_uid_90d` |
| `111` | [Всего заказов 90d](../charts/vsego-zakazov-90d.md) | big number | `orders_total_90d` |
| `112` | [Не связанные с метрикой заказы 90d](../charts/ne-svyazannye-s-metrikoy-zakazy-90d.md) | big number | `unmatched_orders_90d` |
| `113` | [Коэффициент соответствия атрибуции](../charts/koeffitsient-sootvetstviya-atributsii.md) | big number | `attribution_match_rate_90d` |
| `114` | [Динамика - Не связанные с метрикой заказы 90d](../charts/dinamika-ne-svyazannye-s-metrikoy-zakazy-90d.md) | bar | `unmatched_orders_90d` |
| `115` | [Атрибутированная выручка 90d](../charts/atributirovannaya-vyruchka-90d.md) | big number | `attributed_revenue_90d` |
| `116` | [Динамика качества](../charts/dinamika-kachestva.md) | line | все `check_name` |
| `117` | [Доля атрибутированной выручки от общей суммы заказов](../charts/dolya-atributirovannoy-vyruchki-ot-obshchey-summy-zakazov.md) | big number | `attributed_revenue_share_90d` |
| `118` | [История проверок](../charts/istoriya-proverok.md) | table | все checks |

