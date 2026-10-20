# Redragon K552 hardware bring-up

This branch is for validating the rewired K552 matrix before finalizing the physical keymap.

## Controller

Adafruit Feather nRF52840 Express + 74HC595.

### 74HC595 control

| Signal | Feather | nRF52840 | 74HC595 |
| --- | --- | --- | --- |
| Data | MOSI | P0.13 | pin 14 / SER |
| Clock | SCK | P0.14 | pin 11 / SRCLK |
| Latch | D10 | P0.27 | pin 12 / RCLK |

74HC595 power/control:

- pin 16 VCC -> 3.3 V
- pin 8 GND -> GND
- pin 10 SRCLR/MR -> 3.3 V
- pin 13 OE -> GND

Rows are Q0-Q5:

- Q0 / pin 15 -> R1
- Q1 / pin 1 -> R2
- Q2 / pin 2 -> R3
- Q3 / pin 3 -> R4
- Q4 / pin 4 -> R5
- Q5 / pin 5 -> R6

## Columns

| Column | Feather label | nRF GPIO |
| --- | --- | --- |
| C1 | TX | P0.25 |
| C2 | RX | P0.24 |
| C3 | D5 | P1.08 |
| C4 | D6 | P0.07 |
| C5 | D9 | P0.26 |
| C6 | D11 | P0.06 |
| C7 | D12 | P0.08 |
| C8 | D13 | P1.09 |
| C9 | A0 | P0.04 |
| C10 | A1 | P0.05 |
| C11 | A2 | P0.30 |
| C12 | A3 | P0.28 |
| C13 | A4 | P0.02 |
| C14 | A5 | P0.03 |
| C15 | SDA | P0.12 |
| C16 | SCL | P0.11 |
| C17 | MISO | P0.15 |

## Bring-up firmware

`build.yaml` produces two artifacts on this branch:

- `feather17x6-studio`: normal ZMK Studio firmware.
- `feather17x6-bringup`: temporary USB-logging firmware for matrix validation.

The GPIO matrix driver emits debug messages in the form `Sending event at <row>,<col> state on/off` when debug logging is enabled. Use the bring-up artifact over USB while identifying the physical matrix.

## Test order

1. With power disconnected, continuity-check 3.3 V to GND for an accidental short.
2. Verify 74HC595 VCC/GND/MR/OE and MOSI/SCK/D10 wiring.
3. Verify each of the 17 column wires reaches the intended Feather GPIO.
4. Repair the damaged BAW56 common-anode diode locations before expecting their associated keys to scan correctly.
5. Flash `feather17x6-bringup`.
6. Open the USB serial/CDC log and press every physical key one at a time.
7. Record the reported `(row,column)` for each physical key.
8. Use that table to replace the temporary row-major transform/keymap with the real K552 physical layout.
9. After validation, return to `feather17x6-studio` for normal use.

## Current assumptions to verify on hardware

- Matrix size is 6 x 17.
- Rows are driven Q0 through Q5 in R1 through R6 order.
- Diode direction is configured as `row2col`.
- The existing physical layout/keymap is still provisional until every key is mapped from actual scan events.
