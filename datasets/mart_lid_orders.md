# Dataset: mart_lid_orders

---
type: dataset
name: mart_lid_orders
superset_dataset_id: 19
superset_type: virtual
database: ADB
database_id: 4
schema: adb
owner: Дмитрий Пыланкин
depends_on:
  - adb.orders
  - adb.customers
  - adb.contact_events
  - adb.orders_totals
  - adb.ya_metrica_users_goals_detailed
used_by_charts:
  - analitika_lidov_po_zakazam
  - obshchaya_vyruchka_za_period
  - obshchaya_pribyl_za_period
  - dinamika_vyruchka_i_pribyl_po_dnyam
  - novye_i_loyalnye_pokupateli_za_period
  - kolichestvo_zakazov_za_period
  - srednee_vremya_ot_registratsii_lida_do_pervogo_zakaza_v_chasah
  - sredniy_chek_aov_po_zakazam_za_period
  - raspredelenie_menedzherov_po_zakazam
---

## Краткое описание

Virtual dataset связывает заказы интернет-магазина с CRM-лидами, событиями контакта, источниками Яндекс.Метрики и финансовыми строками заказа.

Используется для детализации продаж и воронки лидов: кто купил, какой был контакт/лид, какой источник касания, сколько заказов у покупателя, сколько времени прошло до первого заказа, какая выручка/себестоимость/прибыль.

## Superset

- Dataset ID: `19`
- Dataset name: `mart_lid_orders`
- Dataset type: virtual
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/19`
- Owner: Дмитрий Пыланкин
- Main datetime column: `date_task_added`

## Источники данных

| Источник | Процесс загрузки | Таблицы ADB |
|---|---|---|
| MID / CRM databases | `ADB cron PHP` | `adb.orders`, `adb.customers`, `adb.contact_events`, `adb.orders_totals` |
| Yandex Metrika API | `Yandex metrica stats UTM and Users` | `adb.ya_metrica_users_goals_detailed` |

## SQL датасета

SQL датасета длинный и содержит много коррелированных подзапросов. Полный SQL хранится в Superset dataset `19`.

Ключевая структура:

```text
FROM adb.orders o
LEFT JOIN adb.customers c
  ON o.shop_customer_id = c.shop_customer_id
LEFT JOIN derived pl
  ON pl.contact_id = c.contact_id
 AND pl.crm_task_id = o.crm_task_id
```

Дополнительные данные берутся подзапросами из:

```text
adb.contact_events
adb.ya_metrica_users_goals_detailed
adb.orders
adb.orders_totals
```

## Grain

Одна строка датасета соответствует:

```text
1 неотмененный заказ магазина
```

Ключевая бизнес-сущность:

```text
shop_order_id
```

Уникальность проверена через SQL-проверки в ADB:

- после фильтров датасета в `adb.orders`: `76430` строк и `76430` уникальных `shop_order_id`;
- в `adb.customers`: `115686` строк и `115686` уникальных `shop_customer_id`;
- derived-блок `pl`: `89131` строка и `89131` уникальная пара `contact_id + crm_task_id`.

На основании этих проверок join-цепочка датасета не должна размножать строки заказов.

## Основные фильтры внутри SQL

Исключаются покупки, пришедшие из маркетплейсов.

В MID такие покупки проходят как заказы от специального покупателя, где каждому маркетплейсу или кабинету маркетплейса соответствует свой `customer_id`. Для аналитики продаж и лидовой воронки эти заказы исключаются:

| `shop_customer_id` | Marketplace / кабинет |
|---|---|
| `111661` | kaspi |
| `77219` | ozon fbs |
| `84811` | ozon kz fbs |
| `76752` | яндекс fbs |
| `95888` | яндекс fbs |
| `26278` | wildberries |
| `107545` | wildberries |
| `90224` | wildberries KZ |
| `17536` | goods Москва |
| `50043` | goods СпБ |
| `106532` | яндекс Экспресс МСК |
| `59522` | яндекс Экспресс Спб |
| `88147` | mvideo fbs |

Исключаются статусы заказов:

```text
Отменен
Отмена (дублирующийся заказ)
Неудавшийся
Возврат
Возврат б\у
Возврат средств
```

## Поля

| Поле | Описание | Источник / логика |
|---|---|---|
| `date_task_added` | Дата события CRM-задачи заказа | `MIN(contact_events.date_added)` по `contact_id + crm_task_id` |
| `date_order_added` | Дата заказа | `orders.date_added` |
| `date_lid_added` | Дата создания лида | `customers.date_contact_added` или первое событие контакта |
| `contact_id` | ID CRM-контакта | `customers.contact_id` |
| `customer_id` | ID покупателя магазина | `orders.shop_customer_id` |
| `ym_uid` | ClientID Яндекс.Метрики | `orders.ym_uid` |
| `task_id` | ID CRM-задачи заказа | `orders.crm_task_id` |
| `shop_order_id` | ID заказа магазина | `orders.shop_order_id` |
| `source_from_task` | Источник из задачи | первое `contact_events.task_name` по `contact_id + crm_task_id` |
| `source_task_from_ym` | Источник задачи из Метрики | `last_sign_utm_source | last_sign_utm_medium` для `orders.ym_uid` в периоде заказа |
| `date_stf_ym` | Дата источника задачи из Метрики | `ya_metrica_users_goals_detailed.stat_date` |
| `source_customer_from_ym` | Источник покупателя из Метрики | `first_utm_source | first_utm_medium | first_traffic_source` |
| `date_scf_ym` | Дата первого перехода покупателя из Метрики | `first_visit_date` |
| `source_lid_from_ym` | Источник лида из Метрики | источник первого валидного заказа покупателя |
| `date_slf_ym` | Дата источника лида из Метрики | `first_visit_date` по первому валидному заказу |
| `count_orders` | Количество валидных заказов покупателя | `COUNT(*)` по `orders.shop_customer_id` |
| `date_first_order` | Дата первого валидного заказа покупателя | `MIN(orders.date_added)` |
| `order_status` | Статус текущего заказа | `orders.order_status` |
| `task_status` | Статус CRM-задачи | `orders.crm_task_status` |
| `first_touch_lid_order` | Дата задачи последнего лида перед заказом | derived `pl` по `contact_events.is_project_lid = 1` |
| `first_lid_touch` | Первое касание лида | минимум из дат контакта, источника Метрики и события лида |
| `datetime_to_first_order` | Текстовая длительность до первого заказа | дни/часы между `first_lid_touch` и `date_first_order` |
| `hours_to_first_order_numeric` | Часы до первого заказа | `TIMESTAMPDIFF(HOUR, first_lid_touch, date_first_order)` с нижней границей `0` |
| `manager` | Менеджер по заказу | `orders.manager` |
| `revenue` | Выручка по заказу | `orders_totals.value`, где `code = 'total'` |
| `cost` | Себестоимость по заказу | `orders_totals.value`, где `code = 'cost'` |
| `shipping` | Доставка | `orders_totals.value`, где `code = 'shipping'` |
| `profit` | Прибыль по заказу | `total - cost` |
| `profit_total` | Calculated column в Superset | см. настройки Superset column |

## Агрегации внутри датасета

- `count_orders`: количество валидных заказов покупателя.
- `date_first_order`: первый валидный заказ покупателя.
- `date_task_added`: первое событие контакта по CRM-задаче заказа.
- `first_lid_touch`: минимальная дата из нескольких возможных касаний лида.
- `profit`: разница между `total` и `cost`.

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Аналитика лидов по заказам](../charts/analitika-lidov-po-zakazam.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | Raw table |
| [Общая выручка за период](../charts/obshchaya-vyruchka-za-period.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `SUM(revenue)` |
| [Общая прибыль за период](../charts/obshchaya-pribyl-za-period.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `SUM(profit)` |
| [Динамика выручка и прибыль по дням](../charts/dinamika-vyruchka-i-pribyl-po-dnyam.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `SUM(revenue)`, `SUM(profit)` по дням |
| [Новые и лояльные покупатели за период](../charts/novye-i-loyalnye-pokupateli-za-period.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `COUNT(DISTINCT customer_id)` по сегменту |
| [Количество заказов за период](../charts/kolichestvo-zakazov-za-period.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `COUNT(*)` |
| [Среднее время от регистрации лида до первого заказа (в часах)](../charts/srednee-vremya-ot-registratsii-lida-do-pervogo-zakaza-v-chasah.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `AVG(hours_to_first_order_numeric)` |
| [Средний чек (AOV) по заказам за период](../charts/sredniy-chek-aov-po-zakazam-za-period.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `AVG(revenue)` |
| [Распределение менеджеров по заказам](../charts/raspredelenie-menedzherov-po-zakazam.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `COUNT(*)` по нормализованному менеджеру |


