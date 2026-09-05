# Dataset: order_payment_net_v1

---
type: dataset
name: order_payment_net_v1
superset_dataset_id: 39
superset_type: physical_view
database: ADB
schema: adb
physical_table: adb.order_payment_net_v1
depends_on:
  - adb.order_payments_v2
  - adb.payment_refund_events_v1
used_by_charts: []
---

## Краткое описание

Payment-level net dataset для payment-v2 + refund/cancel v1. Это основная финансовая витрина: одна строка показывает оплату, ее verified gross received amount, подтвержденные возвраты и итоговый net.

Production acceptance: refund/net v1 принят заказчиком по независимой проверке production Superset MCP от 2026-09-04. Проверено: `686` строк = `686` уникальных `order_id + payment_id`.

Grain:

```text
1 строка = 1 order_id + payment_id
```

## Финансовая формула

Считается отдельно по `currency`:

```text
net_paid_amount = verified_gross_received_amount - confirmed_refund_amount
```

Где:

- `verified_gross_received_amount` = `amount_received` из payment-v2 только для `amount_quality IN ('exact','partial')`;
- `confirmed_refund_amount` = сумма `refund_amount` только для `order_refund_v1` + `financial_state='accepted'`.

Cancel events не уменьшают revenue.

Если у исходной оплаты `amount_quality='unknown'` или `base_currency`, то `verified_gross_received_amount` остается `NULL`. Accepted refund по такой оплате сохраняется в net-слое, но помечается `refund_against_unverified_gross_flag=1` и `net_quality='refund_against_unverified_gross'`. Это диагностический случай: refund есть, но verified gross received для корректного net еще не подтвержден.

Рекомендуемый табличный chart:

```text
Net paid по оплатам payment v1
```

## Что отражает view

Эта view отвечает на вопрос: "Какая итоговая net-сумма осталась по каждой оплате после accepted refunds?"

Использовать для:

- общей gross/refund/net выручки по оплатам;
- проверки `net_paid_amount` на уровне `order_id + payment_id`;
- контроля refund rate по валютам;
- диагностики отрицательного net, over-refund, currency mismatch и cancel без refund.

Это основной dataset для net paid на уровне оплаты. Денежные итоги всегда группировать по `currency`.

## Поля

| Поле | Label в Superset                  | Описание |
|---|-----------------------------------|---|
| `order_id` | ID заказа                         | ID заказа MID |
| `payment_id` | ID платежа                        | ID платежа |
| `canonical_event_id` | ID канонической оплаты            | ID `order_paid_v2`, создавшего payment fact |
| `paid_at` | Время оплаты                      | Время исходной оплаты |
| `paid_date` | Дата оплаты                       | `DATE(paid_at)` |
| `currency` | Валюта                            | Валюта расчета, без FX |
| `order_amount` | Сумма заказа                      | `amount` исходной оплаты |
| `gross_paid_amount` | Legacy gross/order amount         | Сумма исходной оплаты для совместимости |
| `amount_received` | Полученная сумма                  | Source `amount_received` исходной оплаты |
| `amount_quality` | Качество суммы оплаты             | `exact`, `partial`, `unknown`, `base_currency` |
| `verified_gross_received_amount` | Verified gross received           | Подтвержденная полученная сумма оплаты |
| `confirmed_refund_amount` | Подтвержденный возврат            | Accepted refund amount |
| `net_paid_amount` | Net paid                          | Gross received минус accepted refunds |
| `accepted_refund_events` | Accepted refund events            | Количество финансовых refund events |
| `accepted_refund_ids` | Accepted refund IDs               | Количество уникальных refund_id |
| `last_refunded_at` | Последний возврат                 | Время последнего accepted refund |
| `cancel_events` | Cancel events                     | Количество cancel events |
| `last_cancelled_at` | Последняя отмена                  | Время последнего cancel |
| `payment_attribution_trust` | Доверие к ClientID                | Trust исходной оплаты |
| `campaign_attribution_quality` | Качество campaign attribution     | Bucket исходной оплаты |
| `direct_campaign_id` | CampaignId Директа                | CampaignId исходной оплаты |
| `attribution_platform` | Площадка атрибуции                | Platform/bucket исходной оплаты |
| `is_proven_campaign_attribution` | Campaign attribution доказана     | Флаг exact campaign attribution |
| `is_proven_received_revenue` | Received revenue доказана         | Exact campaign + `amount_quality IN ('exact','partial')` |
| `direct_spend_missing_flag` | Нет расхода Direct                | CampaignId с неизвестным spend |
| `is_campaign_roas_eligible` | Можно использовать для ROAS       | Exact/partial RUB campaign с известным spend |
| `source_payment_found` | Исходная оплата найдена           | Флаг связи refund с payment-v2 |
| `duplicate_refund_id_flag` | Дубль refund key                  | Quality-флаг |
| `orphan_refund_flag` | Orphan refund (Зависший возврат)  | Quality-флаг |
| `refund_currency_mismatch_flag` | Несовпадение валюты               | Quality-флаг |
| `refund_amount_exceeds_gross_flag` | Refund больше gross               | Quality-флаг |
| `refund_against_unverified_gross_flag` | Refund против неподтвержденного gross | Accepted refund при `amount_quality=unknown/base_currency` у исходной оплаты |
| `negative_net_paid_flag` | Отрицательный net                 | Quality-флаг |
| `cancelled_after_paid_without_refund_flag` | Cancel без refund (Отмена без возврата) | Информационный lifecycle-флаг |
| `refund_without_campaign_source_payment_flag` | Refund без campaign source payment | Входит в общий net, не атрибутируется на кампанию |
| `net_quality` | Качество net                      | `no_refund`, `verified_net`, `refund_against_unverified_gross`, `missing_source_payment`, `negative_net` |
| `grain` | Grain                             | `1 order_id + payment_id` |
| `revenue_semantics` | Семантика выручки                 | `net = verified gross received - accepted refunds, by currency` |
| `dataset_version` | Версия dataset                    | `order_payment_net_v1` |

## Правила использования

- Денежные итоги группировать по `currency`.
- `net_paid_amount` можно использовать только после проверки quality flags.
- Для строгих финансовых выводов отдельно фильтровать или показывать `net_quality='refund_against_unverified_gross'`: в этих строках refund подтвержден, но gross received не подтвержден.
- `cancelled_after_paid_without_refund_flag` не является ошибкой и не уменьшает revenue.

## Read-back SQL

```sql
SELECT
  currency,
  COUNT(*) AS payment_rows,
  COUNT(DISTINCT order_id) AS orders,
  ROUND(SUM(verified_gross_received_amount), 2) AS verified_gross_received_amount,
  ROUND(SUM(confirmed_refund_amount), 2) AS confirmed_refund_amount,
  ROUND(SUM(net_paid_amount), 2) AS net_paid_amount,
  SUM(accepted_refund_events) AS accepted_refund_events,
  SUM(negative_net_paid_flag) AS negative_net_paid_rows,
  SUM(refund_against_unverified_gross_flag) AS refund_against_unverified_gross_rows
FROM adb.order_payment_net_v1
GROUP BY currency
ORDER BY currency;
```
