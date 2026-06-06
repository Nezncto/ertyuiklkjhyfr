# ESP32 Marauder — Complete Command Guide
### Board: XIAO ESP32-S3 | Firmware: v1.11.0



![Marauder Mini](marauder_mini.jpg)


---

## Acknowledgement

A huge thank you to **justcallmekoko** for creating and maintaining the ESP32 Marauder project.
This is an incredible open-source tool and none of this would be possible without their work.

> Original project: [https://github.com/justcallmekoko/ESP32Marauder](https://github.com/justcallmekoko/ESP32Marauder)

---

## Rules of Using This Tool

Before you do anything, read and follow these rules:

1. **Only test on networks and devices you personally own** — do not use this on your neighbour's WiFi, public hotspots, or anyone else's network without written permission.
2. **Do not use this to harm, spy on, or disrupt other people** — deauth attacks, beacon spam, and BLE spam are disruptive and illegal when aimed at others.
3. **Do not use this to steal data or credentials** — the Evil Portal and sniffing tools must only be used in controlled environments you own.
4. **Do not use this at schools, workplaces, airports, or public spaces** — disrupting WiFi in public is a criminal offence in most countries.
5. **This tool cannot crack passwords** — do not use it expecting to break into someone's network.
6. **You are fully responsible** for how you use this firmware. The original developer and this guide take no responsibility for misuse.

> Misuse of this tool can result in **criminal charges** under computer misuse laws (e.g. Computer Fraud and Abuse Act in the US, Computer Misuse Act in the UK, IT Act in Sri Lanka/India).

---

## Changes Made to Compile on XIAO ESP32-S3

The original source code was written for older ESP32 hardware and an older ESP-IDF version. The following changes were made to get it compiling on the **XIAO ESP32-S3** with **Arduino ESP32 Core 3.3.3 (ESP-IDF 5.x)**:

### 1. `configs.h`
- Uncommented `#define XIAO_ESP32_S3` to enable the XIAO S3 board target
- Added `#define HAS_IDF_3` inside the XIAO S3 config block to enable ESP-IDF 5.x compatible code paths

### 2. `EvilPortal.h`
- Changed the `index_html` array from a definition to an `extern` declaration to fix a **"multiple definition"** linker error when included in multiple `.cpp` files

### 3. `EvilPortal.cpp`
- Added the actual definition of `char index_html[MAX_HTML_SIZE]` for the non-PSRAM case to match the `extern` declaration above

### 4. `WiFiScan.cpp`
- Wrapped the definition of `ieee80211_raw_frame_sanity_check()` in `#ifndef HAS_IDF_3` to avoid a **"multiple definition"** conflict with ESP-IDF 5.x which already defines this function
- Added an `extern "C"` declaration for this function in the `#else` branch so it can still be called when `HAS_IDF_3` is defined

### 5. NimBLE-Arduino Library
- Switched the NimBLE-Arduino library git checkout from a broken `3.0.0-dev` branch to tag **`1.4.3`** for API and mbedTLS 3.x compatibility
- Added `#include <ctime>` to `NimBLECharacteristic.h`, `NimBLEAdvertisedDevice.h`, and `NimBLERemoteCharacteristic.h` to fix **`time_t` undeclared** errors caused by stricter GCC 14.2.0

### 6. ESPAsyncWebServer Library
- Replaced the `lacamera/ESPAsyncWebServer` fork with the original **`me-no-dev/ESPAsyncWebServer`** to fix a **`LinkedList` template parameter mismatch** conflict

---

## How to Connect

1. Plug your ESP32-S3 into your PC via USB
2. Open **Arduino IDE** → **Tools** → **Serial Monitor**
3. Set baud rate to **115200**
4. Set line ending to **Newline**
5. You should see the `>` prompt — you're ready

> All commands are typed at the `>` prompt and sent by pressing **Enter**

---

## Basic Workflow (Start Here)

```
scanap              ← find nearby WiFi networks
```
Wait 5-10 seconds while it scans, then:
```
stopscan            ← stop scanning
list -a             ← show what was found
select -a 0         ← select network at index 0
attack -t deauth    ← run deauth attack on selected network
stopscan            ← stop the attack
```

---

## WiFi Scanning Commands

| Command | What it does |
|---|---|
| `scanap` | Scan for nearby WiFi access points (routers) |
| `scanall` | Scan for APs and stations at the same time |
| `stopscan` | Stop any running scan or attack |
| `sniffbeacon` | Capture beacon frames from nearby APs |
| `sniffprobe` | Capture probe requests from nearby devices |
| `sniffdeauth` | Capture deauth packets in the air |
| `sniffraw` | Capture all raw WiFi packets |
| `sniffpmkid` | Sniff for PMKID handshakes |
| `sniffpmkid -d` | Sniff PMKID + send deauths to trigger handshake |
| `sniffsae` | Sniff SAE commit frames (WPA3) |
| `sniffpwn` | Sniff for Pwnagotchi devices |
| `wardrive` | Wardriving mode (logs APs while moving) |
| `packetcount` | Count WiFi packets in real time |
| `sigmon` | Monitor signal strength of selected AP |

---

## WiFi Attack Commands

> **Only use on networks you own or have permission to test**

All attacks use the format: `attack -t <type>`

| Command | What it does |
|---|---|
| `attack -t deauth` | Disconnect all devices from selected AP (broadcast) |
| `attack -t deauth -d <mac>` | Deauth a specific device MAC address |
| `attack -t beacon -r` | Spam random fake WiFi network names |
| `attack -t beacon -a` | Spam beacon frames cloned from your scanned AP list |
| `attack -t beacon -l` | Spam beacons from your custom SSID list |
| `attack -t probe` | Send fake probe requests |
| `attack -t rickroll` | Spam WiFi names that are Rick Astley lyrics |
| `attack -t csa` | Channel Switch Announcement attack |
| `attack -t sae` | SAE (WPA3) attack |
| `attack -t badmsg` | Send malformed 802.11 frames |
| `attack -t sleep` | Send sleep frames |
| `attack -t quiet` | Send quiet action frames |

---

## List & Select Commands

Use these to manage what you've scanned and what to target.

### Listing
| Command | What it shows |
|---|---|
| `list -a` | List scanned Access Points |
| `list -s` | List your custom SSID list |
| `list -c` | List stations (clients) found on APs |
| `list -t` | List detected AirTags |
| `list -i` | List IP addresses found |
| `list -p` | List probe requests captured |

### Selecting targets
| Command | What it does |
|---|---|
| `select -a 0` | Select AP at index 0 |
| `select -a 0,1,2` | Select multiple APs (comma separated) |
| `select -a all` | Toggle select/deselect all APs |
| `select -s 0` | Select SSID at index 0 |
| `select -c 0` | Select station (client) at index 0 |
| `select -a -f "contains Google"` | Select APs whose name contains "Google" |
| `select -a -f "equals MyWiFi"` | Select AP whose name exactly matches |

### Info
| Command | What it does |
|---|---|
| `info` | Show device info (firmware version, board, memory) |
| `info -a 0` | Show detailed info about AP at index 0 |

---

## SSID Management

Build a custom list of SSIDs to use in beacon attacks.

| Command | What it does |
|---|---|
| `ssid -a -n "MyFakeWiFi"` | Add a custom SSID name to your list |
| `ssid -a -g 10` | Generate 10 random SSIDs |
| `ssid -r 0` | Remove SSID at index 0 |
| `list -s` | Show your SSID list |

---

## Network Tools

Must be connected to a network first (use `join`).

| Command | What it does |
|---|---|
| `join -a 0 -p mypassword` | Connect to AP at index 0 with password |
| `join -a 0 -s` | Connect to open (no password) AP at index 0 |
| `pingscan` | Ping scan the local network |
| `portscan -a -t 0` | Port scan IP at index 0 |
| `portscan -s http` | Scan for specific service (http/ssh/telnet/etc) |
| `arpscan` | ARP scan local network to find devices |

---

## MAC Address Commands

| Command | What it does |
|---|---|
| `randapmac` | Set a random MAC address for AP mode |
| `randstamac` | Set a random MAC address for station mode |
| `cloneapmac -a 0` | Clone the MAC of AP at index 0 |
| `clonestamac -s 0` | Clone the MAC of station at index 0 |

---

## Bluetooth Commands

| Command | What it does |
|---|---|
| `sniffbt` | Scan for all nearby Bluetooth devices |
| `sniffbt -t airtag` | Scan specifically for Apple AirTags |
| `sniffbt -t flipper` | Scan for Flipper Zero devices |
| `sniffbt -t flock` | Scan for Flock tracker devices |
| `sniffbt -t meta` | Scan for Meta/Ray-Ban glasses |
| `blespam -t apple` | Spam fake Apple device popups to nearby iPhones |
| `blespam -t google` | Spam fake Google device notifications |
| `blespam -t samsung` | Spam fake Samsung pairing popups |
| `blespam -t windows` | Spam fake Windows Swift Pair popups |
| `blespam -t flipper` | Spam fake Flipper Zero BLE signals |
| `blespam -t all` | Spam all of the above at once |
| `spoofat -t 0` | Spoof AirTag at index 0 |
| `sniffskim` | Sniff for Bluetooth credit card skimmers |
| `btwardrive` | Wardriving mode for Bluetooth |
| `stopscan` | Stop any BLE scan or spam |

---

## Evil Portal

Creates a fake WiFi hotspot that shows a fake login page to connected users.

| Command | What it does |
|---|---|
| `evilportal -c start` | Start evil portal using default HTML |
| `evilportal -c start -w login.html` | Start with a specific HTML file (needs SD card) |
| `evilportal sethtml login.html` | Set which HTML file to use |
| `karma -p 0` | Start Karma attack using probe at index 0 |

---

## Save & Load (requires SD card)

| Command | What it does |
|---|---|
| `save -a` | Save AP list to SD card |
| `save -s` | Save SSID list to SD card |
| `load -a` | Load AP list from SD card |
| `load -s` | Load SSID list from SD card |
| `ls /` | List files in root of SD card |
| `ls /pcaps` | List captured pcap files |

---

## WiFi Channel Control

| Command | What it does |
|---|---|
| `channel` | Show current channel |
| `channel -s 6` | Set WiFi channel to 6 |

---

## System Commands

| Command | What it does |
|---|---|
| `help` | Print all available commands |
| `reboot` | Restart the ESP32 |
| `settings` | Show current settings |
| `settings -s <name> enable` | Enable a setting |
| `settings -s <name> disable` | Disable a setting |
| `settings -r` | Reset settings to default |
| `clearlist -a` | Clear the AP list |
| `clearlist -s` | Clear the SSID list |
| `clearlist -c` | Clear the station list |

---

## Common Error Messages

| Message | What it means | Fix |
|---|---|---|
| `Failed to set AP MAC: 0x3005` | Cannot spoof MAC (normal on S3) | Ignore — scanning still works |
| `0 selected` | No network selected | Run `select -a 0` first |
| `You don't have any targets selected` | Need to select before attacking | Run `select -a <index>` |
| `Index not in range` | Index number doesn't exist | Run `list -a` to check valid indexes |

---

## Typical Usage Examples

### Test your own WiFi (deauth)
```
scanap
stopscan
list -a
select -a 0
attack -t deauth
stopscan
```

### Spam fake WiFi names
```
attack -t beacon -r
stopscan
```

### Find Bluetooth devices nearby
```
sniffbt
stopscan
list -t
```

### Detect AirTags near you
```
sniffbt -t airtag
stopscan
list -t
```

### Spam iPhone popups to test Apple devices
```
blespam -t apple
stopscan
```

---

## What Marauder CANNOT Do

- ❌ Crack or reveal WiFi passwords
- ❌ Intercept encrypted data
- ❌ Hack into accounts
- ❌ Access anyone's files or messages

---

## Legal Warning

> Use only on networks and devices **you own** or have **written permission** to test.
> Using these tools against others without permission is **illegal** in most countries.
