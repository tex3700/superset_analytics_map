# Chart: Атрибуция оплат по рекламным кампаниям

---
type: chart
name: Атрибуция оплат по рекламным кампаниям
superset_chart_id: 122
dataset: payment_campaign_attribution_v2
dashboards:
  - TODO
---

## Назначение

Raw table chart для проверки campaign attribution поверх payment-v2 оплат.

Чарт показывает, какие оплаты можно включать в доказанные campaign CPA/ROAS/CAC, а какие нужно оставить как `utm_fallback`, `unattributed` или `untrusted_client_id`.

## Superset

- Chart name: `Атрибуция оплат по рекламным кампаниям`
- Chart ID: `122`
- Visualization type: table
- Dataset: [payment_campaign_attribution_v2](../datasets/payment_campaign_attribution_v2.md), id `36`
- Dashboard: TODO

## Используемые поля

Чарт выводит все поля dataset [payment_campaign_attribution_v2](../datasets/payment_campaign_attribution_v2.md).

## Агрегация в чарте

Исходный grain датасета:

```text
1 строка = 1 order_id + payment_id + attribution decision
```

Grain после агрегации в чарте:

```text
не агрегируется
```

## Особенности интерпретации

- В доказанные campaign KPI входит только `campaign_attribution_quality = exact_campaign_id_yclid`.
- Денежные суммы нужно смотреть в разрезе `currency`; RUB/KZT/BYN нельзя складывать в один total без FX-курса.
- `untrusted_client_id` сохраняется в общем финансовом факте, но не распределяется по кампаниям.
- `utm_fallback` и `unattributed` показываются отдельно и не считаются доказанной campaign attribution.
