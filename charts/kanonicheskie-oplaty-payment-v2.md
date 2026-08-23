# Chart: Канонические оплаты payment v2

---
type: chart
name: Канонические оплаты payment v2
superset_chart_id: 121
dataset: order_payments_v2
dashboards:
  - TODO
---

## Назначение

Raw table chart для просмотра канонического финансового факта payment v2.

Одна строка соответствует одной оплате на grain `order_id + payment_id`, созданной первым `operator_verified order_paid_v2`.

## Superset

- Chart name: `Канонические оплаты payment v2`
- Chart ID: `121`
- Visualization type: table
- Dataset: [order_payments_v2](../datasets/order_payments_v2.md), id `35`
- Dashboard: TODO

## Используемые поля

Чарт выводит все поля dataset [order_payments_v2](../datasets/order_payments_v2.md).

## Агрегация в чарте

Исходный grain датасета:

```text
1 строка = 1 order_id + payment_id
```

Grain после агрегации в чарте:

```text
не агрегируется
```

## Особенности интерпретации

- `order_amount` / `gross_paid_amount` показывает сумму заказа/события.
- `verified_received_amount` показывает фактически полученную сумму только для `amount_quality IN ('exact','partial')`.
- `amount_quality IN ('unknown','base_currency')` показывать отдельно и не использовать как доказанную received revenue.
- Денежные суммы нужно смотреть в разрезе `currency`; RUB/KZT/BYN нельзя складывать в один total без FX-курса.
- `net_paid` пока не заявляется, потому что нет отдельного refund/cancel контура.
- `payment_attribution_trust = trusted` означает качество ClientID, но не доказывает связь с рекламной кампанией.
