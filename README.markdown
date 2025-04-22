# ESPHome Configuration for Centurion D5 Evo Gate Motor with Sonoff SV

This ESPHome configuration enables remote control and status monitoring of a Centurion D5 Evo gate motor using a Sonoff SV (ESP8266). It integrates with Home Assistant to provide real-time gate status and control. This code is based on the code of wernerhp/esphome.centurion_d5_evo.

## Features

- **Relay Control**: Triggers gate open/stop/close via GPIO12.
- **Button Input**: Monitors a physical button on GPIO0.
- **Status Monitoring**: Detects gate status (e.g., Closed, Opening, Battery Low) via GPIO14 using signal pattern detection.
- **WiFi Status**: Reports signal strength in dB and percentage.
- **Home Assistant Integration**: Publishes gate status and controls via the Home Assistant API.

## Prerequisites

- Sonoff SV (ESP8266-based device)
- Centurion D5 Evo gate motor
- ESPHome (installed and configured with Home Assistant)
- Voltage divider (e.g., 2.2kΩ and 3.3kΩ resistors) for 5V to 3.3V conversion
- Low pass filter (e.g., 2.2µf and 10µf electrolytic capacitors) for a cutoff frequency of 40Hz
- Basic soldering tools and jumper wires

## Hardware Setup

1. **Relay (GPIO12)**:

   - Connect the Sonoff SV’s GPIO12 to the Centurion D5 Evo’s trigger input (consult the motor’s manual for the correct terminal).
   - Ensure the Sonoff SV’s GND is connected to the motor’s GND.

2. **Button (GPIO0)**:

   - Wire a physical push button to GPIO0 and GND on the Sonoff SV.
   - GPIO0 is configured with an internal pull-up resistor.

3. **Status Signal (GPIO5)**:

   - The Centurion D5 Evo’s status output is 5V. Use a voltage divider to step it down to \~3V for the ESP8266:
     - Connect the status output’s positive wire to a 2.2kΩ resistor.
     - Connect the other end of the 2kΩ resistor to GPIO5 and a 3.3kΩ resistor.
     - Connect the other end of the 3.3kΩ resistor to GND.
     - Connect the status output’s negative wire to the Sonoff SV’s GND.
     - Connect a 2.2µf as well as a 10µf electrolytic capacitor in parallel with the 3.3kΩ resistor to create a 40Hz low pass filter.
   - Example: `[Status Output +] --[2kΩ]--[GPIO5]--[3kΩ]--[GND]`

4. **Power**:

   - Power the Sonoff SV with a 5-24V DC supply (per its specifications).
   - Ensure the Centurion D5 Evo and Sonoff SV share a common GND.

## Installation

1. **Clone the Repository**:

   ```bash
   git clone <repository-url>
   ```

2. **Configure ESPHome**:

   - Copy `gate.yaml` to your ESPHome configuration directory.

   - Update secrets (e.g., `wifi_ssid`, `wifi_password`, `api_encryption_key`, `fallback_password`) in your ESPHome `secrets.yaml` file.

   - Example `secrets.yaml`:

     ```yaml
     wifi_ssid: "YourWiFiSSID"
     wifi_password: "YourWiFiPassword"
     api_encryption_key: "YourAPIKey"
     fallback_password: "YourFallbackPassword"
     ```

3. **Upload to Sonoff SV**:

   - Connect the Sonoff SV to your computer via USB (requires a USB-to-serial adapter for flashing).

   - In ESPHome, compile and upload the firmware:

     ```bash
     esphome run gate.yaml
     ```

4. **Add to Home Assistant**:

   - Once the Sonoff SV connects to your WiFi, it will appear in Home Assistant under `Settings > Devices & Services > ESPHome`.
   - Add the device and verify entities (e.g., `Gate`, `Gate Button`, `Gate Status`, `WiFi signal`).

## Usage

- **Control**: Use the `Gate` switch in Home Assistant to trigger open/stop/close (sends a 500ms pulse to the relay).
- **Button**: The `Gate Button` binary sensor detects presses on the physical button connected to GPIO0.
- **Status**: The `Gate Status` text sensor displays the gate’s state (e.g., "Closed", "Opening", "Battery Low") based on the status signal patterns.
- **WiFi**: Monitor `WiFi signal dB` and `WiFi signal` for network performance.

## Status Signal Patterns

The configuration detects the following Centurion D5 Evo status signals on GPIO14:

- **Closed**: Signal OFF for ≥2s
- **Open**: Signal ON for ≥2s
- **Opening**: Slow flash (\~0.5Hz, 1s ON/OFF)
- **Closing**: Fast flash (\~2Hz, 250ms ON/OFF)
- **Pillar Lights On**: 1 flash (200ms) every 2s
- **No Mains**: 2 flashes every 2s
- **Battery Low**: 3 flashes every 2s
- **Multiple Collisions**: 4 flashes every 2s

## Troubleshooting

- **Device Unavailable in Home Assistant**: Check ESPHome logs (`esphome logs gate.yaml`), verify WiFi connectivity, and restart the Sonoff SV and Home Assistant.
- **Status Not Updating**: Ensure the voltage divider is correctly wired and the status signal is reaching GPIO14. Check logs for pattern detection messages (e.g., "Gate is Opening").
- **Relay Not Triggering**: Verify the relay wiring and the Centurion D5 Evo’s trigger input configuration.
- **Signal Noise**: Adjust the `delayed_on_off` filter (currently 50ms) in the `binary_sensor` for GPIO14 if false triggers occur.

## Contributing

Feel free to open issues or submit pull requests for improvements, such as refined signal timings or additional features.

## License

MIT License
