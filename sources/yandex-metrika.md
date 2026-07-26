# Source: Yandex Metrika

---
type: source
name: yandex_metrika
system_type: api
owner: Дмитрий Пыланкин
related_ingestion:
  - yandex_metrica_stats_utm_and_users
related_adb_tables:
  - adb.ya_metrica_daily_stats
  - adb.ya_metrica_users_goals_detailed
---

## Назначение

Источник содержит поведенческую статистику сайта из Яндекс.Метрики: визиты, посетители, просмотры, глубину просмотра, длительность визита, отказы и достижения целей.

Данные используются в Superset для анализа трафика по UTM-источникам, UTM-medium, устройствам и целевым действиям.

## Тип источника

- Система: Яндекс.Метрика
- Тип: API
- API endpoint: `https://api-metrika.yandex.net/stat/v1/data`
- Counter ID: `19444381`
- Авторизация: OAuth token из n8n data table `api_tokens`, запись `service = yandex_id_oauth`

## Какие данные забираем

| Сущность / отчет | Описание | Периодичность |
|---|---|---|
| UTM daily stats | Дневная статистика по UTM-источнику, UTM-medium и устройству | ежедневно |
| Users goals details | Детализация пользователей, достигших целей | ежедневно |

## Важные измерения

| Измерение API | Поле в ADB | Описание |
|---|---|---|
| `ym:s:date` | `stat_date` | Дата статистики |
| `ym:s:lastSignUTMSource` | `utm_source` | Последний значимый UTM source |
| `ym:s:lastSignUTMMedium` | `utm_medium` | Последний значимый UTM medium |
| `ym:s:deviceCategory` | `device_category` | Категория устройства |

## Важные метрики

| Метрика API | Поле в ADB | Описание |
|---|---|---|
| `ym:s:visits` | `visits` | Визиты |
| `ym:s:users` | `users` | Посетители |
| `ym:s:pageviews` | `pageviews` | Просмотры страниц |
| `ym:s:pageDepth` | `page_depth` | Глубина просмотра |
| `ym:s:avgVisitDurationSeconds` | `avg_visit_duration_sec` | Средняя длительность визита в секундах |
| `ym:s:bounceRate` | `bounce_rate` | Доля отказов |
| `ym:s:goal...reaches` | `goal_*_reaches` | Достижения целей |
| `ym:s:goal...users` | `goal_*_users` | Пользователи, достигшие целей |

## Цели Яндекс.Метрики

| Goal ID | Кодовое имя | Бизнес-название |
|---:|---|---|
| `494050704` | `order_paid` | Заказ оплачен |
| `464707612` | `add_to_cart` | Кнопка купить |
| `464710001` | `ecommerce` | Оформить заказ |
| `464710042` | `quick_order_registration` | Оформление после быстрого заказа |
| `464077614` | `zakaz_oformlen` | Покупка |
| `254435576` | `callback` | Обратный звонок |
| `541193162` | `novofon_call` | Звонок Новофон |
| `541193167` | `zadarma_call` | Звонок Zadarma |

## Ограничения и особенности

- Workflow забирает данные за период `1daysAgo` -> `yesterday`.
- В API-запросах используется `accuracy = full`.
- Для таблицы `ya_metrica_daily_stats` данные пишутся через `INSERT ... ON DUPLICATE KEY UPDATE`.
- Для части целей данные забираются вторым API-запросом и объединяются в n8n по ключу `stat_date, utm_source, utm_medium, device_category`.
- Пустые UTM-значения сохраняются в ADB как пришли из Яндекс.Метрики, без дополнительной нормализации.
- Первичная историческая загрузка была выполнена вручную тем же workflow с даты `2025-09-01`. Более ранние данные Яндекс.Метрика не вернула.

## Вопросы

- Сейчас отдельный регулярный backfill-процесс не описан. Если появится отдельная процедура перезагрузки истории, ее нужно документировать отдельно.
