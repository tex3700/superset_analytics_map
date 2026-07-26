# Chart: Яндекс Директ общая статистика

---
type: chart
name: Яндекс Директ общая статистика
superset_chart_id: 46
dataset: yandex_direct_spend_and_stats_daily
dashboards:
  - Детализация по рекламным платформам
---

## Назначение

Таблица показывает дневные строки датасета `yandex_direct_spend_and_stats_daily` без дополнительной агрегации.

## Superset

- Chart ID: `46`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.yandex_direct_spend_and_stats_daily`
- Dashboard: `13`
- URL: `http://81.200.152.123:8088/explore/?slice_id=46`

## Используемые поля

| Поле датасета | Роль |
|---|---|
| `stat_date` | дата |
| `ad_cost` | расход на рекламу |
| `campaign_count` | количество кампаний |
| `goals_reached` | достигнуто целей |
| `total_clicks` | клики |
| `total_users` | посетители |
| `total_visits` | визиты |

## Фильтры

| Фильтр | Значение |
|---|---|
| `stat_date` | `Last quarter` |

## Агрегация в чарте

Чарт не агрегирует данные. Используется raw table: отображаются строки, уже подготовленные virtual dataset.

## Особенности

- Row limit: `1000`
- Server page length: `10`
- Формат даты: `%Y-%m-%d`

