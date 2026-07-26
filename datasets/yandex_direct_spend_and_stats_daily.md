# Dataset: yandex_direct_spend_and_stats_daily

---
type: dataset
name: yandex_direct_spend_and_stats_daily
superset_dataset_id: 17
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.ad_spend_daily
  - adb.ad_direct_spend_and_goals_daily
used_by_charts:
  - yandex_direkt_obshchaya_statistika
  - statistika_yandex_direkt
  - srednyaya_stoimost_tseley_yandex_direkt
---

## Краткое описание

Virtual dataset объединяет дневные расходы Яндекс.Директа из `adb.ad_spend_daily` с дневной статистикой кликов, пользователей и визитов из `adb.ad_direct_spend_and_goals_daily`.

## Superset

- Dataset ID: `17`
- Dataset name: `yandex_direct_spend_and_stats_daily`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/17`
- Owner: Дмитрий Пыланкин
- Main datetime column: `stat_date`

## Источники данных

| Источник | Процесс загрузки | Таблицы ADB |
|---|---|---|
| Yandex Direct через Yandex Metrika API | `Advertisings stats` | `adb.ad_direct_spend_and_goals_daily`, `adb.ad_spend_daily` |

## SQL датасета

```sql
WITH aggregated_params AS (
    SELECT
        stat_date,
        SUM(clicks) AS total_clicks,
        SUM(users) AS total_users,
        SUM(visits) AS total_visits
    FROM adb.ad_direct_spend_and_goals_daily
    GROUP BY stat_date
)
SELECT
    s.stat_date,
    s.ad_cost,
    s.campaign_count,
    s.goals_reached,
    p.total_clicks,
    p.total_users,
    p.total_visits
FROM adb.ad_spend_daily AS s
LEFT JOIN aggregated_params AS p
    ON s.stat_date = p.stat_date
WHERE s.advertising = 'Yandex.Direct'
ORDER BY s.stat_date DESC;
```

## Grain

Одна строка датасета соответствует:

```text
1 день по рекламной платформе Yandex.Direct
```

Так как в `WHERE` оставлена только платформа `Yandex.Direct`, ожидаемая уникальность результата:

```text
stat_date
```

## Поля

| Поле | Тип | Описание | Источник / формула |
|---|---|---|---|
| `stat_date` | DATE | Дата статистики | `adb.ad_spend_daily.stat_date` |
| `ad_cost` | FLOAT | Расход на рекламу | `adb.ad_spend_daily.ad_cost` |
| `campaign_count` | LONG | Запущено рекламных кампаний | `adb.ad_spend_daily.campaign_count` |
| `goals_reached` | LONG | Достигнуто целей | `adb.ad_spend_daily.goals_reached` |
| `total_clicks` | DECIMAL | Клики | `SUM(ad_direct_spend_and_goals_daily.clicks)` по дате |
| `total_users` | DECIMAL | Посетители | `SUM(ad_direct_spend_and_goals_daily.users)` по дате |
| `total_visits` | DECIMAL | Визиты | `SUM(ad_direct_spend_and_goals_daily.visits)` по дате |

## Метрики в датасете

В Superset определена только стандартная метрика:

| Метрика | Формула | Описание |
|---|---|---|
| `count` | `COUNT(*)` | Количество строк |

## Агрегации внутри датасета

Датасет агрегирует таблицу `adb.ad_direct_spend_and_goals_daily` до уровня дня:

```text
SUM(clicks), SUM(users), SUM(visits)
GROUP BY stat_date
```

Расход, количество кампаний и цели берутся из уже агрегированной таблицы `adb.ad_spend_daily`.

## Обновление данных

- Частота обновления: ежедневно
- Время запуска: 03:00 Europe/Moscow
- Процесс: n8n workflow `Advertisings stats`
- Период загрузки: `1daysAgo` -> `yesterday`
- Режим записи upstream-таблиц: `INSERT ... ON DUPLICATE KEY UPDATE`
- Исторической загрузки не было.
- Данные собираются с момента первого запуска workflow: `2026-03-23`.

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Яндекс Директ общая статистика](../charts/yandex-direkt-obshchaya-statistika.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | Raw table, без агрегации в чарте |
| [Статистика Яндекс Директ](../charts/statistika-yandex-direkt.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | `MAX(...)` дневных значений по `stat_date` |
| [Средняя стоимость целей Яндекс Директ](../charts/srednyaya-stoimost-tseley-yandex-direkt.md) | [Детализация по рекламным платформам](../dashboards/detalizatsiya-po-reklamnym-platformam.md) | SQL-метрики `ad_cost / clicks`, `ad_cost / users`, `ad_cost / goals_reached` |

## Проверки качества

- В датасете должна быть не более одной строки на `stat_date`.
- `ad_cost`, `campaign_count`, `goals_reached`, `total_clicks`, `total_users`, `total_visits` не должны быть отрицательными.
- Для дат с расходом желательно проверять наличие кликов/визитов, иначе CPA/CPC/CPV могут быть пустыми или аномальными.

