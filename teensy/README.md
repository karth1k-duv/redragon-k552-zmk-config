# Teensy 4.1 high-speed wired prototype

This directory tracks the experimental high-performance wired path for the K552 rebuild.

## Architecture

The existing nRF52840 remains the matrix scanner and wireless controller so the current 6x17 wiring does not need to be moved later.

```text
K552 6x17 matrix
       |
       v
Feather nRF52840
- 17 column inputs
- 6 rows via 74HC595
- BLE
- future low-latency 2.4 GHz
       |
       | fast inter-MCU link + IRQ
       v
Teensy 4.1 / i.MX RT1062
- USB High-Speed device
- wired HID endpoint
- latency instrumentation
- future Hall-effect WASD prototyping
       |
       v
PC
```

The Teensy should not replace the existing matrix wiring. In wired mode it receives key state/events from the nRF52840 and handles the high-speed USB endpoint. In battery mode it can be power-gated while the nRF52840 continues BLE/custom-2.4 operation.

## Performance targets

These are engineering targets, not yet verified claims:

- USB host report interval target: 125 us / 8 kHz
- Full switch-matrix scan target: begin at 8 kHz and characterize 16/20/32 kHz
- Keep RGB work outside the latency-critical scan path
- Timestamp transitions as close as possible to the first observed electrical state change
- Measure real end-to-end timing rather than inferring latency from configured rates

## Bring-up order

1. Finish and validate the existing nRF52840 matrix hardware first.
2. Verify the current ZMK configuration against the real 6x17 matrix.
3. Power the Teensy independently and confirm stable USB High-Speed enumeration.
4. Build a minimal wired keyboard HID test before connecting it to the Feather.
5. Define the nRF52840 <-> Teensy transport and an interrupt/event signal.
6. Send known synthetic key events across the link and verify host reports.
7. Measure link latency and jitter.
8. Integrate real matrix events.
9. Only after that, optimize scan/debounce/report scheduling.

## Inter-MCU link

Initial prototype recommendation:

- common 3.3 V logic levels
- common GND
- SPI for payload transfer
- one dedicated IRQ/event GPIO from nRF52840 to Teensy
- optional second ready/reset GPIO if testing shows it is useful

Exact pins are intentionally not frozen until the physical Teensy/protoboard placement is known.

## Hall-effect WASD later

The Teensy 4.1 exposes analog inputs and is useful for prototyping four Hall sensors under W/A/S/D. The final wireless-capable design should eventually place the Hall sampling path on the always-on wireless/scanner side, or otherwise ensure Rapid Trigger still works without keeping the Teensy fully powered on battery.

## Important constraints

- Teensy 4.1 I/O is 3.3 V; do not feed its GPIO/analog inputs 5 V.
- 8 kHz USB is a target that must be verified with actual USB captures/host measurements.
- The K552 remains mechanically limited by ordinary contact switches except for any future Hall-effect keys.
- The original RGB multiplex topology is still being reverse-engineered and should remain electrically independent from early USB/scan bring-up.
