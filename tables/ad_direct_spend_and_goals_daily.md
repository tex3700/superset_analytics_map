# ADB Table: ad_direct_spend_and_goals_daily

---
type: adb_table
name: ad_direct_spend_and_goals_daily
schema: adb
database: ADB
updated_by:
  - n8n:advertisings_stats
source_tables:
  - yandex_direct_via_yandex_metrika_api
used_by_datasets:
  - yandex_direct_spend_and_stats_daily
---

## Назначение

Таблица хранит дневную рекламную статистику Яндекс.Директа в разрезе рекламной кампании.

## Источник и обновление

- Процесс: [Advertisings stats](../ingestion/advertisings-stats.md)
- Источник: [Yandex Direct via Yandex Metrika API](../sources/yandex-direct.md)
- Расписание: ежедневно в 03:00
- Период регулярной загрузки: вчерашний день
- Режим обновления: `INSERT ... ON DUPLICATE KEY UPDATE`

## Grain

Одна строка соответствует:

```text
1 день + 1 direct_order_id
```

Технический уникальный ключ:

```text
stat_date + direct_order_id
```

Ключ подтвержден.

## Поля

| Поле ADB | Источник / логика | Описание |
|---|---|---|
| `stat_date` | `ym:ad:date` | Дата статистики |
| `direct_order_id` | `ym:ad:directOrder.direct_id` | ID рекламной кампании / заказа Директа |
| `ad_cost` | `ym:ad:RUBConvertedAdCost` | Расход на рекламу |
| `visits` | `ym:ad:visits` | Визиты |
| `users` | `ym:ad:users` | Посетители |
| `clicks` | `ym:ad:clicks` | Клики |
| `goal_order_paid_reaches` | `ym:ad:goal494050704reaches` | Цель "Заказ оплачен" |
| `goal_add_to_cart_reaches` | `ym:ad:goal464707612reaches` | Цель "Кнопка купить" |
| `goal_ecommerce_reaches` | `ym:ad:goal464710001reaches` | Цель "Оформить заказ" |
| `goal_quick_order_registration_reaches` | `ym:ad:goal464710042reaches` | Цель оформления после быстрого заказа |
| `goal_zakaz_oformlen_reaches` | `ym:ad:goal464077614reaches` | Цель "Покупка" |

## Используется в датасетах

| Датасет | Роль |
|---|---|
| [yandex_direct_spend_and_stats_daily](../datasets/yandex_direct_spend_and_stats_daily.md) | Источник кликов, пользователей и визитов, агрегируемых по дате |
