# Dataset: mart_lids

---
type: dataset
name: mart_lids
superset_dataset_id: 20
superset_type: physical
database: ADB
database_id: 4
schema: adb
physical_table: mart_lids
owner: Дмитрий Пыланкин
depends_on:
  - php_cron:adb_cron_php
  - crm_mid2.crm_project
  - crm_mid2.crm_task
  - crm_mid2.crm_task_status
  - crm_mid2.crm_contact
  - crm_mid2.crm_phonecall
  - crm_mid2.crm_task_property
  - crm_mid2.crm_contact_property
used_by_charts:
  - detalizatsiya_po_lidam
  - istochniki_lidov_po_crm_za_period
  - raspredelenie_menedzherov_po_lidam
used_by_datasets:
  - con_lids_orders
  - rel_lid_orders
  - source_abadoned_orders
---

## Краткое описание

Physical dataset Superset на таблице `adb.mart_lids`.

Таблица содержит витрину CRM-лидов: контакт, задача, проект, статус, свойства лида, менеджеры и количество отвеченных звонков.

## Superset

- Dataset ID: `20`
- Dataset name: `mart_lids`
- Dataset type: physical
- Database: `ADB`
- Database ID: `4`
- Schema: `adb`
- Physical table: `mart_lids`
- URL: `http://81.200.152.123:8088/tablemodelview/edit/20`
- Owner: Дмитрий Пыланкин
- Main datetime column: `lid_date_added`

## Источники данных

| Источник | Процесс загрузки | Таблица ADB |
|---|---|---|
| CRM database `crm_mid2` | PHP cron `adb.php`, функция `update_mart_lids` | `adb.mart_lids` |

## SQL датасета

Датасет physical, собственного SQL в Superset нет.

Источник данных:

```text
ADB.adb.mart_lids
```

## Логика загрузки в ADB

Таблица пересобирается PHP cron:

```text
TRUNCATE TABLE mart_lids
  -> SELECT из CRM
  -> batch INSERT
```

Функция `update_mart_lids` выбирает задачи из проекта CRM:

```text
crm_project.project_id = 591 OR parent_id = 591
```

Проект `591` подтвержден как корневой CRM-проект "ЛИДЫ"; в выборку входят он сам и все дочерние проекты.

Основные источники в CRM:

| CRM table | Роль |
|---|---|
| `crm_task_to_project` | связь задачи с проектом лидов |
| `crm_task` | задача лида |
| `crm_task_status` | статус задачи |
| `crm_contact_to_task` | связь контакта и задачи |
| `crm_contact` | данные контакта / лида |
| `crm_user` | менеджеры контакта и задачи |
| `crm_project_description` | название проекта |
| `crm_task_property` | свойство задачи `property_id = 319`, тип заявки |
| `crm_contact_property` | свойство контакта `property_id = 151`, тип лида/контакта |
| `crm_phonecall` | количество отвеченных звонков по контакту |

## Grain

Ожидаемый grain:

```text
1 CRM contact / lead
```

В SQL загрузки используется `GROUP BY cc.contact_id`, поэтому одна строка витрины соответствует одному `contact_id`.

Уникальность проверена в ADB:

```text
total_rows = 20191
distinct_contact_ids = 20191
duplicate_contact_rows = 0
```

## Поля

| Поле | Описание | Источник / логика |
|---|---|---|
| `id` | Технический ID строки | ADB |
| `contact_id` | ID лида/контакта | `crm_contact.contact_id` |
| `task_id` | ID задачи по лиду | `crm_task.task_id` |
| `lid_date_added` | Дата создания лида/контакта | `crm_contact.date_added` |
| `task_date_added` | Дата задачи по лиду | `crm_task.date_added` |
| `project_id` | ID проекта CRM | `crm_project_description.project_id` |
| `project_name` | Название проекта CRM | `crm_project_description.name` |
| `lid_reason` | Причина создания лида/контакта | `crm_contact.reason` |
| `task_status` | Статус задачи | `crm_task_status.name` |
| `task_property` | Тип заявки по задаче | `crm_task_property.text`, `property_id = 319` |
| `lid_property` | Тип лида/контакта | `crm_contact_property.text`, `property_id = 151` |
| `lid_manager` | Менеджер контакта | `crm_user.username` по `crm_contact.user_id` |
| `task_manager` | Менеджер задачи | исполнитель из `crm_task_grant.is_executor = 1` |
| `lid_firstname` | Имя лида | `crm_contact.firstname` |
| `lid_lastname` | Фамилия лида | `crm_contact.lastname` |
| `lid_email` | Email лида | `crm_contact.email` |
| `lid_phone` | Телефон лида | `crm_contact.telephone` |
| `lid_company` | Компания лида | `crm_contact.company` |
| `lid_phone_calls` | Количество отвеченных звонков | `COUNT(*)` из `crm_phonecall` where `disposition = 'answered'` |

## Метрики в датасете

В Superset определена только стандартная метрика:

| Метрика | Формула | Описание |
|---|---|---|
| `count` | `COUNT(*)` | Количество строк / лидов |

## Обновление данных

- Частота обновления: ежедневно
- Время запуска: 06:30
- Процесс: PHP cron `adb.php`
- Функция: `update_mart_lids`
- Режим обновления: полная пересборка таблицы

## Используется в чартах

| Чарт | Дашборд | Как агрегирует данные |
|---|---|---|
| [Детализация по лидам](../charts/detalizatsiya-po-lidam.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | Raw table |
| [Источники лидов по CRM за период](../charts/istochniki-lidov-po-crm-za-period.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `COUNT(*)` по `project_name` |
| [Распределение менеджеров по лидам](../charts/raspredelenie-menedzherov-po-lidam.md) | [Детализация по продажам и воронке лидов](../dashboards/detalizatsiya-po-prodazham-i-voronke-lidov.md) | `COUNT(*)` по нормализованному `task_manager` |

## Используется в других датасетах

| Датасет | Роль |
|---|---|
| [con_lids_orders](../datasets/con_lids_orders.md) | Источник дневного количества лидов |
| [rel_lid_orders](../datasets/rel_lid_orders.md) | Источник валидных лидов и связи `contact_id -> shop_customer_id` через `customers` |
| [source_abadoned_orders](../datasets/source_abadoned_orders.md) | Источник лидов из проектов брошенных заказов `648` и `667` |

