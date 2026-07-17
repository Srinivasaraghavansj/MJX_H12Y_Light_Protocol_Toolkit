# MJX H12Y Light Protocol Toolkit

Arduino Nano tools for reading, passing through, and emulating the proprietary
three-field signal used between the MJX H12Y receiver and its stock lighting
board.

The protocol values in this repository were measured from a real H12Y and
validated with a complete ten-state capture matrix.

Download zip to find all code.

## Included sketches

### `H12Y_CH3_Learner`

- Reads the stock receiver signal on Nano D4.
- Mirrors D4 to D7 at interrupt level so the lighting board remains connected
  and can be watched during learning.
- Detects and decodes MJX pulse-field frames.
- Captures all nine steering/throttle combinations.
- Captures CH3 released and pressed values.
- Stores capture rows and calibration in Nano EEPROM.
- Prints machine-readable CSV through Serial Monitor.

Open:
[`sketches/H12Y_CH3_Learner/H12Y_CH3_Learner.ino`](sketches/H12Y_CH3_Learner/H12Y_CH3_Learner.ino)

### `H12Y_D7_Command_Emulator`

- Generates the validated MJX signal on Nano D7 using Timer1.
- Starts with steering centered, throttle neutral, and CH3 released.
- Does not generate an automatic CH3 press at startup, so the stock lighting
  board retains its normal lights-ON default.
- Accepts natural Serial commands such as `forward left`, `straight`,
  `backward right`, and `press`.
- Includes automatic combination, brake, and mode-cycle tests.

Open:
[`sketches/H12Y_D7_Command_Emulator/H12Y_D7_Command_Emulator.ino`](sketches/H12Y_D7_Command_Emulator/H12Y_D7_Command_Emulator.ino)

## Hardware

- MJX H12Y crawler with stock receiver and lighting board
- Classic Arduino Nano with ATmega328P
- Common regulated power and ground appropriate for the installed hardware
- Logic-level conversion if the lighting-board input does not tolerate the
  Nano's 5 V output

## Wiring

| Connection | Nano pin |
|---|---:|
| Stock receiver CH3 signal | D4 input |
| Stock lighting-board signal input | D7 output |
| Receiver and lighting-board ground | GND |

The receiver CH3 output must connect to D4 only. D7 must be the only signal
driver connected to the lighting-board input. Never connect D7 directly to the
receiver's signal output.

See [WIRING_AND_SAFETY.md](docs/WIRING_AND_SAFETY.md) before powering the
circuit.

## Arduino IDE setup

1. Install Arduino IDE.
2. Select **Tools → Board → Arduino AVR Boards → Arduino Nano**.
3. Select **ATmega328P** or **ATmega328P (Old Bootloader)** to match the Nano.
4. Select the correct serial port.
5. Open the `.ino` file from its matching sketch directory.
6. Compile and upload.
7. Open Serial Monitor at **115200 baud** with **Newline** line ending.

No third-party Arduino libraries are required.

## Quick reader workflow

Upload `H12Y_CH3_Learner`, then use one of these commands:

```text
sample
raw
auto
matrix
dump
resume
help
```

`matrix` begins a new ten-state automatic learning run and clears the previous
capture log. `resume` continues or validates an interrupted matrix stored in the
same Nano's EEPROM.

D4 is mirrored to D7 continuously, including while the sketch is waiting for a
command. The lighting board should therefore behave as it did when connected
directly to the stock receiver.

## Quick emulator workflow

Upload `H12Y_D7_Command_Emulator`. D4 is ignored by this sketch. D7 begins
generating the CH3-released, neutral, centered signal immediately when Arduino
`setup()` starts.

Examples:

```text
forward left
straight
neutral
backward right
press
status
```

The words in a driving combination can be reversed:

```text
left forward
right backward
straight forward
```

Mode commands:

```text
press
hold
release
on
hazard
off
```

The lighting board owns the visible mode state. Its observed sequence is:

```text
ON -> HAZARD -> OFF -> ON
```

The emulator's direct `on`, `hazard`, and `off` commands track this sequence in
software. Tracking can become incorrect if the lighting board resets without
the Nano or if a CH3 gesture is missed. Powering both devices together keeps
their state aligned.

Automatic tests:

```text
combos
brake
modes
all
stop
```

- `combos` runs all nine steering/throttle combinations.
- `brake` sends forward, then neutral, to test the board's derived brake logic.
- `modes` sends three spaced CH3 gestures.
- `all` runs combinations, brake, and mode tests.
- `stop` returns to centered steering, neutral throttle, and CH3 released.

## Validated calibration

| Timing or field | Value |
|---|---:|
| Active polarity | HIGH |
| Long LOW sync | 12154 µs |
| Short LOW separator | 1027 µs |
| CH3 released | 6104 µs |
| CH3 pressed | 5110 µs |
| Throttle reverse | 2063 µs |
| Throttle neutral | 3027 µs |
| Throttle forward | 4071 µs |
| Steering left | 2036 µs |
| Steering center | 3026 µs |
| Steering right | 4069 µs |

Field order is CH3, throttle, steering. See [PROTOCOL.md](docs/PROTOCOL.md) and
the original [validated capture](data/validated_capture.csv).

## Repository layout

```text
MJX_H12Y_Light_Protocol_Toolkit/
├── README.md
├── CHANGELOG.md
├── data/
│   ├── calibration.txt
│   └── validated_capture.csv
├── docs/
│   ├── PROTOCOL.md
│   ├── TESTING.md
│   └── WIRING_AND_SAFETY.md
└── sketches/
    ├── H12Y_CH3_Learner/
    │   ├── H12Y_CH3_Learner.ino
    │   └── H12YProtocolData.h
    └── H12Y_D7_Command_Emulator/
        └── H12Y_D7_Command_Emulator.ino
```

## Known limitations

- The stock lighting board controls groups and behaviors, not every LED as an
  independently addressable output.
- Brake behavior is derived by the stock board from throttle transitions.
- ON/Hazard/Off state is remembered by the stock board rather than represented
  by three steady CH3 field widths.
- Captured timings naturally vary by several microseconds. The validated values
  are representative values accepted by the tested board.
- This repository targets the classic ATmega328P Nano and uses AVR registers
  directly for accurate timing.

## License

No license file is included. Choose and add the license you want before making
the GitHub repository public.

