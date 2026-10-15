<div align="center">

  <img alt="W1R3L3556H05T" src="./W1R3L3556H05TLOGO.png" width="260" />

  <h1>W1R3L3556H05T <sub>("WirelessGhost")</sub></h1>

  <p><b>Phantom-class wireless security firmware for the Attaky Cyber Deck Plus</b></p>

  <p>
    <img src="https://img.shields.io/badge/version-0.1.0-blue?style=flat-square" alt="Version" />
    <img src="https://img.shields.io/badge/target-ESP32--S3_%C2%B7_16MB-2C2C2C?style=flat-square" alt="Target" />
    <img src="https://img.shields.io/badge/hardware-Attaky_Cyber_Deck_Plus-C0252B?style=flat-square" alt="Hardware" />
    <img src="https://img.shields.io/badge/build-PlatformIO_%C2%B7_Arduino-007ACC?style=flat-square&logo=platformio&logoColor=white" alt="Build" />
    <img src="https://img.shields.io/badge/framework-ESP--IDF_v5.5.4-E00?style=flat-square&logo=espressif&logoColor=white" alt="Framework" />
  </p>

  <p><i>"Authorized hardware test" — for use only on networks and devices you own or have explicit permission to test.</i></p>

</div>

---

**W1R3L3556H05T** (codename **Phantom**, pronounced *WirelessGhost*) is a from-scratch
wireless auditing and detection firmware for the **Attaky Cyber Deck Plus**. It turns the
handheld into a self-contained 2.4 GHz lab: full Wi-Fi and BLE reconnaissance, live
detection of deauth floods, evil twins, Pineapples, Pwnagotchis, skimmers and Flipper-class
adversaries, a complete offensive suite (evil twin + captive portal, PMKID catcher, honeypot,
beacon flood, BLE spam), HID/BadUSB tooling, and a Wireshark-grade pcap pipeline straight to
USB — all driven from a ~50-screen on-device UI or a full serial CLI.

> ⚠️ **Authorized use only.** Every feature that transmits — deauth, evil twin, BLE spam,
> HID injection — is intended for hardware and networks you own, or assessments you have
> written permission to run. The firmware boots with an *"Authorized hardware test"*
> banner to keep that in view. You are responsible for complying with local law.

## Table of contents

- [Highlights](#highlights)
- [Hardware](#hardware)
- [Firmware at a glance](#firmware-at-a-glance)
- [Feature tour](#feature-tour)
  - [Recon](#recon)
  - [Detection](#detection)
  - [Phantom (offensive suite)](#phantom-offensive-suite)
  - [HID / BadUSB](#hid--badusb)
  - [Connection manager](#connection-manager)
  - [Tools & settings](#tools--settings)
- [Flashing](#flashing)
- [Serial console](#serial-console)
- [pcap → Wireshark workflow](#pcap--wireshark-workflow)
- [Configuration](#configuration)
- [Project status](#project-status)
- [Authorized use](#authorized-use)
- [Credits](#credits)

## Highlights

- **See the whole 2.4 GHz picture.** Continuous Wi-Fi AP scan, client/station tracking,
  BLE scan with GATT explorer, per-channel visualization and Wi-Fi/BLE noise floors —
  with PSRAM-backed scan tables for dense environments.
- **Know you're being attacked.** Dedicated detectors watch for deauth floods, DoS floods,
  evil twins, WiFi Pineapple signatures, Pwnagotchi probing, RF skimmers, Flipper Zero
  advertising and general anomalies, each with its own alert screen.
- **Bio-sense over the air.** CSI (channel state information) sampled at up to 25 frames/s
  is filtered in the 0.6–3.5 Hz band to estimate a target's **heart rate in BPM** from
  Wi-Fi reflections — a genuine contactless vital-sign experiment, on a deck.
- **A full offensive suite.** Evil twin with captive portal, PMKID catcher that clones
  SSID + BSSID, open-network honeypot with bait SSID, beacon flood, deauth bursts and
  Apple/Android/Microsoft BLE spam.
- **HID in three forms.** A BLE keyboard (`WirelessGhost KB`), scripted BadUSB payloads
  (notepad demo, `Run CMD`, WIN+L lock) and keystroke **replay** from a pad log.
- **pcap over USB.** Stream raw 802.11 frames to the host and open them live in Wireshark;
  the console mutes itself while the capture owns the port.
- **Everything is scriptable.** A grouped CLI (`wifi`, `ble`, `lists`, `radio`, `attack`,
  `config`, `screen`, `shot`) with TAB completion, history and scrollback — plus `shot`,
  which dumps the raw RGB565 framebuffer to the host for documentation screenshots.

## Hardware

| Item | Requirement |
|---|---|
| Device | Attaky Cyber Deck Plus (or any Attaky build around the Core) |
| SoC | ESP32-S3, 16 MB QuadSPI flash, PSRAM |
| Modules | Core + keyboard (L/R) + gamepad (L/R) + Fuel power module — detected at boot, LED zones addressed independently |
| Storage | microSD (FAT) for the boot logo (`/W1R3L3556H05TLOGO.png`), pcap logs and captured data |
| Link to PC | USB-C — serial console, pcap streaming and flashing share this port |

Built and validated against the `attaky-cyber-deck-plus` PlatformIO environment using the
Attaky M5Unified fork (`M5Unified-attaky`) for module detection, IMU, fuel gauge and LED control.

## Firmware at a glance

| Property | Value |
|---|---|
| Version | `0.1.0` |
| Boot banner | `v0.1.0 - Cyber Deck Plus` / `Authorized hardware test` |
| Image | merged full-flash image (bootloader + partitions + app) |
| Flash offset | `0x0` (flash mode/frequency: **keep**) |
| Size | 2,480,128 bytes (≈ 2.37 MiB) |
| SHA-256 | `BAE8F265A0FBCAEB92FBDC18898367473D4D8DBD932E129427BD31F6C8464B56` |
| Toolchain | PlatformIO · Arduino core on ESP-IDF v5.5.4 · NimBLE-Arduino |
| BLE stack | NimBLE (host + GATT client/server) |
| Build stamp | `attaky-cyber-deck-plus / toolchain: PlatformIO — 11:17:54, Jun 2 2026` |

Always verify the checksum after downloading:

```bash
sha256sum W1R3L3556H05T.bin
# BAE8F265A0FBCAEB92FBDC18898367473D4D8DBD932E129427BD31F6C8464B56
```

## Feature tour

The UI is a screen tree of roughly 50 screens grouped into five areas. A selection:

### Recon

| Screen | What it does |
|---|---|
| `wifi-scan` / `wifi-detail` / `wifi-track` | AP discovery, detail view (BSSID, channel, cipher, clients), motion tracking of a chosen AP |
| `ble-scan` / `ble-detail` / `ble-gatt` | BLE advertisement scan, device detail, full GATT service/characteristic map with read & write |
| `station-scan` / `scan-clients` / `client-detail` | Client (station) discovery and per-client detail |
| `channel-viz` | Live channel activity visualization |
| `noise-wifi` / `noise-ble` | 2.4 GHz noise-floor measurement for Wi-Fi and BLE bands |

### Detection

| Screen | Catches |
|---|---|
| `det-deauth` | Deauth floods — bursts, sources per channel, reason codes |
| `det-dos` | Denial-of-service floods, forced client disconnects |
| `det-scan` | Generic detection scan across signatures |
| `pineapple` | WiFi Pineapple probing/management-frame signatures |
| `port-scan` / `http-scan` / `ping-sweep` / `traceroute` | LAN reconnaissance against a target range |
| `anomaly` / `anom-alert` / `anom-style` | Behavioral anomaly detection with alert styling |
| `bio-sense` | **Heart-rate estimation from Wi-Fi CSI** — locks the 0.6–3.5 Hz band, reports BPM and lock confidence |

### Phantom (offensive suite)

| Screen | Effect |
|---|---|
| `evil-twin` / `evil-portal` / `evil-pick` / `evil-ssid` / `evil-log` | Clone an AP, serve a captive portal, log hits |
| `pmkid` / `pmk-pick` / `pmk-log` (`GhostPMKID1296`) | PMKID catcher that clones SSID + BSSID, deauth-lures clients, logs crackable PMKIDs |
| `pot-ssid` (`Phantom / Pot`) | Open-network honeypot with a bait SSID and hit counter |
| `phantom` → clone AP / beacon flood | Fake-AP spam |
| `phantom` → deauth | Directed deauth bursts (`deauth start [ch\|bssid]`) |
| `ble-spam` | Apple / Android / Microsoft pairing-popup spam, cycling advertisements |
| `phantom` → captured | Harvested credentials store |

Flipper-class devices are themselves detected and named (`flippers advertise by name`).

### HID / BadUSB

| Screen | What it does |
|---|---|
| `hid-pad` | On-screen pad → keystrokes, with a replayable pad log |
| `hid-scripts` | BadUSB scripts — `Notepad demo`, `Run CMD`, `Lock (WIN+L)` |
| `hid-replay` | Replay captured keystroke sequences |

The deck also advertises as the BLE keyboard **`WirelessGhost KB`** and can inject text
over the air (`input text <words>`).

### Connection manager

`conn-scan` → `conn-entry` (manual or scanned SSID) → `conn-info` / `conn-ap` /
`conn-saved`. Networks are saved to NVS with **auto-reconnect at boot**; `FN+ENTER`
forgets a network. Connection states are surfaced explicitly (`STARTING …`, `ASSOCIATING`,
`WAITING DHCP`, `AUTH FAILED`, `HANDSHAKE TIMEOUT`).

### Tools & settings

- `wireshark` — live pcap stream over USB (see below)
- `terminal` — the **Phantom CLI** on-device
- `theme` — UI theme selection; `settings` — brightness, privacy masking, RGB, device name, channel hop rate
- `about` — firmware, module and build info

## Flashing

The image is a **merged full-flash image** — write it at offset `0x0` and keep the
flash geometry untouched.

**esptool (any OS):**

```bash
esptool.py --chip esp32s3 --port <PORT> -b 460800 write_flash 0x0 W1R3L3556H05T.bin
```

- Use **460800 baud** (the proven rate for this pipeline; 921600 tends to fail mid-handshake)
  and drop to `115200` if your cable is flaky.
- No need for `--flash_mode/--flash_freq` overrides — the merged image carries correct
  headers and the deck runs 16 MB QuadSPI.
- If a previous firmware fills the flash, add `erase_flash` first.

**From a phone:** flash it with the Attaky Flasher Android app over USB-C — put this
`.bin` in the bundled flasher's `firmware/` folder and register it in `manifest.json`
(offset `0`, `flashSize: "keep"`, the SHA-256 above), or side-load it through any
esptool-js web flasher.

After flashing the deck reboots to the splash (logo loaded from SD, then the version
banner) and lands on the main menu.

## Serial console

Open the USB serial port at **115200 baud**. The console supports TAB completion,
command history (`FN+↑/↓`), scrollback (`↑/↓`), and exits on an empty line + `BKSP`.
While a pcap/log-analyzer stream is running the console mutes itself — stop the stream first.

```
commands (grouped):
  system   ping | version | status | help | reboot
  screen   screen [list|<name>]        jump to a screen by name
  input    key <up|down|left|right|enter|back|tab|c> [n] | text <words>
  wifi     wifi scan start|stop|status | wifi join <ssid> [pass] | wifi leave
  ble      ble scan|noise start|stop
  lists    aps [n] | bles [n] | stas [n]   masked when privacy is on
  radio    rf [s] | chan [watch|hist] | noise | noisek   (these DRAIN)
  attack   deauth start [ch|bssid] | deauth stop
  config   set hop_ms|brightness|privacy|rgb|name <value>
  shot     shot   raw RGB565 framebuffer blob (host only)
```

`status` returns one machine-readable line:

```
fw=%s ver=%s scr=%s uptime=%lus batt=%d sd=%d sniffer=%d pcap=%d ble=%d deauth=%d sta=%s ap=%d rssi=%d
```

## pcap → Wireshark workflow

1. On the deck, open `wireshark` (`Phantom USB / Wireshark`) and start a capture —
   the sniffer owns the USB port from here.
2. On the host, the stream arrives as a raw packet blob (`WG/1 =BLOB` framing);
   open the stream in Wireshark (or a capture file from SD via the log analyzer).
3. Stop the capture to hand the port back to the console.

Frame and volume counters are shown on-device (`%lu frames  %lu KB`), and full-screen
documentation dumps can be taken any time with the `shot` command.

## Configuration

Runtime settings persist and are editable from `settings` or the CLI:

| Setting | CLI | Notes |
|---|---|---|
| Channel hop rate | `set hop_ms <ms>` | scan dwell per channel |
| Backlight | `set brightness <n>` | |
| Privacy | `set privacy <on\|off>` | masks SSIDs/MACs in `aps`/`bles`/`stas` listings |
| LEDs | `set rgb <n>` | per-zone control (Core / kbd L-R / gamepad L-R) |
| Device name | `set name <text>` | shown in BLE identity and UI |

## Project status

**v0.1.0 — feature-complete alpha on real hardware.**

- Boots and runs on the Attaky Cyber Deck Plus; modules (kbd/gamepad/fuel) detected at boot.
- Recon, detection, Phantom suite, HID and CLI all operational.
- Known rough edges: BLE init occasionally fails under low-internal-heap conditions and
  retries automatically (see boot diagnostics); `bio-sense` is experimental and accuracy
  depends on geometry and motion.
- No OTA channel yet — flashing is USB only.

## Authorized use

This firmware can actively transmit on the 2.4 GHz band, emulate access points, request
client disassociations and emulate input devices. Run it **only**:

- on networks and hardware you own,
- on networks where you have explicit, prior written permission,
- within the scope of an authorized security assessment,
- in compliance with applicable law (which varies by country — jamming and unauthorized
  access are criminal offenses nearly everywhere).

The author assumes no liability for misuse.

## Credits

- **Author:** WirelessGhost
- Hardware: Attaky (https://attaky.com) Cyber Deck Plus — Core, keyboard, gamepad and Fuel modules
- Stack: ESP-IDF v5.5.4 · Arduino core · NimBLE-Arduino · `M5Unified-attaky`
- Flashing: esptool.py · Attaky Flasher (Android)

<div align="center">
  <sub>W1R3L3556H05T · "WirelessGhost" · v0.1.0 — <i>see everything, fear nothing.</i></sub>
</div>
