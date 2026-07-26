# Chart: Средняя стоимость целей Яндекс Директ

---
type: chart
name: Средняя стоимость целей Яндекс Директ
superset_chart_id: 48
dataset: yandex_direct_spend_and_stats_daily
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Mixed timeseries показывает среднюю стоимость действий по Яндекс.Директу:

- CPC: стоимость клика;
- CPV: стоимость посетителя;
- CPA: стоимость достигнутой цели.

## Superset

- Chart ID: `48`
- Visualization type: `mixed_timeseries`
- Dataset: `adb.yandex_direct_spend_and_stats_daily`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=48`
- X-axis: `stat_date`
- Time grain: `P1D`

## Используемые поля

| Поле датасета | Роль |
|---|---|
| `stat_date` | x-axis / дата |
| `ad_cost` | числитель расчетных метрик |
| `total_clicks` | знаменатель CPC |
| `total_users` | знаменатель CPV |
| `goals_reached` | знаменатель CPA |

## Метрики

| Метрика | Формула |
|---|---|
| `CPC (клик)` | `CASE WHEN MAX(total_clicks) = 0 OR MAX(total_clicks) IS NULL THEN NULL ELSE MAX(ad_cost) / MAX(total_clicks) END` |
| `CPV (посещение)` | `CASE WHEN MAX(total_users) = 0 OR MAX(total_users) IS NULL THEN NULL ELSE MAX(ad_cost) / MAX(total_users) END` |
| `CPA (цели)` | `CASE WHEN MAX(goals_reached) = 0 OR MAX(goals_reached) IS NULL THEN NULL ELSE MAX(ad_cost) / MAX(goals_reached) END` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last month` для основной группы метрик |

## Агрегация в чарте

Чарт рассчитывает показатели стоимости на уровне дня через SQL-метрики.

`MAX(...)` используется потому, что датасет уже должен отдавать одну строку на дату. Деление защищено от нуля и `NULL`: если знаменатель отсутствует или равен нулю, результат метрики будет `NULL`.

## Особенности

- `CPC` и `CPV` отображаются как line/area.
- `CPA` отображается как bar/area.
- Для `CPA` в Superset включена log axis.
- Для второй группы метрик в Superset указан temporal filter `No filter`; фактический общий временной диапазон нужно проверять при изменении чарта.

