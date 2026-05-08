# R503 Fingerprint Component Documentation

This document describes all configuration variables, actions, and sensors available for the `r503_fingerprint` custom ESPHome component.

## Base Configuration

```yaml
r503_fingerprint:
  id: my_fingerprint
  password: 0 # Optional: The module's password (default is 0)
  sensing_pin: GPIOX # Optional: Pin connected to the touch sensor wire (WAKE)
  sensor_power_pin: GPIOY # Optional: Pin to control power to the sensor
  idle_period_to_sleep: 5s # Optional: Time before putting the sensor to sleep

  # Callbacks
  on_finger_scan_matched:
    - logger.log: "Matched!"
  on_finger_scan_unmatched:
    - logger.log: "Unmatched!"
  # ... (see below)
```

### Configuration Variables

* **id** (*Optional*, ID): The ID of the component, used for actions and sensors.
* **password** (*Optional*, int): The 32-bit password to communicate with the module. Default: `0`.
* **sensing_pin** (*Optional*, Pin): The pin connected to the `WAKE` line of the sensor. The R503 pulls this high when touched.
* **sensor_power_pin** (*Optional*, Pin): A pin that provides power to the sensor (for deep sleep scenarios).
* **idle_period_to_sleep** (*Optional*, Time): If `sensor_power_pin` is used, the time of inactivity before cutting power.
* **new_password** (*Optional*, int): If provided, the sensor's password will be changed to this value on boot. **Warning:** If you forget this password, the sensor cannot be accessed!

### Callbacks & Automations

You can trigger automations based on sensor events:

* **on_finger_scan_start**: Triggered when a finger is placed on the sensor and scanning begins.
* **on_finger_scan_matched**: Triggered when a scanned finger matches a saved template.
  * Variables: `finger_id` (int), `confidence` (int).
* **on_finger_scan_unmatched**: Triggered when a scanned finger does not match any saved template.
* **on_finger_scan_misplaced**: Triggered when the scan fails because the finger is not placed correctly.
* **on_finger_scan_invalid**: Triggered when the image read is invalid.
* **on_enrollment_scan**: Triggered during the enrollment process when a scan is taken.
  * Variables: `scan_num` (int), `finger_id` (int).
* **on_enrollment_done**: Triggered when enrollment completes successfully.
  * Variables: `finger_id` (int).
* **on_enrollment_failed**: Triggered when enrollment fails.
  * Variables: `finger_id` (int).
* **on_notepad_read**: Triggered when `r503_fingerprint.read_notepad` is called and data is successfully returned.
  * Variables: `page` (int), `text` (string).

---

## Sensors and Binary Sensors

### Sensors

```yaml
sensor:
  - platform: r503_fingerprint
    fingerprint_count:
      name: "Enrolled Fingerprints"
    status:
      name: "Sensor Status"
    capacity:
      name: "Sensor Capacity"
    security_level:
      name: "Security Level"
    last_finger_id:
      name: "Last Matched ID"
    last_confidence:
      name: "Last Confidence"
```

* **fingerprint_count**: The number of enrolled fingerprint templates.
* **status**: The raw status register of the module.
* **capacity**: The maximum number of fingerprints the module can store.
* **security_level**: The current matching threshold (1-5).
* **last_finger_id**: The ID of the last successfully matched fingerprint.
* **last_confidence**: The matching confidence score of the last matched fingerprint.

### Binary Sensors

```yaml
binary_sensor:
  - platform: r503_fingerprint
    enrolling:
      name: "Sensor is Enrolling"
```

* **enrolling**: Turns ON while the sensor is actively in enrollment mode.

---

## Actions

The component provides several actions to control the sensor from ESPHome automations.

### `r503_fingerprint.enroll`

Start the fingerprint enrollment process.

```yaml
on_...:
  then:
    - r503_fingerprint.enroll:
        id: my_fingerprint
        finger_id: 12
        num_scans: 2 # Default is 2. The R503 generally requires 2 to 6 scans.
```

### `r503_fingerprint.cancel_enroll`

Cancel an active enrollment process.

```yaml
on_...:
  then:
    - r503_fingerprint.cancel_enroll:
        id: my_fingerprint
```

### `r503_fingerprint.delete` / `delete_all`

Delete a specific fingerprint or wipe the entire database.

```yaml
on_...:
  then:
    - r503_fingerprint.delete:
        id: my_fingerprint
        finger_id: 12

    - r503_fingerprint.delete_all:
        id: my_fingerprint
```

### `r503_fingerprint.aura_led_control`

Control the RGB LED ring around the R503 sensor.

```yaml
on_...:
  then:
    - r503_fingerprint.aura_led_control:
        id: my_fingerprint
        state: BREATHING # ALWAYS_ON, ALWAYS_OFF, FLASHING, BREATHING, GRADUAL_ON, GRADUAL_OFF
        speed: 100       # 0 to 255. Lower is faster.
        color: BLUE      # RED, BLUE, PURPLE, GREEN, YELLOW, CYAN, WHITE
        count: 5         # Number of flashes/breaths (0-255). 0 = infinite.
```

### `r503_fingerprint.set_security_level`

Set the matching threshold security level (1-5). Level 1 has the highest False Acceptance Rate (FAR) and lowest False Rejection Rate (FRR). Level 5 has the lowest FAR and highest FRR. Default is usually 3.

```yaml
on_...:
  then:
    - r503_fingerprint.set_security_level:
        id: my_fingerprint
        level: 4
```

### `r503_fingerprint.write_notepad` / `read_notepad`

The module has 16 pages (0-15) of 32 bytes each for user data storage that persists across reboots.

```yaml
on_...:
  then:
    - r503_fingerprint.write_notepad:
        id: my_fingerprint
        page: 0
        text: "Hello World" # Maximum 32 characters

    - r503_fingerprint.read_notepad:
        id: my_fingerprint
        page: 0
```
*(The read data will be returned via the `on_notepad_read` callback on the main component).*

### Additional Hardware Commands

```yaml
on_...:
  then:
    # Read the template index table (pages 0-3). Output goes to the ESPHome logger.
    - r503_fingerprint.read_index_table:
        id: my_fingerprint
        page: 0

    # Execute GetImageEx (advanced image collection command)
    - r503_fingerprint.get_image_ex:
        id: my_fingerprint

    # Cancel any ongoing hardware instruction
    - r503_fingerprint.cancel:
        id: my_fingerprint

    # Query sensor health
    - r503_fingerprint.check_sensor:
        id: my_fingerprint

    # Perform a soft reboot of the module
    - r503_fingerprint.soft_reset:
        id: my_fingerprint
```