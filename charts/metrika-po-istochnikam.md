# Chart: Метрика по источникам

---
type: chart
name: Метрика по источникам
superset_chart_id: 42
dataset: ya_metrica_daily_stats
dashboards:
  - Поведенческий из Яндекс.Метрики
---

## Назначение

Табличный чарт показывает сырые строки датасета `ya_metrica_daily_stats` по датам, UTM-источникам, UTM-medium, устройствам и метрикам поведения / целей.

## Superset

- Chart ID: `42`
- Chart name: `Метрика по источникам`
- Visualization type: `table`
- Dataset: `adb.ya_metrica_daily_stats`
- Dashboard ID: `12`
- Dashboard: `Поведенческий из Яндекс.Метрики`
- URL: `http://81.200.152.123:8088/explore/?slice_id=42`

## Используемые поля

Чарт работает в `query_mode = raw` и выводит следующие поля:

```text
stat_date
utm_source
utm_medium
device_category
visits
users
pageviews
page_depth
avg_visit_duration_sec
bounce_rate
goal_order_paid_reaches
goal_order_paid_users
goal_add_to_cart_reaches
goal_add_to_cart_users
goal_ecommerce_reaches
goal_ecommerce_users
goal_quick_order_registration_reaches
goal_quick_order_registration_users
goal_zakaz_oformlen_reaches
goal_zakaz_oformlen_users
goal_callback_reaches
goal_callback_users
goal_novofon_call_reaches
goal_novofon_call_users
goal_zadarma_call_reaches
goal_zadarma_call_users
```

## Метрики

В чарте нет отдельных Superset-метрик: данные выводятся как raw table.

## Группировки

Нет.

## Фильтры

| Фильтр | Значение / логика |
|---|---|
| `stat_date` | `Last quarter` |

## Агрегация в чарте

Исходный grain датасета:

```text
1 день + 1 utm_source + 1 utm_medium + 1 device_category
```

Grain после агрегации в чарте:

```text
без изменения
```

Описание:

Чарт не агрегирует данные. Он показывает строки физической таблицы, отсортированные по `stat_date` по убыванию.

## Особенности интерпретации

- Это диагностическая таблица для просмотра исходного агрегата из Яндекс.Метрики.
- Лимит строк: `1000`.
- Серверная пагинация включена, размер страницы `50`.

