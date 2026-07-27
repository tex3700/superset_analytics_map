# Chart: ROI по площадкам adsarV2

---
type: chart
name: ROI по площадкам adsarV2
superset_chart_id: 99
dataset: ad_stats_and_roi_v2
dashboards:
  - Детализация по рекламным платформам
purpose: comparison
---

## Назначение

Mixed time series сравнивает расходы, выручку и ROI по площадкам на базе [ad_stats_and_roi_v2](../datasets/ad_stats_and_roi_v2.md).

## Superset

- Chart ID: `99`
- Visualization type: `mixed_timeseries`
- Dataset: `adb.ad_stats_and_roi_v2` (id `30`)
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=99`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `platform` |
| primary metrics | `SUM(spend)`, `SUM(revenue)` |
| secondary metric | SQL ROI через суммы |
| primary temporal filter | `stat_date = Last month` |
| secondary temporal filter | `stat_date = Last month` |
| row limit | `10000` |

SQL-метрика ROI:

```sql
CASE
  WHEN SUM(spend) = 0 THEN NULL
  ELSE (SUM(revenue) - SUM(spend)) / SUM(spend) * 100
END
```
