# Ingestion: Advertisings stats

---
type: ingestion
name: advertisings_stats
tool: n8n
workflow_id: 561dJKrRzSma3T8G
schedule: daily 03:00 Europe/Moscow
source: yandex_direct_via_yandex_metrika_api
writes_to:
  - adb.ad_direct_spend_and_goals_daily
  - adb.ad_spend_daily
---

## Назначение

n8n workflow загружает рекламную статистику Яндекс.Директа через API Яндекс.Метрики и пишет ее в две таблицы ADB:

- детальную дневную статистику по рекламным кампаниям в `adb.ad_direct_spend_and_goals_daily`;
- дневную агрегированную статистику по рекламной платформе в `adb.ad_spend_daily`.

## Техническая информация

- n8n workflow: `Advertisings stats`
- Workflow ID: `561dJKrRzSma3T8G`
- Активен: да
- Расписание: каждый день в 03:00, timezone `Europe/Moscow`
- Период загрузки: `1daysAgo` -> `yesterday`
- Режим записи: `INSERT ... ON DUPLICATE KEY UPDATE`
- Авторизационный токен есть в n8n HTTP node; секреты в документацию не переносятся.

## История данных

Исторической загрузки для этого workflow не было.

- Статистика собирается с момента первого запуска workflow: `2026-03-23`.
- Регулярно workflow загружает только вчерашний день.

## Поток данных

```text
Schedule Trigger
  -> HTTP Request
  -> Prepare data filds for direct_daily_stats
  -> Execute a SQL query ad_direct_spend_and_goals_daily INSERT or UPDATE
```

Параллельная ветка дневной агрегации:

```text
HTTP Request
  -> Prepare data filds for ad_daily_stats from direct
  -> Execute a SQL query direct ad_spend_daily INSERT or UPDATE
```

## API-запрос

- Endpoint: `https://api-metrika.yandex.net/stat/v1/data`
- Counter ID: `19444381`
- `date1`: `1daysAgo`
- `date2`: `yesterday`
- `direct_client_logins`: `e-17323892`
- `dimensions`: `ym:ad:directOrder,ym:ad:date`
- `accuracy`: `full`
- `lang`: `ru`
- `limit`: `5000`
- `sort`: `ym:ad:date`

Метрики:

```text
ym:ad:RUBConvertedAdCost
ym:ad:visits
ym:ad:users
ym:ad:clicks
ym:ad:goal494050704reaches
ym:ad:goal464707612reaches
ym:ad:goal464710001reaches
ym:ad:goal464710042reaches
ym:ad:goal464077614reaches
```

## Запись в `adb.ad_direct_spend_and_goals_daily`

Workflow преобразует каждую строку API в запись:

| Поле ADB | Источник / логика |
|---|---|
| `stat_date` | `ym:ad:date` |
| `direct_order_id` | `ym:ad:directOrder.direct_id` |
| `ad_cost` | `ym:ad:RUBConvertedAdCost`, округляется до 2 знаков |
| `visits` | `ym:ad:visits` |
| `users` | `ym:ad:users` |
| `clicks` | `ym:ad:clicks` |
| `goal_order_paid_reaches` | `ym:ad:goal494050704reaches` |
| `goal_add_to_cart_reaches` | `ym:ad:goal464707612reaches` |
| `goal_ecommerce_reaches` | `ym:ad:goal464710001reaches` |
| `goal_quick_order_registration_reaches` | `ym:ad:goal464710042reaches` |
| `goal_zakaz_oformlen_reaches` | `ym:ad:goal464077614reaches` |

## Запись в `adb.ad_spend_daily`

Workflow дополнительно агрегирует ответ API по дате:

| Поле ADB | Логика |
|---|---|
| `stat_date` | Дата из `ym:ad:date` |
| `advertising` | Константа `Yandex.Direct` |
| `campaign_count` | Количество уникальных `direct_order_id` за дату |
| `goals_reached` | Сумма пяти goal reach metrics за дату |
| `ad_cost` | Сумма `ym:ad:RUBConvertedAdCost` за дату, округляется до 2 знаков |

## Связанные датасеты Superset

- [yandex_direct_spend_and_stats_daily](../datasets/yandex_direct_spend_and_stats_daily.md)


