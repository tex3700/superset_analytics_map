# Data Lineage Map

Этот файл содержит общую карту движения данных: от первичных источников до дашбордов Superset.

Заполняем его постепенно по мере описания реальных источников, загрузок, датасетов и чартов.

## Общий шаблон

```text
Источник данных
  -> процесс загрузки
  -> таблица / view в ADB
  -> Superset dataset
  -> Superset chart
  -> Superset dashboard
```

## Карта зависимостей

```text
Yandex Metrika API
  -> n8n workflow: Yandex metrica stats UTM and Users
  -> ADB.adb.ya_metrica_daily_stats
  -> Superset dataset: ya_metrica_daily_stats (id=14)
  -> Chart: Метрика по источникам (id=42)
  -> Dashboard: Поведенческий из Яндекс.Метрики (id=12)

Yandex Metrika API
  -> n8n workflow: Yandex metrica stats UTM and Users
  -> ADB.adb.ya_metrica_daily_stats
  -> Superset dataset: ya_metrica_daily_stats (id=14)
  -> Chart: Доля трафика по девайсам (id=57)
  -> Dashboard: Поведенческий из Яндекс.Метрики (id=12)

Yandex Metrika API
  -> n8n workflow: Yandex metrica stats UTM and Users
  -> ADB.adb.ya_metrica_users_goals_detailed
  -> Superset virtual dataset: ya_metrica_users_goals_detailed (id=15)
  -> Chart: Метрика по пользователям достигшим цели (id=43)
  -> Dashboard: Поведенческий из Яндекс.Метрики (id=12)

MID / CRM databases
  -> PHP cron: adb.php, daily 06:30
  -> ADB.adb.orders + ADB.adb.order_products + ADB.adb.orders_totals
  -> Superset virtual dataset: ya_metrica_users_goals_detailed (id=15)
  -> Chart: Выручка по источникам (id=45)
  -> Dashboards: Поведенческий из Яндекс.Метрики (id=12), Детализация по рекламным платформам (id=13)

MID database
  -> PHP cron: adb.php, daily 06:30
  -> ADB.adb.order_products
  -> Superset virtual dataset: order_products_ym (id=16)
  -> Chart: Товары в заказе (id=44)
  -> Dashboards: Поведенческий из Яндекс.Метрики (id=12), Детализация по продажам и воронке лидов (id=14)

Yandex Metrika API
  -> n8n workflow: Yandex metrica stats UTM and Users
  -> ADB.adb.ya_metrica_users_goals_detailed
  -> Superset virtual dataset: ya_metrica_users_goals_detailed (id=15)
  -> Chart: Доля трафика по источнику первого визита (id=56)
  -> Dashboard: Поведенческий из Яндекс.Метрики (id=12)

Yandex Direct via Yandex Metrika API
  -> n8n workflow: Advertisings stats
  -> ADB.adb.ad_direct_spend_and_goals_daily
  -> Superset virtual dataset: yandex_direct_spend_and_stats_daily (id=17)
  -> Chart: Яндекс Директ общая статистика (id=46)
  -> Dashboard: Детализация по рекламным платформам (id=13)

Yandex Direct via Yandex Metrika API
  -> n8n workflow: Advertisings stats
  -> ADB.adb.ad_direct_spend_and_goals_daily + ADB.adb.ad_spend_daily
  -> Superset virtual dataset: yandex_direct_spend_and_stats_daily (id=17)
  -> Chart: Статистика Яндекс Директ (id=47)
  -> Dashboard: Детализация по рекламным платформам (id=13)

Yandex Direct via Yandex Metrika API
  -> n8n workflow: Advertisings stats
  -> ADB.adb.ad_direct_spend_and_goals_daily + ADB.adb.ad_spend_daily
  -> Superset virtual dataset: yandex_direct_spend_and_stats_daily (id=17)
  -> Chart: Средняя стоимость целей Яндекс Директ (id=48)
  -> Dashboard: Детализация по рекламным платформам (id=13)

Yandex Direct via Yandex Metrika API + Yandex Metrika users/goals + MID / CRM orders
  -> n8n workflow: Advertisings stats + Yandex metrica stats UTM and Users + PHP cron adb.php
  -> ADB.adb.ad_spend_daily + ADB.adb.ya_metrica_users_goals_detailed + ADB.adb.orders + ADB.adb.orders_totals
  -> Superset virtual dataset: ad_stats_and_roi (id=18)
  -> Charts: Статистические данные по рекламным площадкам (id=49), ROI по площадкам (id=50), Выручка по площадкам (id=51), Рекламные расходы за период (id=52), Выручка с площадок за период (id=53), Усредненный ROI по всем площадкам (id=54), Статистика ROI Яндекс Директ (id=55)
  -> Dashboard: Детализация по рекламным платформам (id=13)

Yandex Metrika API + MID / CRM orders
  -> n8n workflow: Yandex metrica stats UTM and Users + PHP cron adb.php
  -> ADB.adb.ya_metrica_users_goals_detailed + ADB.adb.orders + ADB.adb.order_products + ADB.adb.orders_totals
  -> Superset virtual dataset: ya_metrica_users_goals_detailed_v2 (id=29)
  -> Charts: Метрика по пользователям достигшим цели ymugdV2 (id=105), Доля трафика по источнику первого визита ymugdV2 (id=106), Выручка по источникам ymugdV2 (id=107)
  -> Purpose: comparison / corrected order-level attribution

Yandex Direct via Yandex Metrika API + Yandex Metrika users/goals + MID / CRM orders
  -> n8n workflow: Advertisings stats + Yandex metrica stats UTM and Users + PHP cron adb.php
  -> ADB.adb.ad_spend_daily + ADB.adb.ya_metrica_users_goals_detailed + ADB.adb.orders + ADB.adb.orders_totals
  -> Superset virtual dataset: ad_stats_and_roi_v2 (id=30)
  -> Charts: Статистика ROI Яндекс Директ adsarV2 (id=98), ROI по площадкам adsarV2 (id=99), Усредненный ROI по всем площадкам adsarV2 (id=100), Выручка с площадок за период adsarV2 (id=101), Рекламные расходы за период adsarV2 (id=102), Статистические данные по рекламным площадкам adsarV2 (id=103), Выручка по площадкам adsarV2 (id=104)
  -> Purpose: comparison / ROI revenue overcount fix

MID / CRM orders + Yandex Metrika users/goals
  -> PHP cron adb.php + n8n workflow: Yandex metrica stats UTM and Users
  -> ADB.adb.orders + ADB.adb.orders_totals + ADB.adb.ya_metrica_users_goals_detailed
  -> Superset virtual dataset: ad_order_attribution_v2 (id=31)
  -> Chart: Не агрегированные данные по заказам с рекламных площадок (id=108)
  -> Dashboard: Детализация по рекламным платформам (id=13)

ADB quality checks
  -> n8n workflow: Analytics data quality
  -> MySQL procedure: adb.refresh_analytics_data_quality_daily
  -> ADB.adb.analytics_data_quality_daily
  -> Superset physical dataset: analytics_data_quality_daily (id=32)
  -> Charts: 109, 110, 111, 112, 113, 114, 115, 116, 117, 118
  -> Dashboard: Качество данных аналитики (id=16)

MID / CRM databases + Yandex Metrika users/goals
  -> PHP cron adb.php + n8n workflow: Yandex metrica stats UTM and Users
  -> ADB.adb.orders + ADB.adb.customers + ADB.adb.contact_events + ADB.adb.orders_totals + ADB.adb.ya_metrica_users_goals_detailed
  -> Superset virtual dataset: mart_lid_orders (id=19)
  -> Charts: Аналитика лидов по заказам (id=58), Общая выручка за период (id=60), Общая прибыль за период (id=61), Динамика выручка и прибыль по дням (id=62), Новые и лояльные покупатели за период (id=63), Количество заказов за период (id=64), Среднее время от регистрации лида до первого заказа (id=65), Средний чек AOV (id=70), Распределение менеджеров по заказам (id=74)
  -> Dashboard: Детализация по продажам и воронке лидов (id=14)

CRM database
  -> PHP cron adb.php, function update_mart_lids
  -> ADB.adb.mart_lids
  -> Superset physical dataset: mart_lids (id=20)
  -> Charts: Детализация по лидам (id=59), Источники лидов по CRM за период (id=69), Распределение менеджеров по лидам (id=75)
  -> Dashboard: Детализация по продажам и воронке лидов (id=14)

CRM leads + MID orders
  -> PHP cron adb.php
  -> ADB.adb.mart_lids + ADB.adb.orders
  -> Superset virtual dataset: con_lids_orders (id=21)
  -> Chart: Отношение Заказы - Лиды (id=72)
  -> Dashboard: Детализация по продажам и воронке лидов (id=14)

CRM leads + customers + MID orders
  -> PHP cron adb.php
  -> ADB.adb.mart_lids + ADB.adb.customers + ADB.adb.orders
  -> Superset virtual dataset: rel_lid_orders (id=22)
  -> Chart: Конверсия (Лид -> Заказ) (id=73)
  -> Dashboard: Детализация по продажам и воронке лидов (id=14)

CRM abandoned order leads + MID abandoned orders + Yandex Metrika users/goals
  -> PHP cron adb.php + n8n workflow: Yandex metrica stats UTM and Users
  -> ADB.adb.mart_lids + ADB.adb.ym_abandoned_orders + ADB.adb.orders + ADB.adb.ya_metrica_users_goals_detailed
  -> Superset virtual dataset: source_abadoned_orders (id=23)
  -> Charts: Брошенные заказы с привязкой к источнику (id=76), Источники по брошенным заказам (id=77)
  -> Dashboard: Детализация по продажам и воронке лидов (id=14)
```

## Нерешенные вопросы

- TODO: определить полный список источников данных.
- TODO: определить полный список n8n workflow.
- TODO: определить полный список PHP cron jobs.
- TODO: выгрузить список Superset datasets.
- TODO: выгрузить список Superset charts и dashboards.
