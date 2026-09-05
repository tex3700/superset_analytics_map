# Dataset: payment_refund_events_v1

---
type: dataset
name: payment_refund_events_v1
superset_dataset_id: 37
superset_type: physical_view
database: ADB
schema: adb
physical_table: adb.payment_refund_events_v1
depends_on:
  - adb.mid_analytics_refund_event
  - adb.payment_campaign_attribution_v2
used_by_charts: []
---

## Краткое описание

Event-level dataset для refund/cancel v1. Это самый подробный диагностический слой: он показывает каждое событие возврата или отмены, поля source table `analytics_refund_event`, найденную исходную оплату и quality-флаги.

Production acceptance: refund/net v1 принят заказчиком по независимой проверке production Superset MCP от 2026-09-04.

Grain:

```text
1 строка = 1 order_id + payment_id + refund_id
```

Dataset показывает refund/cancel events и наследуемую attribution исходной оплаты. Возврат не атрибутируется заново по ClientID/UTM.

Рекомендуемый табличный chart:

```text
События возвратов и отмен payment v1
```

## Что отражает view

Эта view отвечает на вопрос: "Какие refund/cancel события пришли из backend source и как они связаны с исходной оплатой?"

Использовать для:

- проверки source rows `analytics_refund_event`;
- просмотра cancel events отдельно от financial refunds;
- диагностики `orphan_refund`, `currency_mismatch`, `refund_amount_exceeds_gross`;
- диагностики accepted refund против оплаты с неподтвержденной received-суммой;
- проверки, от какой оплаты и какого attribution bucket наследуется refund.

Не использовать как итоговую net-выручку: здесь одна оплата может иметь несколько refund/cancel events.

## Финансовая семантика

В net входит только:

```sql
event_name = 'order_refund_v1'
AND financial_state = 'accepted'
```

`order_cancelled_v1`, `rejected` и `non_financial` не уменьшают revenue.

## Поля и labels

| Поле | Label в Superset                              | Описание |
|---|-----------------------------------------------|---|
| `event_id` | ID события возврата                           | Уникальное refund/cancel event |
| `event_name` | Тип события                                   | `order_refund_v1` / `order_cancelled_v1` |
| `order_id` | ID заказа                                     | ID заказа MID |
| `payment_id` | ID платежа                                    | ID исходной оплаты |
| `refund_id` | ID возврата                                   | Стабильный ключ возврата/отмены |
| `paid_event_id` | ID payment event                              | ID исходного payment event из refund source |
| `refunded_at` | Время возврата/отмены                         | Время события |
| `refunded_date` | Дата возврата/отмены                          | `DATE(refunded_at)` |
| `refund_amount` | Сумма события возврата                        | Сумма из source event |
| `refund_source_gross_paid_amount` | Gross оплаты из refund source (Валовая оплата из источника возврата) | Source `gross_paid_amount` для диагностики |
| `refund_currency` | Валюта возврата                               | Валюта без FX |
| `refund_type` | Тип возврата                                  | `full` / `partial` |
| `financial_state` | Финансовое состояние                          | `accepted`, `rejected`, `non_financial` |
| `refund_source` | Источник refund/cancel                        | Source event |
| `reason_code` | Код причины                                   | Причина без персональных данных |
| `operator_id` | ID оператора                                  | Оператор, зафиксировавший событие |
| `operator_username` | Оператор                                      | Username оператора |
| `order_status_id` | Статус заказа                                 | Статус заказа на момент события |
| `paid_before_cancel` | Оплачен до отмены                             | Флаг оплаты до cancel |
| `refund_source_attribution_trust` | Trust из refund source                        | Trust исходной оплаты для трассировки |
| `quality_flags` | Quality flags refund source                   | Quality-флаги backend |
| `payload_hash` | Hash payload                                  | Hash события |
| `refund_status` | Статус refund event                           | Технический статус source event |
| `attempts` | Попытки отправки                              | Обычно `0`, refund/cancel не отправляется в Метрику |
| `last_http_code` | Последний HTTP-код                            | Обычно `0` |
| `last_error_code` | Последняя ошибка                              | Код ошибки, если был |
| `refund_created_at` | Refund event создан  (Создано событие возврата) | Source `created_at` |
| `refund_updated_at` | Обновлено (Обновлено событие возврата)        | Source `updated_at` |
| `refund_sent_at` | Refund event отправлен (Возврат отправлен)    | Source `sent_at`; обычно `NULL` |
| `adb_synced_at` | Синхронизировано в ADB                        | Время переноса в ADB |
| `payment_canonical_event_id` | ID канонической оплаты                        | `canonical_event_id` исходной оплаты |
| `paid_at` | Время оплаты                                  | Время исходной оплаты |
| `paid_date` | Дата оплаты                                   | Дата исходной оплаты |
| `payment_currency` | Валюта оплаты                                 | Валюта исходной оплаты |
| `order_amount` | Сумма заказа                                  | `amount` исходной оплаты |
| `gross_paid_amount` | Legacy gross/order amount                     | Сумма исходной оплаты для совместимости |
| `amount_received` | Полученная сумма                              | `amount_received` исходной оплаты |
| `amount_quality` | Качество суммы оплаты                         | `exact`, `partial`, `unknown`, `base_currency` |
| `verified_received_amount` | Подтвержденная полученная сумма               | Received amount исходной оплаты только для `exact/partial` |
| `payment_attribution_trust` | Trust исходной оплаты                         | Trust из payment-v2 |
| `source_campaign_attribution_quality` | Campaign quality исходной оплаты              | Bucket исходной оплаты |
| `source_direct_campaign_id` | CampaignId исходной оплаты                    | CampaignId исходной оплаты |
| `source_attribution_platform` | Площадка исходной оплаты                      | Platform/bucket исходной оплаты |
| `source_is_proven_campaign_attribution` | Исходная campaign attribution доказана        | Флаг exact campaign attribution исходной оплаты |
| `source_is_campaign_roas_eligible` | Исходная оплата пригодна для ROAS             | Флаг ROAS eligibility исходной оплаты |
| `source_payment_found` | Исходная оплата найдена                       | Есть связь с `payment_campaign_attribution_v2` |
| `confirmed_refund_amount` | Подтвержденная сумма возврата                 | `refund_amount`, только для accepted refund |
| `is_accepted_refund` | Accepted refund                               | Флаг попадания в net formula |
| `is_cancel_event` | Cancel event                                  | Флаг отмены без финансового уменьшения |
| `inherited_campaign_attribution_quality` | Наследуемое качество кампании                 | Bucket исходной оплаты или `unattributed` |
| `inherited_direct_campaign_id` | Наследуемый CampaignId                        | CampaignId исходной оплаты |
| `inherited_attribution_platform` | Наследуемая площадка                          | Platform/bucket исходной оплаты или `Unattributed` |
| `duplicate_refund_id_flag` | Дубль refund key                              | Повтор business key |
| `orphan_refund_flag` | Orphan refund                                 | Refund без исходной оплаты или backend flag |
| `refund_currency_mismatch_flag` | Несовпадение валюты                           | Валюта refund отличается от оплаты |
| `refund_amount_exceeds_gross_flag` | Refund больше gross                           | Refund превышает verified gross received |
| `refund_against_unverified_gross_flag` | Refund против неподтвержденного gross         | Accepted refund есть, но у оплаты нет `verified_received_amount` из-за `amount_quality=unknown/base_currency` |
| `grain` | Grain                                         | `1 order_id + payment_id + refund_id` |
| `attribution_model` | Модель атрибуции refund                       | Refund наследует attribution bucket исходной оплаты |
| `dataset_version` | Версия dataset                                | `payment_refund_events_v1` |

## Read-back SQL

```sql
SELECT
  event_name,
  financial_state,
  refund_currency,
  COUNT(*) AS event_rows,
  COUNT(DISTINCT event_name, order_id, payment_id, refund_id) AS refund_keys,
  ROUND(SUM(refund_amount), 2) AS refund_amount,
  ROUND(SUM(confirmed_refund_amount), 2) AS confirmed_refund_amount
FROM adb.payment_refund_events_v1
GROUP BY event_name, financial_state, refund_currency
ORDER BY event_name, financial_state, refund_currency;
```
