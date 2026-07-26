# Chart: Статистика ROI Яндекс Директ

---
type: chart
name: Статистика ROI Яндекс Директ
superset_chart_id: 55
dataset: ad_stats_and_roi
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Mixed timeseries показывает расходы, выручку и ROI по площадке `Yandex.Direct` в динамике по дням.

## Superset

- Chart ID: `55`
- Visualization type: `mixed_timeseries`
- Dataset: `adb.ad_stats_and_roi`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=55`
- X-axis: `stat_date`
- Time grain: `P1D`

## Метрики

| Метрика | Формула | Ось / тип |
|---|---|---|
| `Расход на рекламу` | `MAX(spend)` | primary axis, bar |
| `Выручка от рекламы` | `MAX(revenue)` | primary axis, bar |
| `ROI` | `MAX(roi_percent)` | secondary axis, bar |

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last month` |
| `platform` | `Yandex.Direct` |

## Агрегация в чарте

Чарт использует `MAX(...)` по дневным значениям для `Yandex.Direct`.

Это корректно при ожидаемом grain датасета `1 строка = 1 день + 1 platform`: `MAX` не суммирует повторно уже дневные показатели.

## Особенности

- Расход и выручка отображаются на основной оси.
- ROI отображается на secondary axis с подписью `ROI %`.

