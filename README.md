<p align="center">
  <a href="https://github.com/babeinlovexd"><img src="https://raw.githubusercontent.com/babeinlovexd/babeinlovexd/main/bxd-github-header.svg" width="520" alt="BxD / BabeinlovexD"></a>
</p>

# ESPHome R503

[Issues](https://github.com/babeinlovexd/ESPHOME_r503/issues)

An external ESPHome component for the **GROW R503 fingerprint sensor**, based on ESPHome's `fingerprint_grow` implementation and extended with R503 commands.

## Features

- Fingerprint enrollment, matching and deletion.
- Aura LED ring control and configurable security level.
- Read/write access to the 512-byte notepad and template index table.
- Sensor checks, soft reset, cancellation and extended image collection.
- Sensors for fingerprint count, capacity, status, security level, last ID and confidence.

## Installation

Add the component to an existing ESPHome device configuration:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/babeinlovexd/ESPHOME_r503
      ref: main
    components: [r503_fingerprint]

uart:
  tx_pin: GPIO17
  rx_pin: GPIO16
  baud_rate: 57600

r503_fingerprint:
  id: my_fingerprint
  sensing_pin: GPIO18
  on_finger_scan_matched:
    - logger.log:
        format: "Finger matched: ID=%d, confidence=%d"
        args: [finger_id, confidence]
  on_finger_scan_unmatched:
    - logger.log: "Finger not matched"

sensor:
  - platform: r503_fingerprint
    fingerprint_count:
      name: "Fingerprint Count"
    last_finger_id:
      name: "Last Finger ID"
```

GPIO assignments are an example for an ESP32, not a universal wiring map. Connect controller TX to sensor RX and controller RX to sensor TX, and follow the module's supply and logic-level requirements. `sensing_pin` is optional.

## Configuration & actions

The full [configuration reference](docu.md) documents callbacks, enrollment/deletion, LED control, notepad and diagnostic commands. The repository also includes [test.yaml](test.yaml) and the [R503 datasheet](R503_datasheet.pdf).

If several sensors share the configuration, assign each sensor platform's `r503_fingerprint_id` to the relevant component. `sensor_power_pin` requires `sensing_pin`; `idle_period_to_sleep` additionally requires `sensor_power_pin`.

## Troubleshooting

- **No response:** Check UART RX/TX direction, baud rate, common ground and power supply.
- **Enrollment fails:** Read the enrollment callbacks and sensor capacity before retrying.
- **Unknown sensor password:** Match the configured password to the module. `new_password` changes the sensor password; do not set it casually or lose the new value.
- **LED state differs from expectations:** Check the supported Aura LED state, color, speed and count in the reference.

Fingerprint recognition provides an event for your own automation. It does not, by itself, implement a complete door-lock system.

---

## Builds und Releases

Version: **0.1.0** ([VERSION](VERSION)). [GitHub Actions](.github/workflows/release.yml) prüft Pushes und Pull Requests. Ein Tag passend zu `v` + `VERSION` veröffentlicht das Release erst nach erfolgreichen Jobs; manuelle Läufe liefern Build-Artefakte. [Downloads](https://github.com/babeinlovexd/ESPHOME_r503/releases).

ESPHome-CI verwendet Platzhalter-Zugangsdaten und flasht keine Geräte. Vor dem Einsatz eigene Secrets und Pinbelegung prüfen.

## Contributing

Questions, bug reports and contributions are welcome. Please include the version, setup and steps to reproduce in an [issue](https://github.com/babeinlovexd/ESPHOME_r503/issues). Discuss larger changes before submitting a pull request.

[BxD / BabeinlovexD](https://github.com/babeinlovexd) · [Website](https://babeinlovexd.de/)
