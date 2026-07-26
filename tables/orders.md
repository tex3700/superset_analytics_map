# ADB Table: orders

---
type: adb_table
name: orders
schema: adb
database: ADB
updated_by:
  - php_cron:adb_cron_php
source_tables:
  - madeindream.order
  - madeindream.order_status
  - crm_mid2.crm_task
  - crm_mid2.crm_task_status
  - crm_mid2.crm_task_grant
  - crm_mid2.crm_user
used_by_datasets:
  - ya_metrica_users_goals_detailed
  - ad_stats_and_roi
  - mart_lid_orders
  - con_lids_orders
  - rel_lid_orders
  - source_abadoned_orders
---

## Назначение

Таблица содержит заказы интернет-магазина в ADB и дополняет их данными, нужными для аналитики:

- `_ym_uid` из cookie заказа для связи с Яндекс.Метрикой;
- статус заказа;
- CRM task ID;
- статус CRM-задачи;
- менеджер / исполнитель CRM-задачи.

## Источник и обновление

- Процесс: [ADB cron PHP](../ingestion/adb-cron-php.md)
- Функция в PHP: `move_orders`
- Расписание: ежедневно в 06:30
- Режим обновления: `TRUNCATE -> batch INSERT`
- Источник заказов: MID table `order`
- Источник статуса заказа: MID table `order_status`
- Источник CRM-статуса и менеджера: CRM tables `crm_task`, `crm_task_status`, `crm_task_grant`, `crm_user`

## Grain

Одна строка соответствует:

```text
1 заказ магазина
```

Ключевая бизнес-сущность:

```text
shop_order_id
```

## Поля

| Поле ADB | Источник / логика | Описание |
|---|---|---|
| `shop_order_id` | `order.order_id` | ID заказа интернет-магазина |
| `ym_uid` | `_ym_uid` из `order.ym_cookie` | ClientID Яндекс.Метрики для атрибуции заказа |
| `date_added` | `order.date_added` | Дата создания заказа |
| `date_modified` | `order.date_modified` | Дата изменения заказа |
| `shop_customer_id` | `order.customer_id` | ID покупателя в магазине |
| `order_status` | `order_status.name` | Статус заказа |
| `telephone` | `order.telephone` | Телефон покупателя |
| `crm_task_id` | `order.crm_task_id` | Связанная CRM-задача |
| `total` | `order.total` | Сумма заказа из магазина |
| `crm_task_status` | `crm_task_status.name` | Статус CRM-задачи |
| `manager` | `crm_user.username` where `crm_task_grant.is_executor = 1` | Исполнитель CRM-задачи |

## Логика `_ym_uid`

PHP-скрипт извлекает `_ym_uid` из поля `ym_cookie` регулярным выражением:

```text
\[_ym_uid\]\s*=>\s*(\d+)
```

Это поле используется в датасете `ya_metrica_users_goals_detailed` для связи:

```text
ya_metrica_users_goals_detailed.client_id = orders.ym_uid
```

## Используется в датасетах

| Датасет | Роль |
|---|---|
| [ya_metrica_users_goals_detailed](../datasets/ya_metrica_users_goals_detailed.md) | Связь пользователей Метрики с заказами |
| [ad_stats_and_roi](../datasets/ad_stats_and_roi.md) | Атрибуция выручки рекламным площадкам через `ym_uid` |
| [mart_lid_orders](../datasets/mart_lid_orders.md) | Основной источник заказов, статусов, CRM task, менеджера и `ym_uid` |
| [con_lids_orders](../datasets/con_lids_orders.md) | Источник дневного количества заказов новых покупателей |
| [rel_lid_orders](../datasets/rel_lid_orders.md) | Источник заказов новых покупателей, связанных с валидными лидами |
| [source_abadoned_orders](../datasets/source_abadoned_orders.md) | Fallback-источник `ym_uid` по `crm_task_id + telephone` |

## Особенности

- Таблица полностью пересобирается каждый день.
- Если `_ym_uid` отсутствует в cookie заказа, `ym_uid` будет пустым и заказ не сможет связаться с пользователем Метрики по этой логике.
- CRM-статус и менеджер подмешиваются на момент выполнения cron.
