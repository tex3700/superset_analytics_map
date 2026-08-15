# Dataset: order_payments_v2

---
type: dataset
name: order_payments_v2
superset_dataset_id: 35
superset_type: physical_view
database: ADB
database_id: 4
schema: adb
physical_table: adb.order_payments_v2
depends_on:
  - adb.mid_analytics_payment_event_outbox
  - adb.mid_analytics_payment_reconciliation
used_by_charts:
  - kanonicheskie-oplaty-payment-v2
---

## Краткое описание

Основной payment-v2 dataset для финансового факта оплаты.

Статус на 2026-08-14: ADB view создана, Superset dataset создан с ID `35`.

Канонический grain:

```text
1 строка = 1 order_id + payment_id
```

Финансовый факт создается только первым `operator_verified order_paid_v2`. `purchase_paid` используется как контроль ecommerce-доставки, полноты, суммы и валюты, но не создает вторую продажу.

## Что отображает

Dataset показывает один канонический финансовый факт оплаты на пару `order_id + payment_id`.

Он отвечает на вопросы:

- сколько уникальных оплат подтверждено оператором;
- какая gross paid сумма по оплатам;
- какие оплаты trusted/untrusted по ClientID;
- у каких оплат нет sibling `purchase_paid`;
- есть ли расхождения суммы/валюты между `order_paid_v2` и sibling events.

Источник SQL для Superset: ADB view `adb.order_payments_v2`.

## Финансовая семантика

Текущий показатель:

```text
gross_paid_amount
```

Пока нет отдельного источника refund/cancel-событий и сумм, dataset не заявляет `net_paid`.

## Поля и русские labels

| Поле | Label в Superset | Источник / формула | Описание |
|---|---|---|---|
| `order_id` | ID заказа | canonical `order_paid_v2.order_id` | ID заказа MID |
| `payment_id` | ID платежа | canonical `order_paid_v2.payment_id` | ID платежа |
| `canonical_event_id` | ID канонического события | canonical `order_paid_v2.event_id` | Событие, создавшее финансовый факт |
| `paid_at` | Время оплаты | canonical `order_paid_v2.paid_at` | Время подтверждения оплаты |
| `paid_date` | Дата оплаты | `DATE(paid_at)` | Дата оплаты |
| `gross_paid_amount` | Gross paid сумма | canonical `order_paid_v2.amount` | Каноническая сумма оплаты до refund/cancel |
| `currency` | Валюта | canonical `order_paid_v2.currency` | Валюта оплаты |
| `client_id` | ClientID Метрики | canonical `order_paid_v2.client_id` | ClientID оплаты |
| `yclid` | YCLID | canonical `order_paid_v2.yclid` | Click ID Яндекс.Директа |
| `utm_source` | UTM source | canonical `order_paid_v2.utm_source` | UTM source оплаты |
| `utm_medium` | UTM medium | canonical `order_paid_v2.utm_medium` | UTM medium оплаты |
| `utm_campaign` | UTM campaign / CampaignId | canonical `order_paid_v2.utm_campaign` | UTM campaign; для Direct может быть CampaignId |
| `utm_content` | UTM content | canonical `order_paid_v2.utm_content` | UTM content |
| `utm_term` | UTM term | canonical `order_paid_v2.utm_term` | UTM term |
| `payment_source` | Источник оплаты | canonical `order_paid_v2.source` | Должен быть `operator_verified` |
| `payment_attribution_trust` | Доверие к ClientID оплаты | `untrusted`, если любое sibling-событие untrusted; иначе trust канонического события | Качество ClientID, не качество кампании |
| `attribution_untrusted` | Недоверенная атрибуция | `payment_attribution_trust='untrusted'` | Числовой флаг |
| `outbox_event_rows` | Строк outbox по оплате | count sibling events | Количество event rows по `order_id + payment_id` |
| `distinct_event_ids` | Уникальных событий | count distinct `event_id` | Контроль дублей |
| `has_order_paid_legacy` | Есть старый order_paid | max flag | Есть sibling `order_paid` |
| `has_order_paid_v2` | Есть order_paid_v2 | max flag | Есть canonical event type |
| `has_purchase_paid` | Есть purchase_paid | max flag | Есть ecommerce sibling |
| `purchase_paid_amount` | Сумма purchase_paid | sibling `purchase_paid.amount` | Сумма ecommerce purchase |
| `purchase_paid_currency` | Валюта purchase_paid | sibling `purchase_paid.currency` | Валюта ecommerce purchase |
| `purchase_paid_sent_at` | purchase_paid отправлен | sibling `purchase_paid.sent_at` | Время отправки ecommerce purchase |
| `amount_mismatch_flag` | Расхождение суммы | distinct amounts > 1 or purchase amount differs | Quality-флаг суммы |
| `currency_mismatch_flag` | Расхождение валюты | distinct currencies > 1 or purchase currency differs | Quality-флаг валюты |
| `missing_purchase_paid_flag` | Нет purchase_paid | `has_purchase_paid=0` | Quality-флаг полноты ecommerce |
| `canonical_outbox_status` | Статус канонического outbox | canonical `status` | Статус отправки `order_paid_v2` |
| `canonical_last_http_code` | HTTP-код канонического события | canonical `last_http_code` | Последний HTTP-код |
| `canonical_attempts` | Попытки канонического события | canonical `attempts` | Количество попыток отправки |
| `canonical_outbox_created_at` | Каноническое событие создано | canonical `created_at` | Время создания |
| `canonical_sent_at` | Каноническое событие отправлено | canonical `sent_at` | Время отправки |
| `adb_synced_at` | Синхронизировано в ADB | canonical `adb_synced_at` | Время переноса в ADB |
| `reconciliation_classification` | Классификация сверки | reconciliation aggregate | Классификация reconciliation |
| `reconciled_status` | Статус сверки | reconciliation aggregate | Статус reconciliation |
| `goal_readback_ok` | Read-back цели OK | reconciliation aggregate | Флаг read-back цели |
| `purchase_seen` | Purchase найден | reconciliation aggregate | Флаг наличия purchase |
| `reconciled_attempts` | Попытки сверки | reconciliation aggregate | Количество попыток сверки |
| `reconciliation_updated_at` | Сверка обновлена | reconciliation aggregate | Время обновления |
| `goal_http_code` | HTTP-код цели | reconciliation aggregate | HTTP-код цели |
| `grain` | Grain | constant | `1 order_id + payment_id` |
| `financial_fact_source` | Источник финансового факта | constant | `first operator_verified order_paid_v2` |
| `revenue_semantics` | Семантика выручки | constant | Gross paid only; refund/cancel пока нет |

## Правила использования

- В общую gross paid выручку входят и `trusted`, и `untrusted`.
- В доказанные campaign CPA/ROAS/CAC нельзя включать `payment_attribution_trust = untrusted`.
- `payment_attribution_trust = trusted` не означает доказанную кампанию; для этого нужен [payment_campaign_attribution_v2](./payment_campaign_attribution_v2.md) и `campaign_attribution_quality`.

## Read-back SQL

```sql
SELECT
  payment_attribution_trust,
  COUNT(*) AS payment_rows,
  COUNT(DISTINCT order_id) AS orders,
  ROUND(SUM(gross_paid_amount), 2) AS gross_paid_amount,
  SUM(missing_purchase_paid_flag) AS missing_purchase_paid_orders,
  SUM(amount_mismatch_flag) AS amount_mismatch_orders,
  SUM(currency_mismatch_flag) AS currency_mismatch_orders
FROM adb.order_payments_v2
GROUP BY payment_attribution_trust
ORDER BY payment_attribution_trust;
```

Результат read-back на 2026-08-14:

| `payment_attribution_trust` | `payment_rows` | `orders` | `gross_paid_amount` | `missing_purchase_paid_orders` | `amount_mismatch_orders` | `currency_mismatch_orders` |
|---|---:|---:|---:|---:|---:|---:|
| `trusted` | 199 | 199 | 3000486.90 | 1 | 0 | 0 |
| `untrusted` | 188 | 188 | 22317241.43 | 0 | 0 | 0 |
