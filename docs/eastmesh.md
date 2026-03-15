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

## What You Need

- a Heltec V3 or Heltec V4 board
- a USB data cable
- WiFi credentials for the network the device will use
- this repo checked out locally

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

## Step 2: Install Repo Dependencies

From the repo root:

```bash
poetry install
```

This installs the Python build tooling used by MeshCore, including PlatformIO.

## Step 3: Plug In the Device

Connect the Heltec board over USB.

If the board does not automatically enter upload mode during flashing, manually put it into the ESP32 bootloader:

1. Hold `BOOT`
2. Tap `RESET`
3. Release `BOOT` when upload starts

If normal upload works for your board, you do not need to do this manually.

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

## Built-In EastMesh Defaults

Both observer builds include these EastMesh MQTT defaults:

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

## Known Issue: First Flash Does Not Always Apply MQTT Defaults

There is a current bug where the `platformio.ini` MQTT values for Heltec V3 and V4 do not always apply correctly on the first flash.

Because of that, after flashing you should always connect to the device console and verify the live values with `get` commands.

If the values are missing or wrong, set them manually and reboot.

## Step 6: Open the Device Console

Use either:

- serial console at `115200` baud, or
- the repeater console from another MeshCore device

The commands below work from either interface unless otherwise noted.

## Step 7: Configure WiFi

Set your WiFi credentials:

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

## Step 8: Verify or Repair MQTT Settings

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

## Step 9: Optional Analyzer Settings

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

## Step 10: Reboot

After changing WiFi or MQTT settings, reboot the device:

```text
reboot
```

## Step 11: Confirm It Connected

Expected connection logs include:

- `MQTT client connected, session present: ...`
- `Broker 0 marked as connected`

If you enabled analyzer brokers, you may also see additional broker connection messages.

Observer visibility can be confirmed on:

- `https://obs.eastmesh.au`

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
