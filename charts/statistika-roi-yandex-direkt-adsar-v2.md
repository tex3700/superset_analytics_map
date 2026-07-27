# Chart: Статистика ROI Яндекс Директ adsarV2

---
type: chart
name: Статистика ROI Яндекс Директ adsarV2
superset_chart_id: 98
dataset: ad_stats_and_roi_v2
dashboards:
  - Детализация по рекламным платформам
purpose: comparison
---

## Назначение

Mixed time series показывает расходы, выручку и ROI только для `Yandex.Direct` на базе [ad_stats_and_roi_v2](../datasets/ad_stats_and_roi_v2.md).

## Superset

- Chart ID: `98`
- Visualization type: `mixed_timeseries`
- Dataset: `adb.ad_stats_and_roi_v2` (id `30`)
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=98`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `stat_date` |
| time grain | `P1D` |
| primary metrics | `MAX(spend)`, `MAX(revenue)` |
| secondary metric | `MAX(roi_percent)` |
| temporal filter | `stat_date = Last month` |
| platform filter | `platform = Yandex.Direct` |

## Агрегация

Так как grain dataset `30` уже `stat_date + platform`, для выбранного `Yandex.Direct` используются `MAX(spend)`, `MAX(revenue)` и `MAX(roi_percent)` как способ вывести дневное значение строки.

