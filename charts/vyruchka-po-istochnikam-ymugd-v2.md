# Chart: Выручка по источникам ymugdV2

---
type: chart
name: Выручка по источникам ymugdV2
superset_chart_id: 107
dataset: ya_metrica_users_goals_detailed_v2
dashboards:
  - Поведенческий из Яндекс.Метрики
  - Детализация по рекламным платформам
purpose: comparison
---

## Назначение

Pivot table для сравнения выручки по источникам после дедупликации заказов в [ya_metrica_users_goals_detailed_v2](../datasets/ya_metrica_users_goals_detailed_v2.md).

## Superset

- Chart ID: `107`
- Visualization type: `pivot_table_v2`
- Dataset: `adb.ya_metrica_users_goals_detailed_v2` (id `29`)
- Dashboards: `12`, `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=107`

## Настройки

| Параметр | Значение |
|---|---|
| rows | `stat_date` |
| columns | SQL CASE по `last_sign_utm_source` |
| metric | `SUM(total_value)` |
| temporal filter | `stat_date = Last quarter` |
| row limit | `10000` |
| totals | row totals и column totals включены |

## Нормализация источников

Chart нормализует `last_sign_utm_source`: `ya` / `yandex` -> `Yandex.Direct`, `admitad` -> `Admitad`, `email` -> `Email`, `instagram` -> `Instagram`, `telegram` -> `Telegram`, `null` -> `Органика`, остальные значения оставляет как есть.

## Агрегация

```text
SUM(total_value) по stat_date и нормализованному last_sign_utm_source
```

