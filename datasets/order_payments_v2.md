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
- какая сумма заказа указана в outbox;
- какая сумма фактически подтверждена как полученная;
- какое качество суммы (`exact`, `partial`, `unknown`, `base_currency`);
- какие оплаты trusted/untrusted по ClientID;
- у каких оплат нет sibling `purchase_paid`;
- есть ли расхождения суммы/валюты между `order_paid_v2` и sibling events.

Источник SQL для Superset: ADB view `adb.order_payments_v2`.

## Финансовая семантика

Текущие показатели после нового amount contract:

```text
order_amount
verified_received_amount
amount_quality
```

`order_amount` / legacy `gross_paid_amount` берется из `outbox.amount` и означает сумму заказа/события. Это поле нельзя молча считать фактически полученными деньгами, если `amount_quality` не подтверждает сумму.

`verified_received_amount` берется из `outbox.amount_received` только для:

- `amount_quality = exact`;
- `amount_quality = partial`.

Для `amount_quality = unknown` и `amount_quality = base_currency` полученная сумма не считается доказанной revenue и должна показываться отдельным bucket.

Пока нет отдельного источника refund/cancel-событий и сумм, dataset не заявляет `net_paid`.

## Валюты

Денежные суммы нельзя складывать между разными валютами без документированного FX-курса.

В текущем payment-v2 слое суммы показываются в исходной валюте оплаты:

```text
RUB / KZT / BYN
```

Все финансовые read-back запросы и charts должны группировать денежные показатели по `currency`, либо использовать отдельную документированную FX-конвертацию. На 2026-08-16 FX-конвертация не внедрена.

## Поля и русские labels

| Поле | Label в Superset | Источник / формула | Описание |
|---|---|---|---|
| `order_id` | ID заказа | canonical `order_paid_v2.order_id` | ID заказа MID |
| `payment_id` | ID платежа | canonical `order_paid_v2.payment_id` | ID платежа |
| `canonical_event_id` | ID канонического события | canonical `order_paid_v2.event_id` | Событие, создавшее финансовый факт |
| `paid_at` | Время оплаты | canonical `order_paid_v2.paid_at` | Время подтверждения оплаты |
| `paid_date` | Дата оплаты | `DATE(paid_at)` | Дата оплаты |
| `order_amount` | Сумма заказа | canonical `order_paid_v2.amount` | Сумма заказа/события по backend contract |
| `gross_paid_amount` | Legacy gross/order amount | canonical `order_paid_v2.amount` | Совместимость со старыми chart-полями; не использовать как доказанную received revenue без `amount_quality` |
| `amount_received` | Полученная сумма | canonical `order_paid_v2.amount_received` | Фактически полученная сумма из backend |
| `amount_quality` | Качество суммы | canonical `order_paid_v2.amount_quality` | `exact`, `partial`, `unknown`, `base_currency` |
| `verified_received_amount` | Подтвержденная полученная сумма | `amount_received`, если `amount_quality IN ('exact','partial')` | Денежное поле для доказанной revenue |
| `has_verified_received_amount` | Есть подтвержденная сумма | `amount_quality IN ('exact','partial')` | Флаг возможности использовать `verified_received_amount` |
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
- Для доказанной денежной revenue использовать `verified_received_amount`, а не `gross_paid_amount`.
- `amount_quality IN ('unknown','base_currency')` показывать отдельно и не включать в proven revenue/ROAS.
- В доказанные campaign CPA/ROAS/CAC нельзя включать `payment_attribution_trust = untrusted`.
- `payment_attribution_trust = trusted` не означает доказанную кампанию; для этого нужен [payment_campaign_attribution_v2](./payment_campaign_attribution_v2.md) и `campaign_attribution_quality`.

## Read-back SQL

```sql
SELECT
  payment_attribution_trust,
  currency,
  COUNT(*) AS payment_rows,
  COUNT(DISTINCT order_id) AS orders,
  amount_quality,
  ROUND(SUM(gross_paid_amount), 2) AS gross_paid_amount,
  ROUND(SUM(order_amount), 2) AS order_amount,
  ROUND(SUM(COALESCE(verified_received_amount, 0)), 2) AS verified_received_amount,
  SUM(missing_purchase_paid_flag) AS missing_purchase_paid_orders,
  SUM(amount_mismatch_flag) AS amount_mismatch_orders,
  SUM(currency_mismatch_flag) AS currency_mismatch_orders
FROM adb.order_payments_v2
GROUP BY payment_attribution_trust, currency, amount_quality
ORDER BY payment_attribution_trust, currency, amount_quality;
```

Read-back чистого окна 2026-08-13..2026-08-19, который должен совпасть после обновления ADB-зеркала и views:

| `amount_quality` | `currency` | rows | `order_amount` | `verified_received_amount` |
|---|---|---:|---:|---:|
| `exact` | `RUB` | 34 | 386634.15 | 386634.00 |
| `partial` | `RUB` | 1 | 44199.00 | 41105.07 |
| `unknown` | `RUB` | 20 | 270043.00 | 0.00 |
| `base_currency` | `BYN` | 2 | 1383.99 | 0.00 |
| `exact` + failed/no_client_id | `KZT` | 35 | 4143300.00 | 0.00 |

Результат независимой production-приемки на 2026-08-16:

| `payment_attribution_trust` | `currency` | `payment_rows / orders` | `gross_paid_amount` |
|---|---|---:|---:|
| `trusted` | `RUB` | 216 total trusted payments across currencies | 2797071.26 |
| `trusted` | `KZT` | included in 216 trusted payments | 625000.00 |
| `trusted` | `BYN` | included in 216 trusted payments | 2300.63 |
| `untrusted` | `RUB` | 188 total untrusted payments across currencies | 10023957.00 |
| `untrusted` | `KZT` | included in 188 untrusted payments | 12293284.43 |

Итоги качества приемки:

- `trusted`: 216 оплат;
- `untrusted`: 188 оплат;
- `missing_purchase_paid` у trusted: 1;
- `amount_mismatch`: 0;
- `currency_mismatch`: 0.
