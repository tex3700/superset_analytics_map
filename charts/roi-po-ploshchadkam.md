# Chart: ROI по площадкам

---
type: chart
name: ROI по площадкам
superset_chart_id: 50
dataset: ad_stats_and_roi
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Mixed chart сравнивает рекламные расходы, выручку и ROI по площадкам за выбранный период.

## Superset

- Chart ID: `50`
- Visualization type: `mixed_timeseries`
- Dataset: `adb.ad_stats_and_roi`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=50`
- X-axis: `platform`

## Метрики

| Метрика | Формула | Ось / тип |
|---|---|---|
| `Рекламные расходы` | `SUM(spend)` | secondary axis, bar |
| `Выручка от рекламы` | `SUM(revenue)` | secondary axis, bar |
| `ROI` | `CASE WHEN SUM(spend) = 0 THEN NULL ELSE (SUM(revenue) - SUM(spend)) / SUM(spend) * 100 END` | primary axis, bar |

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last month` для основной группы метрик |

## Агрегация в чарте

Чарт агрегирует строки датасета по площадке за период.

ROI считается через суммы расходов и выручки за период, а не через среднее дневных ROI:

```text
(SUM(revenue) - SUM(spend)) / SUM(spend) * 100
```

## Особенности

- Для второй группы метрик в Superset указан temporal filter `No filter`; фактический общий временной диапазон нужно проверять при изменении чарта.
- Для основной оси включена log axis.

