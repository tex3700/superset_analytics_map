# Chart: Выручка по площадкам

---
type: chart
name: Выручка по площадкам
superset_chart_id: 51
dataset: ad_stats_and_roi
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Sunburst показывает распределение выручки от рекламы по площадкам.

## Superset

- Chart ID: `51`
- Visualization type: `sunburst_v2`
- Dataset: `adb.ad_stats_and_roi`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=51`

## Метрика

| Метрика | Формула |
|---|---|
| `SUM(Выручка от рекламы)` | `SUM(revenue)` |

## Группировки

```text
platform
```

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last month` |

## Агрегация в чарте

Чарт суммирует `revenue` по `platform`.

## Особенности

- Row limit: `10000`.
- Сортировка по метрике включена.
- Labels показывают ключ площадки.

