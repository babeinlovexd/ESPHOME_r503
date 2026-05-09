# R503 Fingerprint Sensor for ESPHome

This is an external component for ESPHome that adds support for the GROW R503 fingerprint sensor. It is based on the built-in `fingerprint_grow` component but includes several additional commands specific to the R503 sensor module, allowing you to access extended functionalities documented in the R503 datasheet.

## Features

- **Fingerprint Enrollment & Matching**: Standard fingerprint scanning, matching, and enrollment.
- **Aura LED Control**: Full control over the built-in LED ring (colors, states, speed, and count).
- **Security Level Control**: Dynamically adjust the matching threshold (Security Level 1-5).
- **Notepad Storage**: Read from and write to the internal 512-byte flash notepad (16 pages x 32 bytes).
- **Index Table**: Read the fingerprint template index table to see which slots are occupied.
- **Extended Commands**:
  - `GetImageEx`: Advanced image collection.
  - `Cancel`: Cancel the current ongoing instruction.
  - `Soft Reset`: Reboot the sensor module over UART.
  - `Check Sensor`: Query the health status of the sensor.

## Installation

You can include this external component in your ESPHome configuration directly from the Git repository's main branch.

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/babeinlovexd/ESPHOME_r503
      ref: main
    components: [ r503_fingerprint ]
```

## Basic Configuration Example

```yaml
uart:
  tx_pin: GPIO17
  rx_pin: GPIO16
  baud_rate: 57600

r503_fingerprint:
  id: my_fingerprint
  sensing_pin: GPIO18 # Optional: Wakes up the ESP when touched
  on_finger_scan_matched:
    - logger.log:
        format: "Finger matched! ID: %d, Confidence: %d"
        args: [finger_id, confidence]
  on_finger_scan_unmatched:
    - logger.log: "Finger not matched!"

sensor:
  - platform: r503_fingerprint
    fingerprint_count:
      name: "Fingerprint Count"
    last_finger_id:
      name: "Last Finger ID"

```

For detailed information on all available configuration options, actions, and sensors, please refer to [docu.md](docu.md).