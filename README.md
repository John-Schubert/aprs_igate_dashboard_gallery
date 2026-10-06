# APRS iGate Dashboard — Screenshot Gallery

A public, static timeline of screenshots from the web dashboard of a home
APRS iGate, callsign W5UFV-1. The dashboard itself runs on a private network;
this repo exists to make its screenshots viewable.

**Live gallery:** https://john-schubert.github.io/aprs_igate_dashboard_gallery/

## What the project does

APRS is an amateur radio system for short digital packets: positions,
weather, and text messages, on 144.390 MHz in North America. An iGate is a
station that passes those packets between radio and the internet (APRS-IS).

This one runs on a Raspberry Pi Zero 2 W:

- **Receive:** an RTL-SDR dongle on a SlimJim antenna feeds `rtl_fm`, piped
  into the Dire Wolf software modem, which decodes packets and gates them to
  APRS-IS.
- **Transmit:** a DigiRig Mobile sound-card interface drives a Baofeng UV-5R
  handheld. Dire Wolf sends audio through the DigiRig and keys the radio with
  the serial RTS line.
- **Message return path:** when a message or ACK arrives from the internet
  for a station the iGate has heard on the air in the last three hours, the
  iGate transmits it. Without this, a handheld that sends a message never
  hears the ACK and keeps retrying. Digipeating is off; the station only
  transmits return traffic.
- **Self-recovery:** a udev rule restarts the service when the SDR
  re-enumerates on USB, because `rtl_fm` otherwise keeps running against a
  device that is gone and silently receives nothing.

The web dashboard is static HTML and JavaScript served by nginx, fed by JSON
files that cron jobs regenerate every minute:

- **Messages:** every message and ACK that passed through the iGate, with
  time, direction (heard on RF, or sent to RF from the internet), from, to
  and text. Dire Wolf's own log does not record messages, so a small Python
  collector reads the service journal and keeps its own history.
- **Dashboard:** packets per hour, signal level over time, a Leaflet map of
  heard positions, and a table of the stations heard most.
- **Live feed:** each packet decoded today.
- **System stats:** CPU, memory, temperature and disk on the Pi Zero.

## How it was built

Built and maintained with Claude Code working over SSH from a second
Raspberry Pi: diagnosing a three-week receive outage down to a USB
re-enumeration, adding the restart rule, bringing up the transmit side, and
writing the Messages page and its collector. Changes on the iGate are backed
up first, and the first on-air transmission was done with the operator
present.

## Adding a new snapshot

Save new captures (PNG, one per view key in `manifest.json`) to the source
folder, then run:

```bash
scripts/add_snapshot.sh "optional note about what changed"
```

This copies the PNGs into a new dated folder under `snapshots/` and appends
an entry to `manifest.json`. It does not commit or push.
