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

- какая часть оплат имеет exact-связь с CampaignId;
- какая часть денег может быть использована как доказанная received revenue;
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

На 2026-08-22 этот spend join еще не входит в `payment_campaign_attribution_v2`; dataset принят для payment attribution и received revenue classification, но не является готовым CPA/ROAS dataset.

CampaignId `114341332` нельзя использовать для CPA/ROAS, потому что на 2026-08-21 он не наблюдается ни в одном из двух подключенных Direct-аккаунтов, и расход по нему неизвестен.

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
| `proven_campaign_verified_received_amount` | Фактически полученная сумма для exact campaign attribution, только `amount_quality IN ('exact','partial')` |
| `roas_eligible_verified_received_amount` | Полученная сумма, пригодная для future ROAS после spend join: exact/partial, RUB, не CampaignId `114341332` |
| `proven_campaign_unverified_order_amount` | Exact campaign attribution есть, но сумма не доказана как received из-за `unknown/base_currency` |
| `untrusted_gross_paid_amount` | Gross paid с загрязненным ClientID |
| `untrusted_verified_received_amount` | Фактически полученная сумма в untrusted bucket, не для proven campaign KPI |
| `trusted_unattributed_gross_paid_amount` | Trusted payment без точной campaign-связи |
| `trusted_unattributed_verified_received_amount` | Фактически полученная сумма trusted/unattributed bucket, не для proven campaign KPI |

## Поля и русские labels

Dataset наследует все поля [order_payments_v2](./order_payments_v2.md) и добавляет поля campaign attribution:

| Поле | Label в Superset | Источник / формула | Описание |
|---|---|---|---|
| `campaign_attribution_quality` | Качество атрибуции кампании | CASE по `payment_attribution_trust`, `client_id`, `yclid`, `utm_source`, `utm_medium`, `utm_campaign` | Качество связи оплаты с кампанией |
| `is_proven_campaign_attribution` | Доказанная campaign attribution | `1` только для trusted + `yclid` + Direct CPC + numeric CampaignId | Флаг включения в доказанные campaign KPI |
| `direct_campaign_id` | CampaignId Директа | `utm_campaign`, только для exact Direct attribution | ID кампании Яндекс.Директа |
| `is_proven_received_revenue` | Доказанная received revenue | exact campaign attribution + `amount_quality IN ('exact','partial')` | Флаг денежной суммы, которую можно использовать как доказанную revenue |
| `direct_spend_missing_flag` | Нет расхода Директа | `utm_campaign='114341332'` | CampaignId найден в оплате, но расход в подключенных Direct-аккаунтах не наблюдается |
| `is_campaign_roas_eligible` | Можно использовать для ROAS | exact campaign + `amount_quality IN ('exact','partial')` + `currency='RUB'` + CampaignId не `114341332` | Флаг пригодности для будущего ROAS после join с расходом |
| `attribution_platform` | Площадка атрибуции | CASE по exact/untrusted/unattributed/source | Площадка или bucket attribution |
| `proven_campaign_gross_paid_amount` | Доказанная gross paid по кампании | `gross_paid_amount`, только если `is_proven_campaign_attribution=1` | Сумма для доказанного campaign ROAS/CPA/CAC |
| `proven_campaign_verified_received_amount` | Доказанная полученная сумма по кампании | `verified_received_amount`, только если exact campaign + `amount_quality IN ('exact','partial')` | Основная revenue-мера для future CPA/ROAS |
| `roas_eligible_verified_received_amount` | Полученная сумма для ROAS | `verified_received_amount`, только если `is_campaign_roas_eligible=1` | Revenue-мера, из которой исключен CampaignId без spend |
| `proven_campaign_unverified_order_amount` | Недоказанная сумма заказа exact campaign | `gross_paid_amount`, если exact campaign + `amount_quality IN ('unknown','base_currency')` | Показывать отдельно, не включать в proven revenue/ROAS |
| `untrusted_gross_paid_amount` | Untrusted gross paid | `gross_paid_amount`, если `payment_attribution_trust='untrusted'` | Реальная оплата, но без доверенной campaign attribution |
| `untrusted_verified_received_amount` | Untrusted received amount | `verified_received_amount`, если `payment_attribution_trust='untrusted'` | Полученная сумма, но campaign attribution недоверенная |
| `trusted_unattributed_gross_paid_amount` | Trusted, но без кампании | `gross_paid_amount`, если trusted, но не exact campaign | Trusted оплата без доказанной связи с кампанией |
| `trusted_unattributed_verified_received_amount` | Trusted/unattributed received amount | `verified_received_amount`, если trusted, но не exact campaign | Полученная сумма без доказанной связи с кампанией |
| `dataset_version` | Версия dataset | constant | `payment_campaign_attribution_v2` |
| `attribution_model` | Модель атрибуции | constant | Правило exact: `yclid + numeric CampaignId` |

## Важные правила

- `trusted` без exact campaign-связи остается `unattributed`.
- `untrusted` сохраняется в общем финансовом факте, но не распределяется по кампаниям.
- Доказанная денежная campaign revenue считается по `proven_campaign_verified_received_amount`, а не по `proven_campaign_gross_paid_amount`.
- `amount_quality IN ('unknown','base_currency')` показывается отдельно через `proven_campaign_unverified_order_amount`.
- Для CampaignId `114341332` нельзя считать CPA/ROAS без подключения owning Direct account/spend.
- Для future CPA/ROAS использовать `roas_eligible_verified_received_amount`, а не все `proven_campaign_verified_received_amount`.
- До появления refund/cancel источника в dataset нет `net_paid`.

## Read-back SQL

```sql
SELECT
  campaign_attribution_quality,
  currency,
  COUNT(*) AS payment_rows,
  COUNT(DISTINCT order_id) AS orders,
  amount_quality,
  direct_campaign_id,
  ROUND(SUM(gross_paid_amount), 2) AS gross_paid_amount,
  ROUND(SUM(COALESCE(proven_campaign_gross_paid_amount, 0)), 2) AS proven_campaign_gross_paid_amount,
  ROUND(SUM(COALESCE(proven_campaign_verified_received_amount, 0)), 2) AS proven_campaign_verified_received_amount,
  ROUND(SUM(COALESCE(roas_eligible_verified_received_amount, 0)), 2) AS roas_eligible_verified_received_amount,
  ROUND(SUM(COALESCE(proven_campaign_unverified_order_amount, 0)), 2) AS proven_campaign_unverified_order_amount,
  ROUND(SUM(COALESCE(untrusted_gross_paid_amount, 0)), 2) AS untrusted_gross_paid_amount,
  ROUND(SUM(COALESCE(trusted_unattributed_gross_paid_amount, 0)), 2) AS trusted_unattributed_gross_paid_amount
FROM adb.payment_campaign_attribution_v2
GROUP BY campaign_attribution_quality, currency, amount_quality, direct_campaign_id
ORDER BY campaign_attribution_quality, currency, amount_quality, direct_campaign_id;
```

Read-back чистого окна 2026-08-13..2026-08-19 по exact CampaignId:

| `direct_campaign_id` | payments | `amount_quality` | `verified_received_amount` | Примечание |
|---|---:|---|---:|---|
| `709990033` | 2 | `exact` | 26478.00 | Можно использовать как received revenue после spend join |
| `709990033` | 1 | `unknown` | 0.00 | `order_amount=27529`, не proven revenue |
| `709998427` | 1 | `exact` | 6962.00 | Можно использовать как received revenue после spend join |
| `114341332` | 1 | `exact` | 2395.00 | CPA/ROAS не считать: spend/account не подключен |

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
