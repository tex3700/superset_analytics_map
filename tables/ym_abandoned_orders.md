# ADB Table: ym_abandoned_orders

---
type: adb_table
name: ym_abandoned_orders
schema: adb
database: ADB
updated_by:
  - php_cron:adb_cron_php
source_tables:
  - madeindream.abandoned2_order
used_by_datasets:
  - source_abadoned_orders
---

## Назначение

Таблица хранит брошенные заказы из магазина с извлеченным `ym_uid`, CRM task ID и телефоном.

Используется для привязки брошенного заказа к пользователю Яндекс.Метрики и дальнейшего определения источника.

## Источник и обновление

- Процесс: [ADB cron PHP](../ingestion/adb-cron-php.md)
- Функция в PHP: `update_ym_abandoned_orders`
- Расписание: ежедневно в 06:30
- Режим обновления: `TRUNCATE -> batch INSERT`
- Источник: MID table `abandoned2_order`

## Фильтры загрузки

В ADB попадают строки, где:

- `crm_task_id IS NOT NULL`;
- `crm_task_id > 0`;
- `ym_cookie` не пустой;
- `date_added >= NOW() - INTERVAL 1 YEAR`.

## Поля

| Поле ADB | Источник / логика | Описание |
|---|---|---|
| `date_added` | `abandoned2_order.date_added` | Дата брошенного заказа |
| `date_modified` | `abandoned2_order.date_modified` | Дата изменения |
| `phone` | `abandoned2_order.telephone`, пробелы удаляются | Телефон |
| `ym_uid` | `_ym_uid` из `ym_cookie` | ClientID Яндекс.Метрики |
| `crm_task_id` | `abandoned2_order.crm_task_id` | CRM-задача |
| `sent_to_crm` | `abandoned2_order.sent_to_crm` | Флаг отправки в CRM |

## Логика `_ym_uid`

PHP-скрипт извлекает `_ym_uid` из `ym_cookie` регулярным выражением:

```text
\[_ym_uid\]\s*=>\s*(\d+)
```

## Используется в датасетах

| Датасет | Роль |
|---|---|
| [source_abadoned_orders](../datasets/source_abadoned_orders.md) | Приоритетный источник `ym_uid` для брошенных заказов |

