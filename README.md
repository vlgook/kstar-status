# KSTAR · status page

**English** · [Українська](README.uk.md)

A compact status page for KSTAR hybrid inverters that runs on top of the [KSTAR Sync](https://sync.ksdatacloud.com/) portal. It was built by a KSTAR system owner in Ukraine, where scheduled and emergency grid outages happen almost daily, to answer the questions the official web and Android apps leave open: *is the grid on right now, since when, how long will the battery last, and how often does this happen?*

Tested on an **E5KS-D22** with two **BluE-Pack** modules (Wi-Fi datalogger, firmware ARM V1.8.12 / DSP V2.2.04). The interface is available in English and Ukrainian (UA/EN switch in the header; the default follows the browser language).

**Install:** https://vlgook.github.io/kstar-status/

## How it works

The portal API does not allow cross-origin requests (no CORS headers), so the page cannot run from a separate website. Instead it is started by a bookmarklet from an open portal tab:

1. the bookmarklet loads `dash.html` from GitHub Pages;
2. it opens a new tab that shares the portal's origin;
3. the page reads the portal session token from `localStorage` and polls the API every 60 seconds.

The password and token never leave the browser; requests go only to sync.ksdatacloud.com. If the bookmarklet is clicked on a station page (`/station/<id>/…`), that station is shown; otherwise the first station in the account.

## What it shows

| Block | Content |
|---|---|
| Header band | Grid on/off and the exact time of the last change (from the alarm log: `saveTime` of the active W00/W02/W03 alarm, `removeTime` of the last one) |
| Battery | SoC, runtime estimate down to the off-grid discharge limit read from the inverter (`offgriddod`), current load |
| Status | System / inverter / operating mode, connection (by data freshness), firmware with update check |
| Grid and inverter output | Grid voltage and frequency (daily min–max), inverter output, backup line, output current limit, system self-consumption |
| Alarms | Active alarms with codes |
| Battery modules | Per-module SoC, voltage, current; BMS: SOH, cycle count, capacity, cell voltage spread, cell and MOS temperatures, balancing |
| Energy | Today / month / year / total: import, house load, battery charge and discharge, export, self-consumption balance |
| Battery wear | BMS cycle counter vs. a nominal cycle life, lifetime charged and discharged energy |
| Outage statistics | 7/14/28 days: count, total and longest outage, short drops, a 24-hour timeline per day, share of days without grid for each hour |
| Daily chart | SoC, load and grid import with exact outage intervals and short drops marked; grid and inverter voltage with ±10% limits; day navigation |

Extras: **live mode** (portal Quick Update, data every ~10 s, switches itself off after 10 min) and **browser alerts with sound** on grid loss/return, SoC 30/15/95% and datalogger offline while the tab is open.

## Ideas for KSTAR Sync

Things that turned out to be useful in daily life and could live in the official apps:

1. **Push notifications** on grid loss and return, low SoC and datalogger offline. This was the original reason for the project.
2. **"Grid on/off since …"** on the overview screen. The data is already in the alarm log (`saveTime` / `removeTime`), it just is not shown.
3. **Outage statistics**: a per-day timeline and an hour-of-day histogram. In outage-heavy regions this is the first thing owners want to know.
4. **Runtime estimate** at the current load, using the real off-grid DOD setting.
5. **Short grid drops** (40–60 s) are visible only in the alarm log; on 5-minute charts they disappear completely.
6. **System self-consumption** (battery DC minus inverter AC during outages): on this unit it is a steady ~60–65 W, about 1.5 kWh per day.

Observations from working with the API that may help the portal team:

- `overview/version` and `overview/realdata/v2` return HTTP 500 unless `stime` (current station-local time) and an `Accept-Language` header are sent.
- `communStatus` in the station list can show 0 while fresh data keeps arriving.
- The remote settings API returns labels for `ongriddod` / `offgriddod` and Demand Control in Chinese even with `Accept-Language: en` (the web UI translates them itself), and their current values stay empty until a manual Read.
- Firmware updates show only version numbers; release notes would help owners and installers decide when to update.

## API endpoints used

| Data | Request |
|---|---|
| Stations | `POST /api/web/station/list/page/v2` `{current, size}` |
| Station info, time zone | `GET /api/web/station/manager/getinfo?stationId=…` |
| SoC, power flows, mode | `GET /api/residential/device/detail/logic?collectId=…&refreshFlag=false` |
| Time of last data | `GET /api/residential/device/detail/navigation/data?collectId=…` |
| System status, firmware, limits | `GET /api/residential/device/detail/overview/version?deviceId=…&stime=…` |
| Modules, energy counters | `GET /api/residential/device/detail/overview/realdata/v2?deviceId=…&stime=…` |
| 5-min history for a day (grid/inverter U, f, P) | `GET /api/residential/history/data/kse/list?deviceId=…&stime=YYYY-MM-DD` |
| BMS per module (5 min) | `GET /api/residential/history/data/bms/list?deviceId=…&stime=YYYY-MM-DD` |
| Active alarms | `POST /api/web/fault/list/real` `{current, size}` |
| Alarm history with `removeTime` | `POST /api/web/residential/station/detail/history/fault` `{stationId, pageNumber, pageSize}` (up to 200 per page, date filters are ignored) |
| Firmware updates | `GET /api/web/station/detail/need/upgarde/V2?stationId=…` |
| Read an inverter parameter | `POST /api/residential/kse/remote/readvalue` `{deviceId, setItem, command, coefficient, registerCount}` → `GET …/readvalue/result?commandId=…` |
| Quick Update | `GET /api/residential/device/detail/start/refresh?collectId=…` / `stop/refresh`, then `logic?refreshFlag=true` |

All requests use `Authorization: Bearer <token>` from the portal session.

## Settings

At the top of the script in `dash.html`: battery capacity (`BATT_KWH`), fallback discharge limit (`MIN_SOC`), refresh interval and nominal cycle life.

## Disclaimer

This is an independent project, not affiliated with or endorsed by KSTAR. It uses the same API as the portal's own web interface and only reads data; the only commands it sends are Quick Update start/stop and parameter reads.
