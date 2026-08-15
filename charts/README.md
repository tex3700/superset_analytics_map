# Charts

## Data quality charts

| Chart ID | Chart | Dataset |
|---:|---|---|
| `108` | [Не агрегированные данные по заказам с рекламных площадок](./ne-agregirovannye-dannye-po-zakazam-s-reklamnyh-ploshchadok.md) | `ad_order_attribution_v2` |
| `109` | [Динамика - Заказы без ym_uid 90d](./dinamika-zakazy-bez-ym-uid-90d.md) | `analytics_data_quality_daily` |
| `110` | [Заказы без ym_uid 90d](./zakazy-bez-ym-uid-90d.md) | `analytics_data_quality_daily` |
| `111` | [Всего заказов 90d](./vsego-zakazov-90d.md) | `analytics_data_quality_daily` |
| `112` | [Не связанные с метрикой заказы 90d](./ne-svyazannye-s-metrikoy-zakazy-90d.md) | `analytics_data_quality_daily` |
| `113` | [Коэффициент соответствия атрибуции](./koeffitsient-sootvetstviya-atributsii.md) | `analytics_data_quality_daily` |
| `114` | [Динамика - Не связанные с метрикой заказы 90d](./dinamika-ne-svyazannye-s-metrikoy-zakazy-90d.md) | `analytics_data_quality_daily` |
| `115` | [Атрибутированная выручка 90d](./atributirovannaya-vyruchka-90d.md) | `analytics_data_quality_daily` |
| `116` | [Динамика качества](./dinamika-kachestva.md) | `analytics_data_quality_daily` |
| `117` | [Доля атрибутированной выручки от общей суммы заказов](./dolya-atributirovannoy-vyruchki-ot-obshchey-summy-zakazov.md) | `analytics_data_quality_daily` |
| `118` | [История проверок](./istoriya-proverok.md) | `analytics_data_quality_daily` |

## Payment v2 raw charts

| Chart ID | Chart | Dataset |
|---:|---|---|
| `120` | [События payment outbox и сверка](./sobytiya-payment-outbox-i-sverka.md) | `payment_events_v2` |
| `121` | [Канонические оплаты payment v2](./kanonicheskie-oplaty-payment-v2.md) | `order_payments_v2` |
| `122` | [Атрибуция оплат по рекламным кампаниям](./atributsiya-oplat-po-reklamnym-kampaniyam.md) | `payment_campaign_attribution_v2` |

В этой папке описываем чарты Superset.

Для каждого чарта нужно указать:

- какой датасет используется;
- какие поля используются;
- какие метрики считаются;
- какие группировки и фильтры применяются;
- агрегирует ли чарт данные поверх уже агрегированного датасета;
- на каких дашбордах расположен чарт.

Для нового чарта используйте шаблон:

[../templates/chart-template.md](../templates/chart-template.md)

## Comparison charts v2

После разбора customer objections по ROI/CAC добавлены comparison charts на v2 datasets:

| Chart ID | Chart | Dataset |
|---:|---|---|
| `105` | [Метрика по пользователям достигшим цели ymugdV2](./metrika-po-polzovatelyam-dostigshim-tseli-ymugd-v2.md) | `ya_metrica_users_goals_detailed_v2` |
| `106` | [Доля трафика по источнику первого визита ymugdV2](./dolya-trafika-po-istochniku-pervogo-vizita-ymugd-v2.md) | `ya_metrica_users_goals_detailed_v2` |
| `107` | [Выручка по источникам ymugdV2](./vyruchka-po-istochnikam-ymugd-v2.md) | `ya_metrica_users_goals_detailed_v2` |
| `98` | [Статистика ROI Яндекс Директ adsarV2](./statistika-roi-yandex-direkt-adsar-v2.md) | `ad_stats_and_roi_v2` |
| `99` | [ROI по площадкам adsarV2](./roi-po-ploshchadkam-adsar-v2.md) | `ad_stats_and_roi_v2` |
| `100` | [Усредненный ROI по всем площадкам adsarV2](./usrednennyy-roi-po-vsem-ploshchadkam-adsar-v2.md) | `ad_stats_and_roi_v2` |
| `101` | [Выручка с площадок за период adsarV2](./vyruchka-s-ploshchadok-za-period-adsar-v2.md) | `ad_stats_and_roi_v2` |
| `102` | [Рекламные расходы за период adsarV2](./reklamnye-rashody-za-period-adsar-v2.md) | `ad_stats_and_roi_v2` |
| `103` | [Статистические данные по рекламным площадкам adsarV2](./statisticheskie-dannye-po-reklamnym-ploshchadkam-adsar-v2.md) | `ad_stats_and_roi_v2` |
| `104` | [Выручка по площадкам adsarV2](./vyruchka-po-ploshchadkam-adsar-v2.md) | `ad_stats_and_roi_v2` |
