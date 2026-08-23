# ADB Table: mid_analytics_payment_event_outbox

---
type: table
name: mid_analytics_payment_event_outbox
database: ADB
schema: adb
source_system: MID / madeindream
loaded_by:
  - php_cron:adb_cron_php
used_by_datasets:
  - payment_events_v2
  - order_payments_v2
  - payment_campaign_attribution_v2
---

## Назначение

ADB-зеркало production-таблицы `madeindream.analytics_payment_event_outbox`.

Таблица нужна, потому что Superset по требованию заказчика должен строить datasets только из ADB, без прямого чтения production MID/madeindream.

## Grain

```text
1 строка = 1 payment event_id
```

В одной оплате может быть несколько событий:

```text
order_paid
order_paid_v2
purchase_paid
```

Поэтому эту таблицу нельзя использовать для суммы продаж напрямую: финансовый факт должен дедуплицироваться до `order_id + payment_id`.

## Ключи

| Ключ | Поля | Назначение |
|---|---|---|
| PRIMARY | `event_id` | Уникальное событие outbox |
| UNIQUE | `event_name`, `order_id`, `payment_id` | Дедупликация события одного типа для оплаты |

## Ключевые поля

| Поле | Описание |
|---|---|
| `event_name` | `order_paid`, `order_paid_v2`, `purchase_paid` |
| `order_id` | ID заказа MID |
| `payment_id` | ID платежа; сейчас встречается формат `operator_verified:<order_id>` |
| `paid_at` | Время подтверждения оплаты |
| `amount` | Сумма заказа/события; не всегда фактически полученная сумма |
| `amount_received` | Фактически полученная сумма, если backend смог ее подтвердить |
| `amount_quality` | Качество суммы: `exact`, `partial`, `unknown`, `base_currency` |
| `currency` | Валюта |
| `client_id` | ClientID Яндекс.Метрики, если доступен |
| `yclid` | Click ID Яндекс.Директа, если доступен |
| `utm_*` | UTM-метки, сохраненные на стороне backend |
| `source` | Источник бизнес-факта оплаты, например `operator_verified` |
| `attribution_trust` | Канонический trust-флаг backend: `trusted` / `untrusted` |
| `status` | Статус доставки outbox |
| `last_http_code` | Последний HTTP-код доставки |
| `sent_at` | Время доставки события |
| `adb_synced_at` | Время переноса строки в ADB |

## Обновление

- Процесс: [ADB cron PHP](../ingestion/adb-cron-php.md)
- Функция: `move_mid_payment_event_outbox`
- Режим: `TRUNCATE -> batch INSERT`
- Источник: `madeindream.analytics_payment_event_outbox`

## Важные правила

- `attribution_trust = trusted` означает только то, что ClientID не признан общим/заимствованным.
- `trusted` не означает доказанную связь с рекламной кампанией.
- `attribution_trust = untrusted` не удаляется из финансового факта, но исключается из доказанных campaign CPA/ROAS/CAC.
- Канонический финансовый факт строится не по всем event rows, а по первому `operator_verified order_paid_v2` на grain `order_id + payment_id`.
- Для доказанной денежной revenue используется `amount_received` только при `amount_quality IN ('exact','partial')`.
- `amount_quality IN ('unknown','base_currency')` показывается отдельно и не входит в proven revenue/ROAS.
