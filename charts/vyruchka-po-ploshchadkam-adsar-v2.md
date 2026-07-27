# Chart: Выручка по площадкам adsarV2

---
type: chart
name: Выручка по площадкам adsarV2
superset_chart_id: 104
dataset: ad_stats_and_roi_v2
dashboards:
  - Детализация по рекламным платформам
purpose: comparison
---

## Назначение

Sunburst chart показывает распределение выручки по рекламным площадкам на базе [ad_stats_and_roi_v2](../datasets/ad_stats_and_roi_v2.md).

## Superset

- Chart ID: `104`
- Visualization type: `sunburst_v2`
- Dataset: `adb.ad_stats_and_roi_v2` (id `30`)
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=104`

## Настройки

| Параметр | Значение |
|---|---|
| columns | `platform` |
| metric | `SUM(revenue)` |
| temporal filter | `stat_date = Last month` |
| row limit | `10000` |

## Агрегация

```text
SUM(revenue) по platform
```

