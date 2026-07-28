# Dashboard: Детализация по рекламным платформам

---
type: dashboard
name: Детализация по рекламным платформам
superset_dashboard_id: 13
datasets:
  - ya_metrica_users_goals_detailed
  - yandex_direct_spend_and_stats_daily
  - ad_stats_and_roi
  - ya_metrica_users_goals_detailed_v2
  - ad_stats_and_roi_v2
  - ad_order_attribution_v2
charts:
  - vyruchka_po_istochnikam
  - yandex_direkt_obshchaya_statistika
  - statistika_yandex_direkt
  - srednyaya_stoimost_tseley_yandex_direkt
  - statisticheskie_dannye_po_reklamnym_ploshchadkam
  - roi_po_ploshchadkam
  - vyruchka_po_ploshchadkam
  - reklamnye_rashody_za_period
  - vyruchka_s_ploshchadok_za_period
  - usrednennyy_roi_po_vsem_ploshchadkam
  - statistika_roi_yandex_direkt
  - vyruchka_po_istochnikam_ymugd_v2
  - ne_agregirovannye_dannye_po_zakazam_s_reklamnyh_ploshchadok
  - statistika_roi_yandex_direkt_adsar_v2
  - roi_po_ploshchadkam_adsar_v2
  - usrednennyy_roi_po_vsem_ploshchadkam_adsar_v2
  - vyruchka_s_ploshchadok_za_period_adsar_v2
  - reklamnye_rashody_za_period_adsar_v2
  - statisticheskie_dannye_po_reklamnym_ploshchadkam_adsar_v2
  - vyruchka_po_ploshchadkam_adsar_v2
---

## Назначение

Дашборд собирает детализацию по рекламным платформам: расходы, рекламную статистику, стоимость действий и связанные показатели выручки.

## Superset

- Dashboard ID: `13`
- Dashboard title: `Детализация по рекламным платформам`
- URL: `http://81.200.152.123:8088/superset/dashboard/13/`

## Чарты

| Chart ID | Чарт | Dataset | Статус документации |
|---|---|---|---|
| `45` | [Выручка по источникам](../charts/vyruchka-po-istochnikam.md) | `ya_metrica_users_goals_detailed` | описан |
| `46` | [Яндекс Директ общая статистика](../charts/yandex-direkt-obshchaya-statistika.md) | `yandex_direct_spend_and_stats_daily` | описан |
| `47` | [Статистика Яндекс Директ](../charts/statistika-yandex-direkt.md) | `yandex_direct_spend_and_stats_daily` | описан |
| `48` | [Средняя стоимость целей Яндекс Директ](../charts/srednyaya-stoimost-tseley-yandex-direkt.md) | `yandex_direct_spend_and_stats_daily` | описан |
| `49` | [Статистические данные по рекламным площадкам](../charts/statisticheskie-dannye-po-reklamnym-ploshchadkam.md) | `ad_stats_and_roi` | описан |
| `50` | [ROI по площадкам](../charts/roi-po-ploshchadkam.md) | `ad_stats_and_roi` | описан |
| `51` | [Выручка по площадкам](../charts/vyruchka-po-ploshchadkam.md) | `ad_stats_and_roi` | описан |
| `52` | [Рекламные расходы за период](../charts/reklamnye-rashody-za-period.md) | `ad_stats_and_roi` | описан |
| `53` | [Выручка с площадок за период](../charts/vyruchka-s-ploshchadok-za-period.md) | `ad_stats_and_roi` | описан |
| `54` | [Усредненный ROI по всем площадкам](../charts/usrednennyy-roi-po-vsem-ploshchadkam.md) | `ad_stats_and_roi` | описан |
| `55` | [Статистика ROI Яндекс Директ](../charts/statistika-roi-yandex-direkt.md) | `ad_stats_and_roi` | описан |

## Карта данных

```text
Yandex Direct via Yandex Metrika API
  -> n8n workflow: Advertisings stats
  -> ADB.adb.ad_direct_spend_and_goals_daily
  -> ADB.adb.ad_spend_daily
  -> Superset dataset: yandex_direct_spend_and_stats_daily
  -> Charts: 46, 47, 48
  -> Dashboard: 13
```

## Comparison charts v2

На dashboard `13` добавлены comparison charts на v2 datasets:

| Chart ID | Чарт | Dataset | Комментарий |
|---:|---|---|---|
| `107` | [Выручка по источникам ymugdV2](../charts/vyruchka-po-istochnikam-ymugd-v2.md) | `adb.ya_metrica_users_goals_detailed_v2` | `SUM(total_value)` после дедупликации заказов |
| `108` | [Не агрегированные данные по заказам с рекламных площадок](../charts/ne-agregirovannye-dannye-po-zakazam-s-reklamnyh-ploshchadok.md) | `adb.ad_order_attribution_v2` | Raw order-level attribution |
| `98` | [Статистика ROI Яндекс Директ adsarV2](../charts/statistika-roi-yandex-direkt-adsar-v2.md) | `adb.ad_stats_and_roi_v2` | `Yandex.Direct`, `MAX(spend/revenue/roi_percent)` |
| `99` | [ROI по площадкам adsarV2](../charts/roi-po-ploshchadkam-adsar-v2.md) | `adb.ad_stats_and_roi_v2` | `SUM(spend)`, `SUM(revenue)`, ROI через суммы |
| `100` | [Усредненный ROI по всем площадкам adsarV2](../charts/usrednennyy-roi-po-vsem-ploshchadkam-adsar-v2.md) | `adb.ad_stats_and_roi_v2` | `AVG(roi_percent)` |
| `101` | [Выручка с площадок за период adsarV2](../charts/vyruchka-s-ploshchadok-za-period-adsar-v2.md) | `adb.ad_stats_and_roi_v2` | `SUM(revenue)` |
| `102` | [Рекламные расходы за период adsarV2](../charts/reklamnye-rashody-za-period-adsar-v2.md) | `adb.ad_stats_and_roi_v2` | `SUM(spend)` |
| `103` | [Статистические данные по рекламным площадкам adsarV2](../charts/statisticheskie-dannye-po-reklamnym-ploshchadkam-adsar-v2.md) | `adb.ad_stats_and_roi_v2` | Raw table с `attributed_orders` и `attribution_model` |
| `104` | [Выручка по площадкам adsarV2](../charts/vyruchka-po-ploshchadkam-adsar-v2.md) | `adb.ad_stats_and_roi_v2` | `SUM(revenue)` по `platform` |

Дополнительно:

```text
MID / CRM databases
  -> PHP cron: adb.php
  -> ADB.orders / order_products / orders_totals
  -> Superset dataset: ya_metrica_users_goals_detailed
  -> Chart: Выручка по источникам
  -> Dashboard: 13
```
