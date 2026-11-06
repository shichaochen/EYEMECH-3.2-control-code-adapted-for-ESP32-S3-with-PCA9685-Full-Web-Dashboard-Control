# Wiring — EyeMech ε3.2, classic ESP32 direct drive

No PCA9685, no I2C. Six servo signal wires go straight to ESP32 GPIOs,
driven by the chip's LEDC hardware PWM (50 Hz, 12-bit).

## Signal pins

| CH | Function           | ESP32 GPIO | LEDC ch | Wire colour (typical) |
|----|--------------------|------------|---------|-----------------------|
| 0  | Left/Right pan     | 13         | 0       | signal (yellow/white) |
| 1  | Up/Down tilt       | 14         | 1       | signal (yellow/white) |
| 2  | Top-Left eyelid    | 25         | 2       | signal (yellow/white) |
| 3  | Bottom-Left eyelid | 26         | 3       | signal (yellow/white) |
| 4  | Top-Right eyelid   | 27         | 4       | signal (yellow/white) |
| 5  | Bottom-Right eyelid| 32         | 5       | signal (yellow/white) |

Change the pin map in one place only: `SERVO_PIN[]` at the top of
`EyeMech/EyeMech.ino`. LEDC channel numbers follow the table order
automatically.

Pins were chosen to avoid:
- strapping pins 0 / 2 / 5 / 12 / 15 (boot behaviour),
- input-only pins 34–39 (cannot output PWM),
- flash pins 6–11,
- PSRAM pins 16 / 17 (occupied on WROVER modules — the earlier shortlist
  used 16/17/18/19, which only fits WROOM boards; the table above works
  on both WROOM and WROVER).

## Power — read this first

```
Servo 5-6 V supply (+) ----+----> all six servo red wires
                           |
ESP32 GND -----------------+----> all six servo brown/black wires
                           |
                           +----> servo supply GND (common ground)

ESP32 GPIO 13/14/25/26/27/32 --> servo signal wires (one each)
```

- **Servos must have their own 5–6 V supply.** A 5 V / 4–5 A supply is
  recommended: six small servos can pull ~0.7 A each at stall, ~4 A peak
  together. A 5 V / 2 A phone charger will brown out and the ESP32 will
  reboot mid-blink.
- **Common ground is mandatory.** Servo supply GND, ESP32 GND and all
  servo GND wires must be one net, or the signal has no reference and
  the servos will jitter or ignore commands.
- **Do not power the servos from the ESP32 board's 5 V pin or 3.3 V rail.**
  The old PCA9685 board provided a servo power rail with screw terminals;
  without it, split power on a small piece of perfboard, a servo breakout
  board, or a 6-way Y-harness. Keep the servo power wires short and
  twisted with ground where possible.
- ESP32 itself is powered separately as usual (USB or its own 5 V in).

## Signal voltage

ESP32 GPIO is 3.3 V logic. Most hobby servos (SG90, MG90S, MG996R)
accept a 3.3 V signal without complaint. If a servo ignores commands or
jitters while the others work, that servo wants a 5 V signal — add a
simple level shifter (e.g. 74HCT245, or a single NPN + pull-up per
channel) on that signal line only.

## First power-on checklist

1. Signal wires to the GPIOs above, servo power disconnected.
2. Flash the sketch, join the `EyeMech-Controller` AP, open
   `http://192.168.4.1` — the dashboard should load and the digital twin
   should respond.
3. Connect the common ground first, then servo power. The eyes should
   wake in Sleep mode.
4. Run **Calibrate** (14-step wizard) once — or reuse calibration saved
   from the PCA9685 build, the pulse mapping is identical.
