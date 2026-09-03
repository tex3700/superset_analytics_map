# Dataset: payment_campaign_net_attribution_v1

---
type: dataset
name: payment_campaign_net_attribution_v1
superset_dataset_id: TODO
superset_type: physical_view
database: ADB
schema: adb
physical_table: adb.payment_campaign_net_attribution_v1
depends_on:
  - adb.order_payment_net_v1
used_by_charts: []
---

## Краткое описание

Campaign-level net attribution dataset для payment-v2 + refund/cancel v1. Это слой для проверки, как confirmed refunds уменьшают net внутри того же attribution bucket, что и исходная оплата.

Grain:

```text
1 строка = 1 order_id + payment_id + net attribution decision
```

Refund не атрибутируется заново. Он наследует bucket исходной оплаты.

Рекомендуемый табличный chart:

```text
Net attribution оплат по кампаниям
```

## Что отражает view

Эта view отвечает на вопрос: "Как accepted refunds уменьшают gross/received revenue внутри campaign attribution bucket исходной оплаты?"

Использовать для:

- proven campaign gross/refund/net;
- проверки, что refund exact campaign payment уменьшает ту же `direct_campaign_id`;
- проверки, что `untrusted`, `utm_fallback` и `unattributed` не попадают в proven campaign KPI;
- будущего ROAS после join с расходом через `roas_eligible_net_paid_amount`.

Это не spend-join dataset: расход Direct здесь еще не присоединен.

## Основное правило

- Exact campaign payment refund уменьшает net той же `direct_campaign_id`.
- `utm_fallback`, `unattributed`, `untrusted_client_id` остаются в своем bucket.
- `refund_without_campaign_source_payment` входит в общий net, но уходит в `unattributed`, не в campaign net.
- Если refund пришел по оплате с `amount_quality=unknown/base_currency`, строка сохраняет bucket исходной оплаты, но остается диагностической через `refund_against_unverified_gross_flag` / `net_quality`.

## Финансовые поля

Dataset наследует все поля [order_payment_net_v1](./order_payment_net_v1.md) и добавляет net attribution поля ниже.

| Поле | Label в Superset | Описание |
|---|---|---|
| `net_campaign_attribution_quality` | Net attribution quality | Финальный bucket для net |
| `net_direct_campaign_id` | Net CampaignId | CampaignId для net; NULL для unattributed/unmatched |
| `net_attribution_platform` | Net attribution platform | Platform/bucket для net |
| `proven_campaign_verified_gross_received_amount` | Proven campaign gross received | Gross для proven campaign |
| `proven_campaign_confirmed_refund_amount` | Proven campaign refund | Refund, уменьшающий proven campaign |
| `proven_campaign_net_paid_amount` | Proven campaign net paid | Net для proven campaign |
| `roas_eligible_net_paid_amount` | ROAS-eligible net paid | Net для будущего ROAS после spend join |
| `non_proven_confirmed_refund_amount` | Non-proven refund | Refund вне proven campaign |
| `net_dataset_version` | Версия net dataset | `payment_campaign_net_attribution_v1` |
| `net_attribution_model` | Модель net attribution | Refund наследует bucket исходной оплаты; unmatched refund становится `unattributed` |

## Read-back SQL

```sql
SELECT
  net_campaign_attribution_quality,
  currency,
  COUNT(*) AS payment_rows,
  ROUND(SUM(verified_gross_received_amount), 2) AS verified_gross_received_amount,
  ROUND(SUM(confirmed_refund_amount), 2) AS confirmed_refund_amount,
  ROUND(SUM(net_paid_amount), 2) AS net_paid_amount,
  ROUND(SUM(COALESCE(proven_campaign_net_paid_amount, 0)), 2) AS proven_campaign_net_paid_amount,
  ROUND(SUM(COALESCE(roas_eligible_net_paid_amount, 0)), 2) AS roas_eligible_net_paid_amount,
  SUM(refund_against_unverified_gross_flag) AS refund_against_unverified_gross_rows
FROM adb.payment_campaign_net_attribution_v1
GROUP BY net_campaign_attribution_quality, currency
ORDER BY net_campaign_attribution_quality, currency;
```
