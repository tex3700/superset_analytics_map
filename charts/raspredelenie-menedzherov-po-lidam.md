# Chart: Распределение менеджеров по лидам

---
type: chart
name: Распределение менеджеров по лидам
superset_chart_id: 75
dataset: mart_lids
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Horizontal bar chart показывает количество лидов по менеджерам задачи.

## Superset

- Chart ID: `75`
- Visualization type: `echarts_timeseries_bar`
- Dataset: `adb.mart_lids`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=75`

## Группировка менеджеров

В `x_axis` используется SQL CASE для нормализации логинов менеджеров:

| Raw `task_manager` | Label |
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
| другое значение | исходный `task_manager` |

## Метрика

| Метрика | Формула |
|---|---|
| `count` | `COUNT(*)` |

## Фильтры

| Фильтр | Значение |
|---|---|
| `lid_date_added` | `No filter` |
| `task_date_added` | `Last month` |

## Агрегация в чарте

Чарт считает строки лидов по нормализованному менеджеру задачи.

