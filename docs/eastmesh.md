# EastMesh MQTT Setup (Heltec V3/V4)

Beginner-friendly setup guide for the EastMesh MQTT observer firmware on:

- `Heltec_v3_repeater_observer_mqtt`
- `heltec_v4_repeater_observer_mqtt`

This guide covers:

- installing the Python tooling
- building and flashing the firmware
- configuring WiFi and MQTT
- verifying that the device is connected
- working around a known first-flash config issue

For broader MQTT bridge behavior and command reference, see [MQTT Bridge Implementation](./MQTT_IMPLEMENTATION.md).

## What You Need

- a Heltec V3 or Heltec V4 board
- a USB data cable
- WiFi credentials for the network the device will use
- this repo checked out locally

## Scope

This guide is written for Heltec V3 and Heltec V4 because those are the known working EastMesh setups.

All preconfigured `*_repeater_observer_mqtt` PlatformIO environments now include the EastMesh MQTT defaults.

Only Heltec V3 and Heltec V4 have been fully tested with EastMesh so far.

The same overall process should also work on the other boards listed below, but those EastMesh configurations have not been fully validated yet.

Preconfigured observer-MQTT environments currently exist for:

- `Heltec_v3_repeater_observer_mqtt`
- `heltec_v4_repeater_observer_mqtt`
- `Heltec_T190_repeater_observer_mqtt`
- `LilyGo_T3S3_sx1262_repeater_observer_mqtt`
- `Tbeam_SX1262_repeater_observer_mqtt`
- `Tbeam_SX1276_repeater_observer_mqtt`
- `T_Beam_S3_Supreme_SX1262_repeater_observer_mqtt`
- `Station_G2_repeater_observer_mqtt`
- `Xiao_S3_WIO_repeater_observer_mqtt`

CI note:

- the current PR build matrix covers standard repeater builds on several platforms
- the observer-MQTT environments are not currently built in CI

## Platform Notes

### macOS / Linux

You can use either:

- `poetry run pio ...` directly, or
- `build.sh`

`build.sh` is a shell script and is intended for macOS/Linux shells.

### Windows

Use `poetry run pio ...` directly.

Do **not** use `build.sh` on Windows. It is a Bash shell script and is not the supported path there.

## Step 1: Install Python and Poetry

### macOS / Linux

Install both with Homebrew:

```bash
brew install python poetry
```

Verify them:

```bash
python3 --version
poetry --version
```

### Windows

Install Python 3 first, then verify it:

```powershell
py --version
```

Install `pipx`:

```powershell
py -m pip install --user pipx
py -m pipx ensurepath
```

Close and reopen PowerShell or Command Prompt after `ensurepath`.

Install Poetry with `pipx`:

```powershell
pipx install poetry
```

Verify it:

```powershell
poetry --version
```

## Step 2: Clone This Repo and Install Dependencies

If you do not already have a local copy of this repo, clone it first.

The simplest path is to install Git and clone the repo locally.

If you expect to do more custom builds and flashing, it is also worth setting this up in VS Code with PlatformIO, but that is not required for this guide.

Example Git workflow:

```bash
git clone https://github.com/xJARiD/MeshCore.git
cd MeshCore
poetry install
```

If you already cloned the repo, just open a terminal in the repo root and run:

```bash
poetry install
```

This installs the Python build tooling used by MeshCore, including PlatformIO.

On Windows, one simple way to open a terminal in the repo root is:

1. Open the local repo folder in Explorer
2. Shift + right-click in the folder
3. Select `Open in Terminal`

## Step 3: Plug In the Device

Connect the Heltec board over USB.

You can either:

- hold `BOOT` while connecting the board to USB, or
- connect it normally and only use `BOOT` + `RESET` if upload does not start

If the board does not automatically enter upload mode during flashing, manually put it into the ESP32 bootloader:

1. Hold `BOOT`
2. Tap `RESET`
3. Release `BOOT` when upload starts

If normal upload works for your board, you do not need to do this manually.

Note:

- if the board enters upload mode after you first connected it, the serial port may change
- if upload fails, run `poetry run pio device list` again and confirm the current port before retrying

## Step 4: Find the Serial Port

List connected serial devices:

```bash
poetry run pio device list
```

Typical port names:

- macOS: `/dev/cu.usbmodem...` or `/dev/cu.usbserial...`
- Linux: `/dev/ttyACM0` or `/dev/ttyUSB0`
- Windows: `COM3`, `COM4`, `COM5`, etc.

## Step 5: Flash the Firmware

### Heltec V3

```bash
poetry run pio run -e Heltec_v3_repeater_observer_mqtt -t upload --upload-port <PORT>
```

### Heltec V4

```bash
poetry run pio run -e heltec_v4_repeater_observer_mqtt -t upload --upload-port <PORT>
```

Replace `<PORT>` with the value from `poetry run pio device list`.

## Optional: Build `.bin` Files for Web Flasher

This is mainly for macOS/Linux users using `build.sh`.

Set a firmware version first:

```bash
export FIRMWARE_VERSION=1.14.0 #or whatever the latest firmware version is
```

Then build the observer-MQTT target you want:

```bash
sh build.sh build-firmware Heltec_v3_repeater_observer_mqtt
```

or:

```bash
sh build.sh build-firmware heltec_v4_repeater_observer_mqtt
```

You can also substitute any other preconfigured `*_repeater_observer_mqtt` environment listed earlier in this guide.

The generated files will appear in the `out` folder.

For ESP32 targets, `build.sh` produces:

- `<env>-<version>-<sha>.bin`
- `<env>-<version>-<sha>-merged.bin`

You can then open:

- `https://flasher.meshcore.co.uk/`
- `https://flasher.meshcoreaus.org/`

and flash using `Custom firmware`.

Use:

- the normal `.bin` file for a standard flash
- the `-merged.bin` file when doing an erase + flash

## Built-In EastMesh Defaults

All preconfigured observer-MQTT builds include these EastMesh MQTT defaults:

- `MQTT_SERVER="wss://mqtt2.eastmesh.au"`
- `MQTT_PORT=443`
- `MQTT_PASSWORD="jwt"`
- `MQTT_AUDIENCE="mqtt2.eastmesh.au"`
- `MQTT_USE_WEBSOCKETS=1`
- `MQTT_USE_TLS=1`
- `MQTT_CUSTOM_CA_ISRG_ROOT_X1=1`

Additional notes:

- V3 also sets `ESP32_CPU_FREQ=160`
- `MQTT_DEBUG`, `MESH_PACKET_LOGGING`, and `MESH_DEBUG` are disabled by default
- do **not** set `mqtt.username` for EastMesh

## Step 6: Configure the Repeater Basics

After flashing, do the normal repeater setup first.

Using WebFlasher:

- `https://flasher.meshcore.co.uk/`
- `https://flasher.meshcoreaus.org/`

Use `Configure via USB`, or open `config.meshcore.dev` directly in a Chromium-based browser.

Then:

1. Connect to the node and select the correct serial port
2. Set the node name
3. Set the admin password
4. Select preset `Australia Narrow`
5. Save the configuration

Recommended:

- keep a companion node nearby and powered on
- after saving, use `advert` so your companion picks up the new MQTT node in contacts
- that makes it easier to remote-administer the node later

## Step 7: Open the Device Console

Use either:

- serial console at `115200` baud, or
- the repeater console from another MeshCore device
- the `Console` tab in WebFlasher, using the correct serial port

The commands below work from either interface unless otherwise noted.

For the general console flow, see the quick start section in [MQTT Bridge Implementation](./MQTT_IMPLEMENTATION.md#quick-start-guide).

## Step 8: Configure WiFi

From the console, set your WiFi credentials by entering each of these commands:

```text
set wifi.ssid YourWiFiNetwork
set wifi.pwd YourWiFiPassword
set wifi.powersave min
```

Verify them:

```text
get wifi.ssid
get wifi.pwd
get wifi.powersave
```

If you want better WiFi performance and do not care about power saving, try:

```text
set wifi.powersave none
```

Additional WiFi command reference is documented in [MQTT Bridge Implementation](./MQTT_IMPLEMENTATION.md#step-3-configure-wifi-connection).

## Known Issue: First Flash Does Not Always Apply MQTT Defaults

There is a current bug where the `platformio.ini` MQTT values for Heltec V3 and V4 do not always apply correctly on the first flash.

Because of that, after WiFi is configured you should always verify the live MQTT values with `get` commands.

If the values are missing or wrong, set them manually and reboot.

## Step 9: Verify or Repair MQTT Settings

Check the live MQTT settings:

```text
get mqtt.server
get mqtt.port
get mqtt.password
get mqtt.analyzer.us
get mqtt.analyzer.eu
get mqtt.config.valid
```

For EastMesh, the expected values are:

- `mqtt.server` -> `wss://mqtt2.eastmesh.au:443`
- `mqtt.port` -> `443`
- `mqtt.password` -> `jwt`

If any of those are wrong or blank, set them manually:

```text
set mqtt.server wss://mqtt2.eastmesh.au:443
set mqtt.port 443
set mqtt.password jwt
```

Do **not** set `mqtt.username`.

Then verify again:

```text
get mqtt.server
get mqtt.port
get mqtt.password
get mqtt.config.valid
```

For the broader MQTT command set, see [MQTT Bridge Implementation](./MQTT_IMPLEMENTATION.md#step-6-configure-mqtt-settings).

## Step 10: Optional Analyzer Settings

You can also publish to the Let's Mesh analyzers:

```text
set mqtt.analyzer.us on
set mqtt.analyzer.eu on
```

Or disable them:

```text
set mqtt.analyzer.us off
set mqtt.analyzer.eu off
```

Strongly recommend disabling one of the LetsMesh analyzer servers. Keeping EastMesh plus both LetsMesh analyzers enabled at the same time uses noticeably more memory.

For example:

```text
set mqtt.analyzer.us off
```

Visibility note:

- if your repeater does not have nearby neighbours, or there is no other MQTT-connected repeater nearby to hear and forward your packets, switch the bridge source to TX so the repeater uplinks its own transmitted adverts
- this helps isolated repeaters show up on both Let's Mesh and EastMesh

Commands:

```text
set bridge.source tx
set mqtt.tx on
reboot
```

If you do have nearby mesh neighbours or another MQTT-connected repeater hearing you reliably, keep the default RX-oriented setup to avoid uplinking your own transmitted packets unnecessarily.

Background on the Let's Mesh analyzer integration is in [MQTT Bridge Implementation](./MQTT_IMPLEMENTATION.md#lets-mesh-analyzer-integration).

## Step 11: Reboot

After changing WiFi or MQTT settings, reboot the device:

```text
reboot
```

Once rebooted, it can be worth re-running the `get` checks from the WiFi and MQTT steps above.

This is not strictly required, but it is useful as a sanity check.

## Step 12: Confirm It Connected

Expected connection logs include:

- `MQTT client connected, session present: ...`
- `Broker 0 marked as connected`

If you enabled analyzer brokers, you may also see additional broker connection messages.

Observer visibility can be confirmed on:

- `https://obs.eastmesh.au`

If the setup was successful, you should see your new node appear in the list of observers on EastMesh.

Current observer credentials:

- user: `mqtt`
- password: `eastmesh`

## Quick Recovery Checklist

If the device flashes successfully but does not connect:

1. Confirm WiFi settings with `get wifi.ssid`, `get wifi.pwd`, and `get wifi.powersave`
2. Confirm MQTT settings with `get mqtt.server`, `get mqtt.port`, and `get mqtt.password`
3. Confirm validity with `get mqtt.config.valid`
4. Re-apply the EastMesh values manually if needed
5. Run `reboot`

For additional troubleshooting context, see [MQTT Bridge Implementation](./MQTT_IMPLEMENTATION.md).

## Unofficial References

Additional community-maintained documentation is also available at these unofficial wikis:

### ACT/NSW/QLD/SA/TAS/VIC

- `https://wiki.eastmesh.au/`
- `https://wiki.meshcoreaus.org/`

### Sydney

- `https://nswmesh.au/`
- `https://github.com/nswmesh/`

### Brisbane

- `https://wiki.mbug.com.au/en/Meshcore/Settings`
