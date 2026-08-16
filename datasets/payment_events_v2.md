# Dataset: payment_events_v2

---
type: dataset
name: payment_events_v2
superset_dataset_id: 34
superset_type: physical_view
database: ADB
database_id: 4
schema: adb
physical_table: adb.payment_events_v2
depends_on:
  - adb.mid_analytics_payment_event_outbox
  - adb.mid_analytics_payment_reconciliation
used_by_charts:
  - sobytiya-payment-outbox-i-sverka
---

## Краткое описание

Диагностический payment-v2 dataset на уровне outbox events.

Статус на 2026-08-14: ADB view создана, Superset dataset создан с ID `34`.

Главный grain:

```text
1 строка = 1 event_id
```

Dataset нужен для проверки доставки `order_paid`, `order_paid_v2`, `purchase_paid`, reconciliation-статусов и `attribution_trust`.

## Что отображает

Dataset показывает все payment measurement events как события outbox. Он отвечает на вопросы:

- сколько технических событий создано по каждой оплате;
- какие события доставлены в Метрику/ecommerce;
- какие события имеют `trusted` / `untrusted`;
- что показала reconciliation-сверка;
- есть ли failed/pending статусы доставки.

Источник SQL для Superset: ADB view `adb.payment_events_v2`. Операционные SQL-файлы внедрения хранятся вне публичной документации; в документации фиксируется уже развернутый ADB-объект и его семантика.

## Важное ограничение

Этот dataset нельзя использовать для финансовой суммы продаж напрямую: одна оплата может иметь несколько event rows. Для gross paid использовать [order_payments_v2](./order_payments_v2.md).

## Поля и русские labels

| Поле | Label в Superset | Источник / формула | Описание |
|---|---|---|---|
| `event_id` | ID события | `mid_analytics_payment_event_outbox.event_id` | Уникальное outbox-событие |
| `event_name` | Тип события | `outbox.event_name` | `order_paid`, `order_paid_v2`, `purchase_paid` |
| `order_id` | ID заказа | `outbox.order_id` | ID заказа MID |
| `payment_id` | ID платежа | `outbox.payment_id` | ID платежа / surrogate `operator_verified:<order_id>` |
| `paid_at` | Время оплаты | `outbox.paid_at` | Время подтверждения оплаты |
| `paid_date` | Дата оплаты | `DATE(outbox.paid_at)` | Дата подтверждения оплаты |
| `amount` | Сумма события | `outbox.amount` | Сумма конкретного event row |
| `currency` | Валюта | `outbox.currency` | Валюта суммы события |
| `client_id` | ClientID Метрики | `outbox.client_id` | ClientID, сохраненный backend |
| `yclid` | YCLID | `outbox.yclid` | Click ID Яндекс.Директа |
| `utm_source` | UTM source | `outbox.utm_source` | UTM source из backend |
| `utm_medium` | UTM medium | `outbox.utm_medium` | UTM medium из backend |
| `utm_campaign` | UTM campaign / CampaignId | `outbox.utm_campaign` | UTM campaign; для Direct может быть CampaignId |
| `utm_content` | UTM content | `outbox.utm_content` | UTM content |
| `utm_term` | UTM term | `outbox.utm_term` | UTM term |
| `payment_source` | Источник оплаты | `outbox.source` | Источник бизнес-факта, например `operator_verified` |
| `attribution_trust` | Доверие к ClientID | `outbox.attribution_trust` | `trusted` / `untrusted`; не равно доказанной кампании |
| `outbox_status` | Статус outbox | `outbox.status` | `sent`, `pending`, `failed` и т.п. |
| `attempts` | Попытки отправки | `outbox.attempts` | Количество попыток доставки |
| `last_http_code` | Последний HTTP-код | `outbox.last_http_code` | HTTP-код последней отправки |
| `last_error_code` | Последняя ошибка | `outbox.last_error_code` | Код ошибки доставки |
| `outbox_created_at` | Создано в outbox | `outbox.created_at` | Время создания outbox-события |
| `sent_at` | Отправлено | `outbox.sent_at` | Время отправки события |
| `next_attempt_at` | Следующая попытка | `outbox.next_attempt_at` | Планируемая повторная отправка |
| `adb_synced_at` | Синхронизировано в ADB | `outbox.adb_synced_at` | Время переноса в ADB |
| `reconciliation_id` | ID сверки | `reconciliation.reconciliation_id` | ID строки reconciliation |
| `reconciliation_confirmed_at` | Подтверждено сверкой | `reconciliation.confirmed_at` | Время подтверждения в reconciliation |
| `reconciliation_sent_at` | Отправлено по сверке | `reconciliation.sent_at` | Время отправки по reconciliation |
| `goal_http_code` | HTTP-код цели | `reconciliation.goal_http_code` | HTTP-код доставки цели |
| `goal_readback_ok` | Read-back цели OK | `reconciliation.goal_readback_ok` | Флаг успешного read-back |
| `purchase_seen` | Purchase найден | `reconciliation.purchase_seen` | Флаг наличия `purchase_paid` для оплаты |
| `reconciliation_amount` | Сумма сверки | `reconciliation.amount` | Сумма в reconciliation |
| `reconciliation_currency` | Валюта сверки | `reconciliation.currency` | Валюта в reconciliation |
| `reconciliation_classification` | Классификация сверки | `reconciliation.classification` | Классификация reconciliation |
| `reconciled_status` | Статус сверки | `reconciliation.reconciled_status` | Статус reconciliation |
| `reconciled_attempts` | Попытки сверки | `reconciliation.reconciled_attempts` | Количество попыток reconciliation |
| `reconciliation_notes` | Примечания сверки | `reconciliation.notes` | Комментарии reconciliation |
| `reconciliation_updated_at` | Сверка обновлена | `reconciliation.updated_at` | Время обновления reconciliation |
| `is_canonical_payment_event` | Каноническое событие оплаты | `event_name='order_paid_v2' AND payment_source='operator_verified'` | Флаг события, которое создает финансовый факт |
| `attribution_untrusted` | Недоверенная атрибуция | `attribution_trust='untrusted'` | Числовой флаг untrusted |
| `grain` | Grain | constant | Описание grain dataset |
| `dataset_version` | Версия dataset | constant | `payment_events_v2` |

## Проверки качества

- trusted/untrusted event rows;
- наличие всех трех событий на оплату;
- failed/pending/sent статусы доставки;
- расхождения outbox и reconciliation.

## Read-back SQL

```sql
SELECT
  attribution_trust,
  event_name,
  currency,
  COUNT(*) AS event_rows,
  COUNT(DISTINCT order_id) AS orders,
  ROUND(SUM(amount), 2) AS amount_sum
FROM adb.payment_events_v2
GROUP BY attribution_trust, event_name, currency
ORDER BY attribution_trust, event_name, currency;
```

Примечание: денежные суммы группируются по `currency`, потому что разные валюты нельзя складывать в один total без FX-курса.

Предыдущий read-back на 2026-08-14 до валютного уточнения:

| `attribution_trust` | `event_name` | `event_rows` | `orders` | `amount_sum` |
|---|---|---:|---:|---:|
| `trusted` | `order_paid` | 199 | 199 | 3000486.90 |
| `trusted` | `order_paid_v2` | 199 | 199 | 3000486.90 |
| `trusted` | `purchase_paid` | 198 | 198 | 2961950.15 |
| `untrusted` | `order_paid` | 188 | 188 | 22317241.43 |
| `untrusted` | `order_paid_v2` | 188 | 188 | 22317241.43 |
| `untrusted` | `purchase_paid` | 188 | 188 | 22317241.43 |
