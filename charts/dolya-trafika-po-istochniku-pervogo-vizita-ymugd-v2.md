# Chart: Доля трафика по источнику первого визита ymugdV2

---
type: chart
name: Доля трафика по источнику первого визита ymugdV2
superset_chart_id: 106
dataset: ya_metrica_users_goals_detailed_v2
dashboards:
  - Поведенческий из Яндекс.Метрики
purpose: comparison
---

## Назначение

Chart показывает распределение строк [ya_metrica_users_goals_detailed_v2](../datasets/ya_metrica_users_goals_detailed_v2.md) по источнику первого визита.

Важно: на v2 это не доля всего трафика Метрики. Это распределение атрибутированных заказов / order-level строк v2 по `first_traffic_source`.

## Superset

- Chart ID: `106`
- Visualization type: `funnel`
- Dataset: `adb.ya_metrica_users_goals_detailed_v2` (id `29`)
- Dashboard: `12`
- URL: `http://81.200.152.123:8088/explore/?slice_id=106`

## Настройки

| Параметр | Значение |
|---|---|
| group by | `first_traffic_source` |
| metric | `COUNT(first_traffic_source)` |
| metric label | `Количество визитов` |
| temporal filter | `stat_date = Last month` |
| row limit | `10` |
| percent calculation | `total` |

## Агрегация

Chart считает количество строк v2 по `first_traffic_source`.

