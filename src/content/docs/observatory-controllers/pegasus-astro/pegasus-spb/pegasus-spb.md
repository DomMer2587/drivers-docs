---
title: Pegasus Saddle Powerbox
categories: ["observatory-controllers"]
description: INDI driver for the Pegasus Astro Saddle Powerbox (SPB) with power outputs, dew heaters and environment sensor.
thumbnail: ./pegasus-spb.webp
---

The Pegasus Astro **Saddle Powerbox** (SPB) is a power distribution box that mounts on the saddle of the telescope. The INDI driver controls its 12 V outputs and two dew heater ports, reports voltage and current, and reads the temperature and humidity sensor.

Implemented INDI interfaces:

- `AUX_INTERFACE`
- `POWER_INTERFACE`
- `WEATHER_INTERFACE`

## Features

- Switch the 12 V outputs (Quad Output) on and off
- Two dew heater ports (A and B), each usable as dew heater or as switchable 12 V output
- Dew heater power per port (0–100 %) and automatic dew control with adjustable aggressiveness
- Input voltage, total current and power; current per output
- Power statistics: average current, amp hours, watt hours
- Temperature, humidity and dew point, with calibration offsets
- Firmware version and uptime
- Custom labels for outputs and dew ports

## Installation

The driver is part of INDI core and needs no extra dependencies. To run it on its own:

```shell
indiserver -v indi_pegasus_spb
```

## Connection

Connect the Saddle Powerbox by USB; it appears as a serial port such as `/dev/ttyUSB0`. Select the port on the **Connection** tab and click **Connect**. The driver sends `P#` and expects a reply containing `SPB`.

## Driver Controls

### Power

- **Quad Output:** switch the 12 V outputs on or off. The label can be changed in **Labels**.
- **Sensors:** input voltage, total current and power.
- **Currents:** current of the 12 V outputs.
- **Power Statistics:** average current, amp hours, watt hours and total current, as reported by the box.

### Dew

- **Port A Mode / Port B Mode:** use the port as **Dew** heater output or as switchable 12 V **Power** output.
- **Dew Heater:** switch dew port A and B on or off.
- **Duty Cycles:** heater power per port in %.
- **Currents:** current per dew port.
- **Dew Control:** enable automatic dew control. While it is enabled, the box sets the heater power of both ports itself, and the driver keeps both ports switched on.
- **Auto Dew:** aggressiveness of the automatic dew control, 0 to 100.
- **Labels:** names for the dew ports, used in the other dew controls.

### Environment

- **Parameters:** temperature, humidity and dew point. The temperature is used as the critical weather parameter.
- **Temperature Offset / Humidity Offset:** correct the sensor readings, for example after comparing them with a reference.

### Firmware

Firmware version and uptime of the box.

## Operation

- Use **Quad Output** to switch the 12 V devices, and the port mode to decide whether a dew port powers a heater or another 12 V device.
- Use either **Dew Control** (automatic) or the **Duty Cycles** (manual) for the dew heaters.
- Name the outputs and dew ports in **Labels**; the names are saved in the driver configuration.

## Troubleshooting

- If the driver cannot connect, check that the box is powered and visible as a serial device, and that no other program (for example Pegasus Unity) uses the same port.
- If a dew port does not heat, check that its mode is **Dew** and not **Power**.
