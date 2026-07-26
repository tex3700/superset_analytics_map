# ADB Table: customers

---
type: adb_table
name: customers
schema: adb
database: ADB
updated_by:
  - php_cron:adb_cron_php
source_tables:
  - madeindream.customer
  - madeindream.user
  - crm_mid2.crm_contact
used_by_datasets:
  - mart_lid_orders
  - rel_lid_orders
---

## Назначение

Таблица связывает покупателя интернет-магазина с CRM-контактом.

Используется в витрине `mart_lid_orders`, чтобы от заказа перейти к `contact_id` и событиям CRM.

## Источник и обновление

- Процесс: [ADB cron PHP](../ingestion/adb-cron-php.md)
- Функции в PHP: `move_customers`, затем `add_contact_id_to_customer`
- Расписание: ежедневно в 06:30
- Режим обновления: `TRUNCATE -> batch INSERT`, затем UPDATE `contact_id` и `date_contact_added`

## Grain

Одна строка соответствует:

```text
1 покупатель магазина
```

Ключевая бизнес-сущность:

```text
shop_customer_id
```

## Поля

| Поле ADB | Источник / логика | Описание |
|---|---|---|
| `shop_customer_id` | `customer.customer_id` | ID покупателя магазина |
| `date_added` | `customer.date_added` | Дата регистрации покупателя в магазине |
| `telephone` | `customer.telephone` | Телефон покупателя |
| `firstname` | `customer.firstname` | Имя |
| `lastname` | `customer.lastname` | Фамилия |
| `username` | `user.username` по `customer.user_id` | Пользователь/менеджер из магазина |
| `contact_id` | CRM `crm_contact.contact_id`, матч по нормализованному телефону | ID CRM-контакта |
| `date_contact_added` | `crm_contact.date_added` | Дата создания CRM-контакта |

## Используется в датасетах

| Датасет | Роль |
|---|---|
| [mart_lid_orders](../datasets/mart_lid_orders.md) | Связь заказа с CRM-контактом и датой создания лида |
| [rel_lid_orders](../datasets/rel_lid_orders.md) | Связь `mart_lids.contact_id` с `shop_customer_id` для конверсии лида в заказ |
