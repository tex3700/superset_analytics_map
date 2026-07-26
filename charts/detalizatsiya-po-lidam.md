# Chart: Детализация по лидам

---
type: chart
name: Детализация по лидам
superset_chart_id: 59
dataset: mart_lids
dashboards:
  - Детализация по продажам и воронке лидов
---

## Назначение

Raw table показывает список лидов/контактов с задачей, проектом, статусом, свойствами и менеджерами.

## Superset

- Chart ID: `59`
- Visualization type: `table`
- Query mode: raw
- Dataset: `adb.mart_lids`
- Dashboard: `14`
- URL: `http://81.200.152.123:8088/explore/?slice_id=59`

## Используемые поля

```text
contact_id, task_id, lid_date_added, task_date_added, project_name,
lid_reason, task_status, task_property, lid_property,
task_manager, lid_manager, lid_firstname, lid_lastname,
lid_email, lid_phone, lid_phone_calls, lid_company
```

## Фильтры

| Фильтр | Значение |
|---|---|
| `task_date_added` | `Last quarter` |

## Сортировка

```text
lid_date_added DESC
```

## Агрегация в чарте

Чарт не агрегирует данные. Показывает строки physical dataset.

