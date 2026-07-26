# Dataset: ya_metrica_daily_stats

---
type: dataset
name: ya_metrica_daily_stats
superset_dataset_id: 14
superset_type: physical
database: ADB
database_id: 4
schema: adb
physical_table: ya_metrica_daily_stats
owner: Дмитрий Пыланкин
depends_on:
  - yandex_metrika
  - n8n:yandex_metrica_stats_utm_and_users
used_by_charts:
  - metrika_po_istochnikam
  - dolya_trafika_po_devaisam
---

## Краткое описание

Физический датасет Superset на таблице `adb.ya_metrica_daily_stats`.

Содержит дневную поведенческую статистику Яндекс.Метрики в разрезе UTM-источника, UTM-medium и категории устройства.

## Superset

- Dataset ID: `14`
- Dataset name: `ya_metrica_daily_stats`
- Dataset type: physical
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- Physical table: `ya_metrica_daily_stats`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/14`
- Owner: Дмитрий Пыланкин
- Main datetime column: `stat_date`

## Источники данных

| Источник | Процесс загрузки | Таблицы ADB |
|---|---|---|
| Яндекс.Метрика API | `Yandex metrica stats UTM and Users` | `adb.ya_metrica_daily_stats` |

## SQL датасета

Датасет physical, собственного SQL в Superset нет.

Источник данных:

```text
ADB.adb.ya_metrica_daily_stats
```

## Grain

Одна строка соответствует:

```text
1 день + 1 lastSignUTMSource + 1 lastSignUTMMedium + 1 deviceCategory
```

Предполагаемый технический ключ:

```text
stat_date + utm_source + utm_medium + device_category
```

Этот ключ подтвержден как уникальный ключ таблицы `ya_metrica_daily_stats`.

## Поля

| Поле | Тип | Описание | Источник / формула |
|---|---|---|---|
| `id` | INTEGER | Технический ID строки | ADB |
| `stat_date` | DATE | Дата статистики | `ym:s:date` |
| `utm_source` | VARCHAR | Источник UTM | `ym:s:lastSignUTMSource` |
| `utm_medium` | VARCHAR | UTM medium | `ym:s:lastSignUTMMedium` |
| `device_category` | VARCHAR | Категория устройства | `ym:s:deviceCategory` |
| `visits` | INTEGER | Визиты | `ym:s:visits` |
| `users` | INTEGER | Посетители | `ym:s:users` |
| `pageviews` | INTEGER | Просмотры страниц | `ym:s:pageviews` |
| `page_depth` | FLOAT | Глубина просмотра | `ym:s:pageDepth` |
| `avg_visit_duration_sec` | FLOAT | Средняя длительность визита, сек | `ym:s:avgVisitDurationSeconds` |
| `bounce_rate` | FLOAT | Доля отказов | `ym:s:bounceRate` |
| `goal_order_paid_reaches` | INTEGER | Достижения цели "Заказ оплачен" | goal API metric |
| `goal_order_paid_users` | INTEGER | Пользователи, достигшие цели "Заказ оплачен" | `ym:s:goal494050704users` |
| `goal_add_to_cart_reaches` | INTEGER | Достижения цели "Кнопка купить" | `ym:s:goal464707612reaches` |
| `goal_add_to_cart_users` | INTEGER | Пользователи, достигшие цели "Кнопка купить" | `ym:s:goal464707612users` |
| `goal_ecommerce_reaches` | INTEGER | Достижения цели "Оформить заказ" | `ym:s:goal464710001reaches` |
| `goal_ecommerce_users` | INTEGER | Пользователи, достигшие цели "Оформить заказ" | `ym:s:goal464710001users` |
| `goal_quick_order_registration_reaches` | INTEGER | Достижения цели оформления после быстрого заказа | `ym:s:goal464710042reaches` |
| `goal_quick_order_registration_users` | INTEGER | Пользователи, достигшие цели оформления после быстрого заказа | `ym:s:goal464710042users` |
| `goal_zakaz_oformlen_reaches` | INTEGER | Достижения цели "Покупка" | `ym:s:goal464077614reaches` |
| `goal_zakaz_oformlen_users` | INTEGER | Пользователи, достигшие цели "Покупка" | `ym:s:goal464077614users` |
| `goal_callback_reaches` | INTEGER | Достижения цели "Обратный звонок" | `ym:s:goal254435576reaches` |
| `goal_callback_users` | INTEGER | Пользователи, достигшие цели "Обратный звонок" | `ym:s:goal254435576users` |
| `goal_novofon_call_reaches` | INTEGER | Достижения цели "Звонок Новофон" | `ym:s:goal541193162reaches` |
| `goal_novofon_call_users` | INTEGER | Пользователи, достигшие цели "Звонок Новофон" | `ym:s:goal541193162users` |
| `goal_zadarma_call_reaches` | INTEGER | Достижения цели "Звонок Zadarma" | `ym:s:goal541193167reaches` |
| `goal_zadarma_call_users` | INTEGER | Пользователи, достигшие цели "Звонок Zadarma" | `ym:s:goal541193167users` |
| `created_at` | TIMESTAMP | Время создания строки | ADB |
| `updated_at` | TIMESTAMP | Время обновления строки | ADB |

## Метрики в датасете

В Superset определена только стандартная метрика:

| Метрика | Формула | Описание |
|---|---|---|
| `count` | `COUNT(*)` | Количество строк |

Большинство чартов используют поля напрямую или задают агрегации на уровне чарта.

## Агрегации внутри датасета

В Superset агрегации внутри датасета нет, так как это physical dataset.

Агрегация происходит до попадания данных в таблицу: Яндекс.Метрика API возвращает агрегированные значения по выбранным измерениям:

```text
date + lastSignUTMSource + lastSignUTMMedium + deviceCategory
```

## Обновление данных

- Частота обновления: ежедневно
- Время запуска: 04:00 Europe/Moscow
- Процесс: n8n workflow `Yandex metrica stats UTM and Users`
- Период загрузки: `1daysAgo` -> `yesterday`
- Режим записи: `INSERT ... ON DUPLICATE KEY UPDATE`
- В этом workflow исторический backfill не выполняется, загружается только вчерашний день.
- Пустые UTM-значения сохраняются как пришли из Яндекс.Метрики.

## Историческая загрузка

Первичная историческая загрузка была выполнена вручную через workflow `Yandex metrica stats UTM and Users`.

- История загружалась начиная с `2025-09-01`.
- Более ранние данные Яндекс.Метрика не вернула.
- После первичной загрузки workflow переведен в регулярный режим: ежедневно загружается только вчерашний день.

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Метрика по источникам](../charts/metrika-po-istochnikam.md) | [Поведенческий из Яндекс.Метрики](../dashboards/povedencheskiy-iz-yandex-metriki.md) | Raw table, без агрегации в чарте |
| [Доля трафика по девайсам](../charts/dolya-trafika-po-devaisam.md) | [Поведенческий из Яндекс.Метрики](../dashboards/povedencheskiy-iz-yandex-metriki.md) | `SUM(users)` по `device_category` |

## Проверки качества

- `stat_date` не должен быть пустым.
- `utm_source`, `utm_medium`, `device_category` должны соответствовать значениям из Яндекс.Метрики.
- Числовые метрики не должны быть отрицательными.
- Для одной комбинации `stat_date + utm_source + utm_medium + device_category` должна быть одна актуальная строка.

## Вопросы

- Если появится отдельный процесс повторной загрузки истории Яндекс.Метрики, добавить его в `ingestion/`.
