# Ingestion

## Описанные процессы

| Процесс | Тип | Назначение |
|---|---|---|
| [ADB cron PHP](./adb-cron-php.md) | PHP cron | Перенос и пересборка таблиц ADB из MID / CRM |
| [Advertisings stats](./advertisings-stats.md) | n8n | Загрузка расходов и целей рекламных кампаний |
| [Yandex metrica stats UTM and Users](./yandex-metrica-stats-utm-and-users.md) | n8n | Загрузка данных Яндекс.Метрики |
| [Analytics data quality](./analytics-data-quality.md) | n8n | Ежедневное заполнение `adb.analytics_data_quality_daily` |
| [Payment measurement v2](./payment-measurement-v2.md) | PHP cron | Перенос payment outbox/reconciliation из MID в ADB |
| [Refund/cancel v1](./refund-cancel-v1.md) | PHP cron | Перенос refund/cancel events из MID в ADB и основа net paid слоя |

В этой папке описываем процессы загрузки и обновления данных в ADB.

Процесс загрузки отвечает на вопросы:

- чем выполняется загрузка: n8n, cron, PHP-скрипт, SQL job;
- по какому расписанию;
- из какого источника берет данные;
- в какие таблицы ADB пишет;
- есть ли агрегация, дедупликация, нормализация или пересчет истории;
- где смотреть ошибки и логи.

Для нового процесса используйте шаблон:

[../templates/ingestion-template.md](../templates/ingestion-template.md)
