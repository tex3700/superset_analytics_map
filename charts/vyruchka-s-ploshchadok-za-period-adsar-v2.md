# Chart: Выручка с площадок за период adsarV2

---
type: chart
name: Выручка с площадок за период adsarV2
superset_chart_id: 101
dataset: ad_stats_and_roi_v2
dashboards:
  - Детализация по рекламным платформам
purpose: comparison
---

## Назначение

Big number показывает суммарную выручку, атрибутированную рекламным площадкам в [ad_stats_and_roi_v2](../datasets/ad_stats_and_roi_v2.md).

## Superset

- Chart ID: `101`
- Visualization type: `big_number`
- Dataset: `adb.ad_stats_and_roi_v2` (id `30`)
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=101`

## Настройки

| Параметр | Значение |
|---|---|
| x axis | `stat_date` |
| time grain | `P1M` |
| metric | `SUM(revenue)` |
| temporal filter | `stat_date = Last quarter` |
| comparison | `compare_lag = 1` |

## Агрегация

```text
SUM(revenue)
```

