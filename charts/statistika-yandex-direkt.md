# Chart: Статистика Яндекс Директ

---
type: chart
name: Статистика Яндекс Директ
superset_chart_id: 47
dataset: yandex_direct_spend_and_stats_daily
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Mixed timeseries показывает динамику рекламных показателей Яндекс.Директа по дням.

## Superset

- Chart ID: `47`
- Visualization type: `mixed_timeseries`
- Dataset: `adb.yandex_direct_spend_and_stats_daily`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=47`
- X-axis: `stat_date`
- Time grain: `P1D`

## Используемые поля

| Поле датасета | Роль |
|---|---|
| `stat_date` | x-axis / дата |
| `total_clicks` | metric |
| `total_users` | metric |
| `total_visits` | metric |
| `goals_reached` | metric |
| `ad_cost` | metric B |

## Метрики

| Метрика | Формула | Ось / тип |
|---|---|---|
| `Клики` | `MAX(total_clicks)` | secondary axis, line/area |
| `Посетители` | `MAX(total_users)` | secondary axis, line/area |
| `Визиты` | `MAX(total_visits)` | secondary axis, line/area |
| `Достигнуто целей` | `MAX(goals_reached)` | secondary axis, line/area |
| `Расход на рекламу` | `MAX(ad_cost)` | primary axis, bar |

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last month` для основной группы метрик |

## Агрегация в чарте

Чарт использует `MAX(...)` по дневным значениям.

Это корректно при ожидаемом grain датасета `1 строка = 1 день`: `MAX` не суммирует повторно уже дневные показатели.

## Особенности

- Основные метрики отображаются линиями/area.
- Расход на рекламу отображается столбцами.
- Для второй группы метрик в Superset указан temporal filter `No filter`; фактический общий временной диапазон нужно проверять при изменении чарта.

