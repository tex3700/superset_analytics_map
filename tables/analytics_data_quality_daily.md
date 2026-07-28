# ADB Table: analytics_data_quality_daily

---
type: table
name: analytics_data_quality_daily
database: ADB
schema: adb
source_system: ADB quality checks
loaded_by:
  - n8n:analytics_data_quality
used_by_datasets:
  - analytics_data_quality_daily
---

## Назначение

Таблица хранит ежедневные результаты проверок качества аналитики. Сейчас проверки считаются за rolling window 90 дней и записываются через n8n workflow `Analytics data quality`.

Заполнение выполняется вызовом хранимой процедуры:

```sql
CALL adb.refresh_analytics_data_quality_daily();
```

## Grain

```text
1 check_date + 1 check_name
```

## Поля

| Поле | Описание |
|---|---|
| `id` | Технический ID строки |
| `check_date` | Дата проверки |
| `check_name` | Техническое имя проверки |
| `check_group` | Группа проверки |
| `metric_value` | Числовое значение |
| `metric_text` | Текстовое значение |
| `status` | `ok`, `warning`, `info` |
| `comment` | Комментарий |
| `created_at` | Время вставки/обновления |

## Обновление

- Процесс: [Analytics data quality](../ingestion/analytics-data-quality.md)
- Частота: ежедневно по расписанию n8n
- Метод: `REPLACE INTO` внутри procedure `refresh_analytics_data_quality_daily`
- Окно расчета: последние 90 дней

