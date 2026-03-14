# EastMesh MQTT Setup (Heltec V4)

Known-good configurations for:
`heltec_v4_repeater_observer_mqtt`
`Xiao_S3_WIO_repeater_observer_mqtt`

## Build Defaults

Configured in:
`variants/heltec_v4/platformio.ini`
`variants/Xiao_S3_WIO_repeater_observer_mqtt/platformio.ini`

- `MQTT_SERVER="wss://mqtt2.eastmesh.au:443"`
- `MQTT_PORT=443`
- `MQTT_PASSWORD="jwt"` (enables JWT auth mode)
- `MQTT_AUDIENCE="mqtt2.eastmesh.au"`
- `MQTT_USE_WEBSOCKETS=1`
- `MQTT_USE_TLS=1`
- `MQTT_CUSTOM_CA_ISRG_ROOT_X1=1`

Notes:

- JWT signature conversion to standard base64url was removed; custom broker auth works with the original token signature format used by `JWTHelper`.
- `MQTT_DEBUG`, `MESH_PACKET_LOGGING`, and `MESH_DEBUG` are disabled by default for cleaner production logs.

## Flash

```bash
poetry run pio run -e heltec_v4_repeater_observer_mqtt -t upload --upload-port <PORT>
```
or
```bash
poetry run pio run -e Xiao_S3_WIO_repeater_observer_mqtt -t upload --upload-port <PORT>
```
To just build the firmware you can also run
```bash
./build.sh build-firmware <target>
```
A list of targets can be found with
```bash
./build.sh -l
```


## Runtime Commands

Configure your WiFi!

```text
set wifi.ssid YourWiFiNetwork
set wifi.pwd YourWiFiPassword
```

Use these from CLI if needed:

```text
set mqtt.server wss://mqtt2.eastmesh.au:443
set mqtt.port 443
set mqtt.password jwt
```

'reboot' to restart with the new settings

Optional analyzer servers:

```text
set mqtt.analyzer.us on
set mqtt.analyzer.eu on
```

or disable:

```text
set mqtt.analyzer.us off
set mqtt.analyzer.eu off
```

## Verify

Expected connection logs include:

- `MQTT client connected, session present: ...`
- `Broker 0 marked as connected`

And observer visibility confirmed on:

- `https://obs.eastmesh.au` (user `mqtt`, password `eastmesh`)
