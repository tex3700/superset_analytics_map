# Dashboard: Поведенческий из Яндекс.Метрики

---
type: dashboard
name: Поведенческий из Яндекс.Метрики
superset_dashboard_id: 12
charts:
  - metrika_po_istochnikam
  - metrika_po_polzovatelyam_dostigshim_tseli
  - tovary_v_zakaze
  - vyruchka_po_istochnikam
  - dolya_trafika_po_istochniku_pervogo_vizita
  - dolya_trafika_po_devaisam
datasets:
  - ya_metrica_daily_stats
  - ya_metrica_users_goals_detailed
  - order_products_ym
---

## Назначение

Дашборд собирает поведенческую аналитику из Яндекс.Метрики и связывает ее с детализацией целей и заказов.

## Superset

- Dashboard ID: `12`
- Dashboard name: `Поведенческий из Яндекс.Метрики`
- URL: `http://81.200.152.123:8088/superset/dashboard/12/`
- Owner: Дмитрий Пыланкин

## Чарты

| Chart ID | Чарт | Датасет | Назначение |
|---:|---|---|---|
| 42 | [Метрика по источникам](../charts/metrika-po-istochnikam.md) | `adb.ya_metrica_daily_stats` | Сырые дневные агрегаты по UTM и устройствам |
| 43 | [Метрика по пользователям достигшим цели](../charts/metrika-po-polzovatelyam-dostigshim-tseli.md) | `adb.ya_metrica_users_goals_detailed` | Пользовательская детализация достигнутых целей |
| 44 | [Товары в заказе](../charts/tovary-v-zakaze.md) | `adb.order_products_ym` | Товары в заказах |
| 45 | [Выручка по источникам](../charts/vyruchka-po-istochnikam.md) | `adb.ya_metrica_users_goals_detailed` | Выручка в разрезе источников |
| 56 | [Доля трафика по источнику первого визита](../charts/dolya-trafika-po-istochniku-pervogo-vizita.md) | `adb.ya_metrica_users_goals_detailed` | Распределение трафика по первому источнику |
| 57 | [Доля трафика по девайсам](../charts/dolya-trafika-po-devaisam.md) | `adb.ya_metrica_daily_stats` | Доля посетителей по устройствам |

## Основные датасеты

| Датасет | Роль |
|---|---|
| `ya_metrica_daily_stats` | Дневные агрегаты по UTM и устройствам |
| `ya_metrica_users_goals_detailed` | Детализация пользователей, достигших целей |
| `order_products_ym` | Заказы / товары для связки с конверсиями и выручкой |

## Как читать дашборд

Дашборд отвечает на вопросы:

- из каких UTM-источников приходит трафик;
- как трафик распределяется по устройствам;
- какие источники приводят пользователей к целевым действиям;
- как поведенческие данные связываются с заказами и выручкой.

## Известные ограничения

- Для `ya_metrica_daily_stats` пользователи в pie chart суммируются по дням и UTM-разрезам, поэтому это не обязательно уникальные пользователи за период.
- Часть чартов дашборда строится на других датасетах, которые нужно документировать отдельно.
