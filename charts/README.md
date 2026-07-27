# Charts

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
