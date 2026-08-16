# Chart: События payment outbox и сверка

---
type: chart
name: События payment outbox и сверка
superset_chart_id: 120
dataset: payment_events_v2
dashboards:
  - TODO
---

## Назначение

Raw table chart для диагностики payment-v2 событий на уровне `event_id`.

Чарт показывает строки outbox и связанные reconciliation-поля: тип события, заказ, платеж, сумму, валюту, ClientID, `attribution_trust`, статус доставки, read-back и `purchase_seen`.

## Superset

- Chart name: `События payment outbox и сверка`
- Chart ID: `120`
- Visualization type: table
- Dataset: [payment_events_v2](../datasets/payment_events_v2.md), id `34`
- Dashboard: TODO

## Используемые поля

Чарт выводит все поля dataset [payment_events_v2](../datasets/payment_events_v2.md).

## Агрегация в чарте

Исходный grain датасета:

```text
1 строка = 1 event_id
```

Grain после агрегации в чарте:

```text
не агрегируется
```

## Особенности интерпретации

- Не использовать сумму `amount` из этого chart как финансовую выручку: одна оплата может иметь несколько event rows.
- Денежные суммы нужно смотреть в разрезе `currency`; RUB/KZT/BYN нельзя складывать в один total без FX-курса.
- Для финансового факта использовать [Канонические оплаты payment v2](./kanonicheskie-oplaty-payment-v2.md).
