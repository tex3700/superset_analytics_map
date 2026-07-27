# Chart: Усредненный ROI по всем площадкам adsarV2

---
type: chart
name: Усредненный ROI по всем площадкам adsarV2
superset_chart_id: 100
dataset: ad_stats_and_roi_v2
dashboards:
  - Детализация по рекламным платформам
purpose: comparison
---

## Назначение

Big number показывает средний процент ROI по строкам [ad_stats_and_roi_v2](../datasets/ad_stats_and_roi_v2.md).

Это сознательно `AVG(roi_percent)`, а не ROI через суммы. Такая логика была подтверждена как нужная для этого chart.

## Superset

- Chart ID: `100`
- Visualization type: `big_number`
- Dataset: `adb.ad_stats_and_roi_v2` (id `30`)
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=100`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `stat_date` |
| time grain | `P1M` |
| metric | `AVG(roi_percent)` |
| temporal filter | `stat_date = Last quarter` |
| comparison | `compare_lag = 1` |

## Агрегация

```text
AVG(roi_percent)
```

