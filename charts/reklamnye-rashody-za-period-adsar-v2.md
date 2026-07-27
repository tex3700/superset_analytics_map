# Chart: Рекламные расходы за период adsarV2

---
type: chart
name: Рекламные расходы за период adsarV2
superset_chart_id: 102
dataset: ad_stats_and_roi_v2
dashboards:
  - Детализация по рекламным платформам
purpose: comparison
---

## Назначение

Big number показывает суммарные рекламные расходы из [ad_stats_and_roi_v2](../datasets/ad_stats_and_roi_v2.md) за выбранный период.

## Superset

- Chart ID: `102`
- Visualization type: `big_number`
- Dataset: `adb.ad_stats_and_roi_v2` (id `30`)
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=102`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `stat_date` |
| time grain | `P1M` |
| metric | `SUM(spend)` |
| temporal filter | `stat_date = Last quarter` |
| comparison | `compare_lag = 1` |

## Агрегация

```text
SUM(spend)
```

