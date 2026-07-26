# Chart: Доля трафика по источнику первого визита

---
type: chart
name: Доля трафика по источнику первого визита
superset_chart_id: 56
dataset: ya_metrica_users_goals_detailed
dashboards:
  - Поведенческий из Яндекс.Метрики
---

## Назначение

Funnel chart показывает распределение строк датасета по источнику первого визита.

## Superset

- Chart ID: `56`
- Visualization type: `funnel`
- Dataset: `adb.ya_metrica_users_goals_detailed`
- Dashboard ID: `12`
- URL: `http://81.200.152.123:8088/explore/?slice_id=56`

## Используемые поля

| Поле датасета | Роль |
|---|---|
| `first_traffic_source` | group by |
| `stat_date` | temporal filter |

## Метрики

| Метрика | Формула |
|---|---|
| `Количество визитов_` | `COUNT(first_traffic_source)` |

## Группировки

```text
first_traffic_source
```

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last month` |

## Агрегация в чарте

Чарт считает количество строк с непустым `first_traffic_source` по каждому источнику первого визита.

## Особенности

- `row_limit = 10`.
- Сортировка по метрике включена.
- Проценты считаются от total.

