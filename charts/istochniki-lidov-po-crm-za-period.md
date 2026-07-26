# Chart: Источники лидов по CRM за период

---
type: chart
name: Источники лидов по CRM за период
superset_chart_id: 69
dataset: mart_lids
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Donut/pie chart показывает распределение лидов по CRM-проектам.

## Superset

- Chart ID: `69`
- Visualization type: `pie`
- Dataset: `adb.mart_lids`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=69`

## Группировка

```text
project_name
```

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

Чарт считает количество строк лидов по `project_name`.

