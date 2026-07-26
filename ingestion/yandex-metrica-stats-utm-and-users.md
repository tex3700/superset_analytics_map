# Ingestion: Yandex metrica stats UTM and Users

---
type: ingestion
name: yandex_metrica_stats_utm_and_users
tool: n8n
workflow_id: obuIyhzmPG4oo7S4
schedule: daily 04:00 Europe/Moscow
source: yandex_metrika
writes_to:
  - adb.ya_metrica_daily_stats
  - adb.ya_metrica_users_goals_detailed
---

## Назначение

n8n workflow загружает статистику Яндекс.Метрики в ADB:

- агрегированную дневную статистику по UTM и устройствам в `adb.ya_metrica_daily_stats`;
- детализацию пользователей, достигших целей, в `adb.ya_metrica_users_goals_detailed`.

## Техническая информация

- n8n workflow: `Yandex metrica stats UTM and Users`
- Workflow ID: `obuIyhzmPG4oo7S4`
- Активен: да
- Расписание: каждый день в 04:00, timezone `Europe/Moscow`
- OAuth token: n8n data table `api_tokens`, запись `service = yandex_id_oauth`
- Период загрузки в этом workflow: только вчерашний день (`1daysAgo` -> `yesterday`)

## Историческая загрузка

Первоначально workflow был запущен вручную в режиме загрузки исторических данных.

- Начальная дата исторической загрузки: `2025-09-01`
- Более ранние данные Яндекс.Метрика не вернула.
- После первичной загрузки workflow работает в ежедневном режиме и забирает только вчерашний день.

## Поток данных

```text
Schedule Trigger
  -> Get yandex token
  -> HTTP Request get metrica UTM data
  -> Prepare data filds for ya_metrica_daily_stats
  -> Merge
  -> Sort
  -> Execute a SQL query INSERT or UPDATE ya_metrica_daily_stats
```

Вторая ветка для дополнительных целей:

```text
Get yandex token
  -> Wait 15s1
  -> HTTP Request get metrica UTM data paprt2
  -> Prepare data filds for ya_metrica_daily_stats part2
  -> Merge
```

Отдельная ветка этого же workflow пишет пользовательскую детализацию:

```text
Get yandex token
  -> HTTP Request get metrica Users data
  -> DELETE ya_metrica_users_goals_detailed data
  -> Wait 15s
  -> Prepare data fields for ya_metrica_users_goals_detailed
  -> Sort1
  -> INSERT ya_metrica_users_goals_detailed
```

## API-запрос для `ya_metrica_daily_stats`

- Endpoint: `https://api-metrika.yandex.net/stat/v1/data`
- Counter ID: `19444381`
- `date1`: `1daysAgo`
- `date2`: `yesterday`
- `accuracy`: `full`
- `lang`: `ru`
- `limit`: `5000`

Измерения:

```text
ym:s:date
ym:s:lastSignUTMSource
ym:s:lastSignUTMMedium
ym:s:deviceCategory
```

Основные метрики:

```text
ym:s:visits
ym:s:users
ym:s:pageviews
ym:s:pageDepth
ym:s:avgVisitDurationSeconds
ym:s:bounceRate
ym:s:goal494050704reaches
ym:s:goal494050704users
ym:s:goal464707612reaches
ym:s:goal464707612users
ym:s:goal464710001reaches
ym:s:goal464710001users
ym:s:goal464710042reaches
ym:s:goal464710042users
ym:s:goal464077614reaches
ym:s:goal464077614users
ym:s:goal254435576reaches
ym:s:goal254435576users
```

Дополнительные метрики из второго API-запроса:

```text
ym:s:goal541193162reaches
ym:s:goal541193162users
ym:s:goal541193167reaches
ym:s:goal541193167users
```

## Запись в ADB

Таблица `ya_metrica_daily_stats` заполняется через:

```sql
INSERT INTO ya_metrica_daily_stats (...)
VALUES (...)
ON DUPLICATE KEY UPDATE ...
```

Это означает, что при повторной загрузке той же комбинации ключей значения метрик обновляются.

## Ключ объединения внутри n8n

Дополнительные метрики коллтрекинга объединяются с основным набором по полям:

```text
stat_date
utm_source
utm_medium
device_category
```

## Уникальный ключ целевой таблицы

Для `adb.ya_metrica_daily_stats` уникальная комбинация:

```text
stat_date + utm_source + utm_medium + device_category
```

Именно эта логика позволяет использовать `INSERT ... ON DUPLICATE KEY UPDATE` для ежедневного обновления вчерашних значений.

## Связанные датасеты Superset

- [ya_metrica_daily_stats](../datasets/ya_metrica_daily_stats.md)
- [ya_metrica_users_goals_detailed](../datasets/ya_metrica_users_goals_detailed.md)
