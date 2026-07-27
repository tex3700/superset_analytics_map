# Chart: Статистические данные по рекламным площадкам adsarV2

---
type: chart
name: Статистические данные по рекламным площадкам adsarV2
superset_chart_id: 103
dataset: ad_stats_and_roi_v2
dashboards:
  - Детализация по рекламным платформам
purpose: comparison
---

## Назначение

Raw table показывает дневные строки расходов, выручки и ROI по рекламным площадкам из [ad_stats_and_roi_v2](../datasets/ad_stats_and_roi_v2.md).

## Superset

- Chart ID: `103`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.ad_stats_and_roi_v2` (id `30`)
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=103`

## Используемые поля

Сейчас выводятся:

```text
stat_date
platform
spend
revenue
roi_percent
```

Для сверки рекомендуется добавить:

```text
attributed_orders
attribution_model
```

## Фильтры и сортировка

| Поле | Значение |
|---|---|
| `stat_date` | `Last quarter` |
| sort | `stat_date DESC` |
| row limit | `10000` |

## Агрегация

Нет. Chart выводит raw rows из virtual dataset.

