# Chart: Статистические данные по рекламным площадкам

---
type: chart
name: Статистические данные по рекламным площадкам
superset_chart_id: 49
dataset: ad_stats_and_roi
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Таблица показывает дневные строки расходов, выручки и ROI по рекламным площадкам.

## Superset

- Chart ID: `49`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.ad_stats_and_roi`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=49`

## Используемые поля

| Поле датасета | Роль |
|---|---|
| `stat_date` | дата |
| `platform` | рекламная площадка |
| `spend` | расход |
| `revenue` | выручка |
| `roi_percent` | ROI, % |

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last quarter` |

## Сортировка

```text
stat_date DESC
```

## Агрегация в чарте

Чарт не агрегирует данные. Используется raw table: отображаются строки, уже подготовленные virtual dataset.

## Особенности

- Включена server pagination.
- Server page length: `50`.
- Формат даты: `%Y-%m-%d`.

