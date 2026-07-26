# Chart: Выручка по источникам

---
type: chart
name: Выручка по источникам
superset_chart_id: 45
dataset: ya_metrica_users_goals_detailed
dashboards:
  - Поведенческий из Яндекс.Метрики
  - Детализация по рекламным платформам
---

## Назначение

Pivot table показывает выручку по дням и источникам последнего значимого перехода.

## Superset

- Chart ID: `45`
- Visualization type: `pivot_table_v2`
- Dataset: `adb.ya_metrica_users_goals_detailed`
- Dashboards: `12`, `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=45`

## Используемые поля

| Поле датасета | Роль |
|---|---|
| `stat_date` | row group / дата |
| `last_sign_utm_source` | column group через SQL CASE |
| `total_value` | metric |

## Метрики

| Метрика | Формула |
|---|---|
| `SUM(Итого после вычетов)` | `SUM(total_value)` |

## Нормализация источников

В чарте используется SQL expression:

```sql
CASE
    WHEN LOWER(last_sign_utm_source) IN ('ya', 'yandex') THEN 'Yandex.Direct'
    WHEN LOWER(last_sign_utm_source) IN ('admitad') THEN 'Admitad'
    WHEN LOWER(last_sign_utm_source) IN ('email') THEN 'Email'
    WHEN LOWER(last_sign_utm_source) IN ('instagram') THEN 'Instagram'
    WHEN LOWER(last_sign_utm_source) IN ('telegram') THEN 'Telegram'
    WHEN LOWER(last_sign_utm_source) IN ('null') THEN 'Органика'
    ELSE last_sign_utm_source
END
```

## Группировки

```text
Rows: stat_date
Columns: normalized last_sign_utm_source
```

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last quarter` |

## Агрегация в чарте

Чарт суммирует `total_value` по дате и нормализованному источнику последнего значимого перехода.

## Особенности

- `total_value` в датасете заполняется только для заказов со статусом не из списка отмен/возвратов.
- Включены итоги по строкам и колонкам.

