# Dashboard: Детализация по продажам и воронке лидов

---
type: dashboard
name: Детализация по продажам и воронке лидов
superset_dashboard_id: 14
datasets:
  - order_products_ym
  - mart_lid_orders
  - mart_lids
  - rel_lids_orders
  - con_lids_orders
  - rel_lid_orders
  - source_abadoned_orders
charts:
  - tovary_v_zakaze
  - analitika_lidov_po_zakazam
  - obshchaya_vyruchka_za_period
  - obshchaya_pribyl_za_period
  - dinamika_vyruchka_i_pribyl_po_dnyam
  - novye_i_loyalnye_pokupateli_za_period
  - kolichestvo_zakazov_za_period
  - srednee_vremya_ot_registratsii_lida_do_pervogo_zakaza_v_chasah
  - sredniy_chek_aov_po_zakazam_za_period
  - raspredelenie_menedzherov_po_zakazam
  - detalizatsiya_po_lidam
  - istochniki_lidov_po_crm_za_period
  - raspredelenie_menedzherov_po_lidam
  - otnoshenie_zakazy_lidy
  - konversiya_lid_zakaz
  - broshennye_zakazy_s_privyazkoy_k_istochniku
  - istochniki_po_broshennym_zakazam
---

## Назначение

Дашборд показывает детализацию продаж, заказов, лидов, менеджеров и конверсии лида в заказ.

## Superset

- Dashboard ID: `14`
- Dashboard title: `Детализация по продажам и воронке лидов`
- URL: `http://81.200.152.123:8088/superset/dashboard/14/`
- Chart count: `21`

## Чарты текущего блока `mart_lid_orders`

| Chart ID | Чарт | Dataset | Статус документации |
|---|---|---|---|
| `58` | [Аналитика лидов по заказам](../charts/analitika-lidov-po-zakazam.md) | `mart_lid_orders` | описан |
| `60` | [Общая выручка за период](../charts/obshchaya-vyruchka-za-period.md) | `mart_lid_orders` | описан |
| `61` | [Общая прибыль за период](../charts/obshchaya-pribyl-za-period.md) | `mart_lid_orders` | описан |
| `62` | [Динамика выручка и прибыль по дням](../charts/dinamika-vyruchka-i-pribyl-po-dnyam.md) | `mart_lid_orders` | описан |
| `63` | [Новые и лояльные покупатели за период](../charts/novye-i-loyalnye-pokupateli-za-period.md) | `mart_lid_orders` | описан |
| `64` | [Количество заказов за период](../charts/kolichestvo-zakazov-za-period.md) | `mart_lid_orders` | описан |
| `65` | [Среднее время от регистрации лида до первого заказа (в часах)](../charts/srednee-vremya-ot-registratsii-lida-do-pervogo-zakaza-v-chasah.md) | `mart_lid_orders` | описан |
| `70` | [Средний чек (AOV) по заказам за период](../charts/sredniy-chek-aov-po-zakazam-za-period.md) | `mart_lid_orders` | описан |
| `74` | [Распределение менеджеров по заказам](../charts/raspredelenie-menedzherov-po-zakazam.md) | `mart_lid_orders` | описан |

## Остальные чарты дашборда

| Chart ID | Чарт | Dataset | Статус документации |
|---|---|---|---|
| `44` | [Товары в заказе](../charts/tovary-v-zakaze.md) | `order_products_ym` | описан |
| `59` | [Детализация по лидам](../charts/detalizatsiya-po-lidam.md) | `mart_lids` | описан |
| `66` | Новых лидов за период | `crm_mid2.mart_lid` | не описан |
| `67` | Квалифицированные лиды за период | `crm_mid2.mart_lid` | не описан |
| `68` | Брошенные заказы за период | `crm_mid2.mart_lid` | не описан |
| `69` | [Источники лидов по CRM за период](../charts/istochniki-lidov-po-crm-za-period.md) | `mart_lids` | описан |
| `71` | Динамика лидов и заказов по дням | `rel_lids_orders` | не описан |
| `72` | [Отношение Заказы - Лиды](../charts/otnoshenie-zakazy-lidy.md) | `con_lids_orders` | описан |
| `73` | [Конверсия (Лид -> Заказ)](../charts/konversiya-lid-zakaz.md) | `rel_lid_orders` | описан |
| `75` | [Распределение менеджеров по лидам](../charts/raspredelenie-menedzherov-po-lidam.md) | `mart_lids` | описан |
| `76` | [Брошенные заказы с привязкой к источнику](../charts/broshennye-zakazy-s-privyazkoy-k-istochniku.md) | `source_abadoned_orders` | описан |
| `77` | [Источники по брошенным заказам](../charts/istochniki-po-broshennym-zakazam.md) | `source_abadoned_orders` | описан |

## Native Filters

| Фильтр | Тип | Target |
|---|---|---|
| `По ID заказа` | select | dataset `19`, column `shop_order_id` |
| `По ID Лида/Контакта` | select | dataset `20`, column `contact_id` |
| `По ID покупателя` | select | dataset `19`, column `customer_id` |
| `Временной промежуток` | time | all |
| `Группировка по периоду` | timegrain | dataset `19` |
