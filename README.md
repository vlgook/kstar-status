# KSTAR · статус

Компактна сторінка статусів гібридного інвертора KSTAR (перевірено на E5KS-D22) поверх порталу [sync.ksdatacloud.com](https://sync.ksdatacloud.com/).

**Встановлення:** https://vlgook.github.io/kstar-status/

## Як це працює

API порталу не дозволяє запити з інших сайтів (немає CORS), тому сторінка запускається закладкою з відкритого порталу:

1. закладка завантажує `dash.html` з GitHub Pages;
2. відкриває нову вкладку з тим самим origin, що й портал;
3. сторінка читає токен сесії порталу з `localStorage` і щохвилини опитує API.

Токен і пароль нікуди не передаються, крім самого sync.ksdatacloud.com.

## Що показує

- наявність мережі (`System Status` = `Island Mode` → мережі немає) і тривалість відключення
- SoC, запас автономії до 10% при поточному навантаженні
- статуси інвертора і DC/DC українською
- активні аварії з кодами (W00, W02, W03…)
- SoC, напругу і струм кожного батарейного модуля
- графік за добу (SoC, навантаження, імпорт, інтервали без мережі) з перемиканням днів
- напругу і частоту мережі (мін–макс за добу), вихід інвертора, резервну лінію
- енергію за день, місяць, рік і весь час
- ресурс батареї в еквівалентних повних циклах
- звук і системне сповіщення про зміну мережі, SoC 30/15/95% і втрату зв'язку, поки вкладка відкрита

## Ендпоінти

| Дані | Запит |
|---|---|
| Станції | `POST /api/web/station/list/page/v2` |
| SoC, потужності, режим | `GET /api/residential/device/detail/logic?collectId=…` |
| Час останніх даних | `GET /api/residential/device/detail/navigation/data?collectId=…` |
| Статуси системи | `GET /api/residential/device/detail/overview/version?deviceId=…&stime=…` |
| Модулі, енергія | `GET /api/residential/device/detail/overview/realdata/v2?deviceId=…&stime=…` |
| Аварії (активні) | `POST /api/web/fault/list/real` |
| Історія тривог з часом зняття (`removeTime`) | `POST /api/web/residential/station/detail/history/fault` `{stationId, pageNumber, pageSize}` |
| 5-хвилинна історія за добу (U, f мережі й інвертора) | `GET /api/residential/history/data/kse/list?deviceId=…&stime=YYYY-MM-DD` |

`overview/*` повертає 500 без поточного київського часу в `stime` і заголовка `Accept-Language`.

Налаштування (ємність батареї, нижня межа SoC, інтервал) на початку скрипта в `dash.html`.
