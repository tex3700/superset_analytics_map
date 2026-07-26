# Chart: Распределение менеджеров по заказам

---
type: chart
name: Распределение менеджеров по заказам
superset_chart_id: 74
dataset: mart_lid_orders
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Horizontal bar chart показывает количество заказов по менеджерам.

## Superset

- Chart ID: `74`
- Visualization type: `echarts_timeseries_bar`
- Dataset: `adb.mart_lid_orders`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=74`

## Группировка менеджеров

В `x_axis` используется SQL CASE для нормализации логинов менеджеров:

| Raw manager | Label |
|---|---|
| empty / `NULL` | `! Нет данных` |
| `Ekhelifi` | `Екатерина Хелифи` |
| `Urovskih` | `Екатерина Юровских` |
| `azambayev` | `Ануар Азамбаев` |
| `dmitriyivanov` | `Дмитрий Иванов` |
| `iskander` | `Искандер Хелифи` |
| `midrawfood` | `Евгения Германовна` |
| `orazbakov` | `Нурбек Оразбаков` |
| `tulyakov` | `Дмитрий Туляков` |
| `yakovlev` | `Александр Яковлев` |
| `akirirowmid` | `Андрей Кирейчук` |
| другое значение | исходный `manager` |

## Метрика

| Метрика | Формула |
|---|---|
| `count` | `COUNT(*)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `date_task_added` | `No filter` |
| `date_order_added` | `Last month` |

## Агрегация в чарте

Чарт считает строки заказов по нормализованному менеджеру.

