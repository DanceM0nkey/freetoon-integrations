# Homey Flows for FreeToon

Start Homey flows from the Toon and keep an eye on a few devices.

- **Screen** (tap the tile): the flows you chose as buttons with an icon (tap
  = start the flow; the button confirms with a check mark), and the devices you
  chose with their state (power, temperature) and an on/off switch.
- **Tile**: how many of the chosen devices are on, and the last flow started.
- **Settings** (gear): Homey address and token, and two pickers that list
  every flow Homey can start by hand (normal and advanced) and every device.
- **Automatic starts** (optional, off by default): a flow when the Toon goes
  to *Weg* / *Thuis* / *Slapen* / *Comfort*, or at a set time.

## Connection

The daemon uses the Homey's local Web API on your network,
`http://<homey>/api/manager/...`, with `Authorization: Bearer <token>`:

| | |
|---|---|
| list flows | `GET flow/flow/`, `GET flow/advancedflow/` (only `triggerable` ones) |
| start a flow | `POST flow/flow/<id>/trigger`, `POST flow/advancedflow/<id>/trigger` |
| devices | `GET devices/device/` every 20 s |
| switch | `PUT devices/device/<id>/capability/onoff` `{"value":true}` |

**Token.** On a Homey Pro (2019 and later with API keys, 2023, mini) create an
API key under *Settings > API keys* with the rights *flows* (start), *devices*
(read, and control for the switches). That key does not expire.

Older Homeys (Early 2016/2018) have no API keys. A session token works (for
example the one the official `homey` CLI keeps after `homey login`), but Homey
ends those sessions after 24 hours; the tile and screen then say
*Homey-token geweigerd of verlopen*. For such a Homey something has to put a
fresh token in `homey.conf` every day, or the flows have to be started another
way (for example an MQTT trigger card per flow).

## Settings

`/mnt/data/integrations/homey/homey.conf`, edited on the Toon or by hand; the
daemon re-reads it within a second.

```
host=192.168.1.30
token=
# chosen in the app; comma-separated ids, in this order on the screen
flows=
devices=
poll=20
# automatic starts, all off by default; auto=1 switches the rules on
auto=0
# flow id to start when the Toon switches to that program (button or weekly schedule)
on_comfort=
on_home=
on_sleep=
on_away=
# a flow at a time of day (every day)
at1_time=07:00
at1_flow=
at2_time=
at2_flow=
```

**Automatic starts** (settings > *Automatisch starten*): start a flow when the
Toon switches to *Weg*, *Thuis*, *Slapen* or *Comfort* (by a button on the Toon
or by its weekly schedule; read from `happ_thermstat` every 10 s while a rule
is set), or at one of two times of day. Everything is off until `auto=1` and a
flow is chosen. The flow must still be one Homey can start by hand.

## Files

| File | What |
|---|---|
| `homey` | launcher (manifest `binary`), runs `homey.toon2` or `homey.toon1` |
| `homey_daemon.c` | daemon: Homey API, commands from the UI, tile, `state.json` |
| `screen_homey.c` | UI plugin `homey_ui.<toon>.so` |
| `ftapp.h`, `ftjson.h`, `ftui.h` | small shared helpers (same copy in the other apps of this set) |

Live state: `/var/volatile/tmp/homey/` (`state.json`, `flows_all.json`,
`devices_all.json`; tmpfs). Log: `/var/volatile/tmp/integration-homey.log`.

## Build

```sh
make TARGET=toon2 FREETOON_SRC=../../../freetoon-lvgl
make TARGET=toon1 FREETOON_SRC=../../../freetoon-lvgl
make package
```

Install without the Marketplace: `scripts/install-local.sh` in this repo.
