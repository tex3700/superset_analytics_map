# ADB Table: ad_spend_daily

---
type: adb_table
name: ad_spend_daily
schema: adb
database: ADB
updated_by:
  - n8n:advertisings_stats
source_tables:
  - yandex_direct_via_yandex_metrika_api
used_by_datasets:
  - yandex_direct_spend_and_stats_daily
  - ad_stats_and_roi
---

## Назначение

Таблица хранит дневные агрегаты рекламных расходов и целей по рекламной платформе.

Для Яндекс.Директа строки пишет workflow [Advertisings stats](../ingestion/advertisings-stats.md) со значением:

```text
advertising = Yandex.Direct
```

## Источник и обновление

- Процесс для Яндекс.Директа: [Advertisings stats](../ingestion/advertisings-stats.md)
- Источник: [Yandex Direct via Yandex Metrika API](../sources/yandex-direct.md)
- Расписание: ежедневно в 03:00
- Период регулярной загрузки: вчерашний день
- Режим обновления: `INSERT ... ON DUPLICATE KEY UPDATE`

## Grain

Одна строка соответствует:

```text
1 день + 1 рекламная платформа
```

Технический уникальный ключ:

```text
stat_date + advertising
```

Ключ подтвержден.

## Поля

| Поле ADB | Источник / логика | Описание |
|---|---|---|
| `stat_date` | `ym:ad:date` | Дата статистики |
| `advertising` | Константа `Yandex.Direct` | Рекламная платформа |
| `campaign_count` | `COUNT(DISTINCT direct_order_id)` внутри n8n | Количество кампаний за дату |
| `goals_reached` | Сумма пяти goal reach metrics внутри n8n | Достигнуто целей |
| `ad_cost` | `SUM(ym:ad:RUBConvertedAdCost)` внутри n8n | Расход на рекламу |

## Используется в датасетах

| Датасет | Роль |
|---|---|
| [yandex_direct_spend_and_stats_daily](../datasets/yandex_direct_spend_and_stats_daily.md) | Источник дневных расходов, количества кампаний и целей |
| [ad_stats_and_roi](../datasets/ad_stats_and_roi.md) | Источник расходов по рекламным площадкам |
