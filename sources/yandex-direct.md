# Source: Yandex Direct via Yandex Metrika API

---
type: source
name: yandex_direct_via_yandex_metrika_api
system: Yandex
service: Yandex Direct / Yandex Metrika
used_by_ingestion:
  - advertisings_stats
---

## Назначение

Источник рекламной статистики Яндекс.Директа, получаемой через API Яндекс.Метрики.

Важно: текущий workflow использует не отдельный API Яндекс.Директа, а endpoint Яндекс.Метрики `stat/v1/data` с рекламными измерениями и метриками `ym:ad:*`.

## API

- Endpoint: `https://api-metrika.yandex.net/stat/v1/data`
- Counter ID: `19444381`
- Direct client login: `e-17323892`
- Основное измерение рекламной кампании: `ym:ad:directOrder`
- Измерение даты: `ym:ad:date`

## Метрики

| API metric | Значение в ADB |
|---|---|
| `ym:ad:RUBConvertedAdCost` | Расход на рекламу в рублях |
| `ym:ad:visits` | Визиты |
| `ym:ad:users` | Посетители |
| `ym:ad:clicks` | Клики |
| `ym:ad:goal494050704reaches` | Достижения цели "Заказ оплачен" |
| `ym:ad:goal464707612reaches` | Достижения цели "Кнопка купить" |
| `ym:ad:goal464710001reaches` | Достижения цели "Оформить заказ" |
| `ym:ad:goal464710042reaches` | Достижения цели оформления после быстрого заказа |
| `ym:ad:goal464077614reaches` | Достижения цели "Покупка" |

## Связанные процессы

- [Advertisings stats](../ingestion/advertisings-stats.md)

