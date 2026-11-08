---
title: Avalon UPAS
categories: ["auxiliaries"]
description: INDI driver for the Avalon Instruments Universal Polar Alignment System (UPAS), with automatic polar alignment correction in Ekos.
thumbnail: ./avalon-upas.webp
---

The Avalon Instruments **UPAS** (Universal Polar Alignment System) is a motorised base that corrects the polar alignment of a mount in azimuth and altitude. Its controller runs GRBL firmware; the driver moves the two axes with GRBL jog commands.

The driver implements the INDI **PAC (Polar Alignment Correction)** interface, so the Ekos Polar Alignment Assistant can measure the error and move the UPAS until the error is below the threshold you choose.

## Features

- Relative corrections in degrees on azimuth and altitude; both axes move in one jog
- Automatic correction loop in Ekos (KStars 3.8.2 or newer)
- Feed rate per axis; in a combined move, no axis runs faster than its own feed rate
- Gear ratio per axis in mm per arcminute, preset to the Avalon values
- Software backlash compensation with a selectable approach direction per axis
- Direction reverse per axis
- Position readout in degrees
- Abort

## Installation

The driver is part of INDI core and needs no extra dependencies. Connect the UPAS controller by USB; it appears as a serial port such as `/dev/ttyUSB0` or `/dev/ttyACM0`.

To run it on its own:

```shell
indiserver -v indi_avalon_upas
```

## Configuration

1. In the Ekos profile editor, select **Avalon UPAS** under Auxiliary.
2. In the INDI Control Panel, choose the port on the **Connection** tab (115200 baud) and click **Connect**. The driver resets GRBL, checks the status reply and reads the firmware version.
3. Check **Gear Ratio** and **Feed Rate**. The defaults are the values Avalon specifies for the UPAS.
4. Check the directions once, as described in [Direction check](#direction-check).
5. Save the configuration on the **Options** tab.

> [!NOTE]
> A saved configuration keeps its values. If you used an older version of the driver, your configuration may still hold other gear ratios. Set them to 2 and 22 and save the configuration again.

| Property | Meaning |
|----------|---------|
| Manual Adjustment | Relative move in **degrees** per axis, range ±10 |
| Abort Motion | Stop both axes (GRBL jog cancel) |
| Position | Position in degrees, from the GRBL machine position since power-on |
| Feed Rate | Speed per axis in mm/min, 10 to 1000. Default: azimuth 1000, altitude 700 |
| Gear Ratio | mm of GRBL travel per arcminute of correction. Default: azimuth 2, altitude 22 |
| Azimuth / Altitude Reverse | Invert an axis if it moves the wrong way |
| Backlash | Enable or disable the backlash compensation |
| Backlash (Azimuth / Altitude) | Backlash per axis in **arcminutes**, 0 to 60 |
| Azimuth / Altitude Approach | Direction (+ or −) in which every move ends while backlash compensation is enabled |
| Firmware | GRBL version reported by the controller |

## How a move works

A correction in degrees is converted to GRBL travel with the gear ratio:

```
travel [mm] = correction [deg] × 60 × gear ratio [mm/arcmin]
```

With the default gear ratios, 1 arcminute is 2 mm in azimuth and 22 mm in altitude. The driver sends one relative jog for both axes, for example `$J=G91G21X2.0000Y22.0000 F703`. GRBL has a single feed rate per jog, so the driver picks the highest feed rate at which neither axis exceeds its own **Feed Rate**. A move is complete when GRBL reports that it is idle again.

## Backlash compensation

GRBL has no backlash compensation of its own, so the driver adds it in software. While **Backlash** is enabled, every move ends in the approach direction you select for that axis:

- A move in the approach direction is sent unchanged.
- A move against the approach direction overshoots by the backlash and is followed by a second jog back by the same amount. GRBL queues both jogs and runs them one after the other.

The backlash is entered in arcminutes and converted with the gear ratio, like the corrections themselves.

> [!TIP]
> On the altitude axis the last move usually works best when it lifts the mount against gravity. Choose the approach direction accordingly.

To measure the backlash, start the Polar Alignment Assistant and wait for the refresh phase. Move one axis by about 10 arcminutes in one direction, note the error, move back by the same amount and note the error again. The difference from the expected travel is the backlash of that axis.

> [!NOTE]
> The backlash uses the standard property `PAC_BACKLASH_STEPS`, but in arcminutes rather than motor steps, because GRBL works in mm and the UPAS has no step count that a user could relate to the correction.

## Direction check

1. Start the Polar Alignment Assistant and let it reach the refresh phase.
2. Enter a small azimuth move, for example `0.1`, in **Manual Adjustment** and watch the error in Ekos. If the error grows instead of shrinking, enable **Azimuth Reverse**.
3. Repeat with altitude and **Altitude Reverse**.

The reverse settings change the GRBL direction mask (`$3`), which the controller keeps after power-off.

## Troubleshooting

- **No connection:** check that no other program (for example the Avalon tool) uses the port. The driver expects a GRBL status reply containing `MPos` or `WPos`.
- **Moves are far too large or too small:** check the gear ratios. Older configurations may hold values from before the unit was mm per arcminute.
- **Moves are refused:** a GRBL alarm blocks all jogs. Disconnect and connect again, which sends a GRBL soft reset; if the alarm remains, power-cycle the controller.
