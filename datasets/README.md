# Datasets

В этой папке описываем датасеты Superset.

Датасет является центральной сущностью документации. Через него связываются источники, таблицы ADB, чарты и дашборды.

Для нового датасета используйте шаблон:

[../templates/dataset-template.md](../templates/dataset-template.md)

## Временные / сравнительные датасеты

После разбора customer objections по ROI/CAC добавлены две v2-версии:

| Dataset | ID | Назначение |
|---|---:|---|
| [ya_metrica_users_goals_detailed_v2](./ya_metrica_users_goals_detailed_v2.md) | `29` | Order-level attribution: один связанный заказ не должен размножаться по нескольким строкам Метрики |
| [ad_stats_and_roi_v2](./ad_stats_and_roi_v2.md) | `30` | Сравнительная версия ROI/выручки с дедупликацией выручки на уровне `shop_order_id` |

Старые datasets `15` и `18` оставлены в Superset: `15` как диагностический, `18` для сверки старых и новых показателей.
