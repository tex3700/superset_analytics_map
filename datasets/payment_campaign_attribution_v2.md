# Dataset: payment_campaign_attribution_v2

---
type: dataset
name: payment_campaign_attribution_v2
superset_dataset_id: 36
superset_type: physical_view
database: ADB
database_id: 4
schema: adb
physical_table: adb.payment_campaign_attribution_v2
depends_on:
  - adb.order_payments_v2
used_by_charts:
  - atributsiya-oplat-po-reklamnym-kampaniyam
---

## Краткое описание

Payment-level dataset для рекламной attribution после payment-v2.

Статус на 2026-08-14: ADB view создана, Superset dataset создан с ID `36`.

Главный grain:

```text
1 строка = 1 order_id + payment_id + attribution decision
```

Dataset разделяет два качества:

| Поле | Что означает |
|---|---|
| `payment_attribution_trust` | Качество ClientID из backend outbox |
| `campaign_attribution_quality` | Качество связи оплаты с рекламной кампанией |

## Что отображает

Dataset показывает payment-level оплату вместе с решением о качестве рекламной campaign attribution.

Он отвечает на вопросы:

- какая часть gross paid может быть использована в доказанных campaign CPA/ROAS/CAC;
- какая часть trusted, но не имеет точной связи с кампанией;
- какая часть untrusted и должна показываться отдельно;
- какие платежи имеют exact-связку `yclid + numeric CampaignId`.

Источник SQL для Superset: ADB view `adb.payment_campaign_attribution_v2`.

## Валюты и campaign KPI

Денежные суммы нельзя складывать между разными валютами без документированного FX-курса.

Campaign CPA/ROAS/CAC требует отдельного join с расходом Direct на одинаковом:

- периоде;
- валюте;
- VAT-базе;
- attribution model;
- зрелом окне данных.

На 2026-08-16 этот spend join еще не входит в `payment_campaign_attribution_v2`; dataset принят для gross paid и классификации attribution, но не является готовым CPA/ROAS dataset.

## Campaign attribution quality

Минимальные значения:

| Значение | Смысл | Входит в доказанный campaign ROAS/CPA/CAC |
|---|---|---|
| `exact_campaign_id_yclid` | Есть trusted ClientID, `yclid`, `utm_source=yandex/ya`, `utm_medium=cpc`, numeric `utm_campaign` как CampaignId | Да |
| `utm_fallback` | Есть UTM, но нет exact `yclid + CampaignId` | Нет |
| `unattributed` | Нет достаточной связи с кампанией | Нет |
| `untrusted_client_id` | Оплата имеет `attribution_trust=untrusted` | Нет |
| `conflict` | Зарезервировано для будущих конфликтов attribution | Нет |

## Финансовые поля

| Поле | Описание |
|---|---|
| `gross_paid_amount` | Общий gross paid факт оплаты |
| `proven_campaign_gross_paid_amount` | Gross paid только для доказанной campaign attribution |
| `untrusted_gross_paid_amount` | Gross paid с загрязненным ClientID |
| `trusted_unattributed_gross_paid_amount` | Trusted payment без точной campaign-связи |

## Поля и русские labels

Dataset наследует все поля [order_payments_v2](./order_payments_v2.md) и добавляет поля campaign attribution:

| Поле | Label в Superset | Источник / формула | Описание |
|---|---|---|---|
| `campaign_attribution_quality` | Качество атрибуции кампании | CASE по `payment_attribution_trust`, `client_id`, `yclid`, `utm_source`, `utm_medium`, `utm_campaign` | Качество связи оплаты с кампанией |
| `is_proven_campaign_attribution` | Доказанная campaign attribution | `1` только для trusted + `yclid` + Direct CPC + numeric CampaignId | Флаг включения в доказанные campaign KPI |
| `direct_campaign_id` | CampaignId Директа | `utm_campaign`, только для exact Direct attribution | ID кампании Яндекс.Директа |
| `attribution_platform` | Площадка атрибуции | CASE по exact/untrusted/unattributed/source | Площадка или bucket attribution |
| `proven_campaign_gross_paid_amount` | Доказанная gross paid по кампании | `gross_paid_amount`, только если `is_proven_campaign_attribution=1` | Сумма для доказанного campaign ROAS/CPA/CAC |
| `untrusted_gross_paid_amount` | Untrusted gross paid | `gross_paid_amount`, если `payment_attribution_trust='untrusted'` | Реальная оплата, но без доверенной campaign attribution |
| `trusted_unattributed_gross_paid_amount` | Trusted, но без кампании | `gross_paid_amount`, если trusted, но не exact campaign | Trusted оплата без доказанной связи с кампанией |
| `dataset_version` | Версия dataset | constant | `payment_campaign_attribution_v2` |
| `attribution_model` | Модель атрибуции | constant | Правило exact: `yclid + numeric CampaignId` |

## Важные правила

- `trusted` без exact campaign-связи остается `unattributed`.
- `untrusted` сохраняется в общем финансовом факте, но не распределяется по кампаниям.
- До появления refund/cancel источника в dataset нет `net_paid`.

## Read-back SQL

```sql
SELECT
  campaign_attribution_quality,
  currency,
  COUNT(*) AS payment_rows,
  COUNT(DISTINCT order_id) AS orders,
  ROUND(SUM(gross_paid_amount), 2) AS gross_paid_amount,
  ROUND(SUM(COALESCE(proven_campaign_gross_paid_amount, 0)), 2) AS proven_campaign_gross_paid_amount,
  ROUND(SUM(COALESCE(untrusted_gross_paid_amount, 0)), 2) AS untrusted_gross_paid_amount,
  ROUND(SUM(COALESCE(trusted_unattributed_gross_paid_amount, 0)), 2) AS trusted_unattributed_gross_paid_amount
FROM adb.payment_campaign_attribution_v2
GROUP BY campaign_attribution_quality, currency
ORDER BY campaign_attribution_quality, currency;
```

Read-back независимой production-приемки на 2026-08-16:

| `campaign_attribution_quality` | `currency` | `payment_rows / orders` | `gross_paid_amount` | `proven_campaign_gross_paid_amount` |
|---|---|---:|---:|---:|
| `exact_campaign_id_yclid` | `RUB` | 23 / 23 | 329475.00 | 329475.00 |
| `utm_fallback` | `RUB` | 8 / 8 | 99056.00 | 0.00 |
| `untrusted_client_id` | `RUB` | part of 188 untrusted payments | 10023957.00 | 0.00 |
| `untrusted_client_id` | `KZT` | part of 188 untrusted payments | 12293284.43 | 0.00 |

Дополнительные итоги приемки:

- нарушений `untrusted -> proven`: 0;
- `utm_fallback`: 8 оплат / 99056.00 RUB, не proven;
- proven campaign gross paid есть только в `exact_campaign_id_yclid`;
- `unattributed`, `utm_fallback` и `untrusted_client_id` не входят в доказанные campaign CPA/ROAS/CAC.
