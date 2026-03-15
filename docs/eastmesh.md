# EastMesh MQTT Setup (Heltec V3/V4)

Known-good configuration for:

- `Heltec_v3_repeater_observer_mqtt`
- `heltec_v4_repeater_observer_mqtt`

## Build Defaults

Configured in:

- `variants/heltec_v3/platformio.ini`
- `variants/heltec_v4/platformio.ini`

Shared MQTT build flags used by both environments:

- `MQTT_SERVER="wss://mqtt2.eastmesh.au"`
- `MQTT_PORT=443`
- `MQTT_PASSWORD="jwt"` (enables JWT auth mode)
- `MQTT_AUDIENCE="mqtt2.eastmesh.au"`
- `MQTT_USE_WEBSOCKETS=1`
- `MQTT_USE_TLS=1`
- `MQTT_CUSTOM_CA_ISRG_ROOT_X1=1`

Notes:

- `MQTT_DEBUG`, `MESH_PACKET_LOGGING`, and `MESH_DEBUG` are disabled by default for cleaner production logs.
- V3 also sets `ESP32_CPU_FREQ=160` in `Heltec_v3_repeater_observer_mqtt`.
- JWT signature conversion to standard base64url was removed; custom broker auth works with the original token signature format used by `JWTHelper`.

## Flash

Find PORT

```bash
poetry run pio device list
```

Flash V3

```bash
poetry run pio run -e Heltec_v3_repeater_observer_mqtt -t upload --upload-port <PORT>
```

Flash V4

```bash
poetry run pio run -e heltec_v4_repeater_observer_mqtt -t upload --upload-port <PORT>
```

## Runtime Commands

Use these from the CLI if needed:

```text
set mqtt.server wss://mqtt2.eastmesh.au:443
set mqtt.port 443
set mqtt.password jwt
```

NOTE: Do **NOT** set mqtt.username!

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

Strongly recommend disabling one of the LetsMesh analyzer servers. Keeping EastMesh plus both LetsMesh analyzers enabled at the same time uses noticeably more memory.

```text
set mqtt.analyzer.us off
```

## Verify

Expected connection logs include:

- `MQTT client connected, session present: ...`
- `Broker 0 marked as connected`

And observer visibility confirmed on:

- `https://obs.eastmesh.au` (user `mqtt`, password `eastmesh`)
