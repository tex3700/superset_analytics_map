# ADB Table: contact_events

---
type: adb_table
name: contact_events
schema: adb
database: ADB
updated_by:
  - php_cron:adb_cron_php
source_tables:
  - crm_mid2.crm_task_event
  - crm_mid2.crm_contact_history
  - crm_mid2.crm_thread
  - crm_mid2.crm_phonecall
  - crm_mid2.crm_task_to_project
used_by_datasets:
  - mart_lid_orders
---

## Назначение

Таблица собирает события CRM-контактов из нескольких CRM-источников в единый поток событий.

В `mart_lid_orders` используется для:

- поиска даты события по задаче заказа;
- определения источника из задачи;
- поиска первого касания лида;
- определения событий, относящихся к проектам "ЛИДЫ".

## Источник и обновление

- Процесс: [ADB cron PHP](../ingestion/adb-cron-php.md)
- Функция в PHP: `move_contact_events`
- Расписание: ежедневно в 06:30
- Режим обновления: `TRUNCATE -> batch INSERT`

## Источники событий

| Тип источника | CRM tables | Что попадает в ADB |
|---|---|---|
| Task event | `crm_task_event`, `crm_task`, `crm_task_description`, `crm_contact_to_task` | события задач |
| Contact history | `crm_contact_history` | история контакта |
| Auto/thread templates | `crm_thread`, `crm_task`, `crm_task_description`, `crm_contact_to_task` | авто-сообщения / шаблоны |
| Regular thread | `crm_thread`, `crm_task`, `crm_task_description`, `crm_contact_to_task` | обычная переписка |
| Phonecall | `crm_phonecall` | звонки контакта |

## Grain

Одна строка соответствует:

```text
1 событие CRM-контакта
```

## Поля

| Поле ADB | Источник / логика | Описание |
|---|---|---|
| `task_name` | имя задачи / тип события | Название события или задачи |
| `message` | текст события | Сообщение / комментарий / описание |
| `crm_task_id` | CRM task id, если есть | ID CRM-задачи |
| `date_added` | дата события | Время события |
| `username` | CRM user | Пользователь, связанный с событием |
| `contact_id` | CRM contact id | ID контакта |
| `is_project_lid` | `1`, если задача входит в проект `591` или его дочерние проекты | Флаг лида |

## Используется в датасетах

| Датасет | Роль |
|---|---|
| [mart_lid_orders](../datasets/mart_lid_orders.md) | Источник CRM-событий, касаний и источника задачи |

