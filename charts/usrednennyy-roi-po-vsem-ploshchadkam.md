# Chart: Усредненный ROI по всем площадкам

---
type: chart
name: Усредненный ROI по всем площадкам
superset_chart_id: 54
dataset: ad_stats_and_roi
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Big number показывает среднее значение `roi_percent` по строкам датасета за выбранный период.

## Superset

- Chart ID: `54`
- Visualization type: `big_number`
- Dataset: `adb.ad_stats_and_roi`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=54`
- X-axis: `stat_date`
- Time grain: `P1M`

## Метрика

| Метрика | Формула |
|---|---|
| `AVG(ROI)` | `AVG(roi_percent)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last quarter` |

## Агрегация в чарте

Чарт усредняет уже рассчитанные проценты ROI:

```text
AVG(roi_percent)
```

Это не то же самое, что ROI по суммарным расходам и выручке за период.

Текущая бизнес-логика подтверждена: для этого KPI нужен именно средний процент ROI, поэтому используется `AVG(roi_percent)`, а не расчет через суммы.

## Особенности

- Big number использует aggregation `LAST_VALUE` для временного ряда.
- Включены timestamp, trend line и сравнение с предыдущим периодом (`compare_lag = 1`).

