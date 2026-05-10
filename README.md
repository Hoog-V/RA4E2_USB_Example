# USB CDC ACM — EK-RA4E2

Echo demo using the Zephyr USB CDC ACM driver on the Renesas EK-RA4E2 board.
Data received on the USB virtual serial port is echoed back to the sender.

## Overview

This sample brings up a USB CDC ACM device using Zephyr's next-generation USB device stack (`CONFIG_USB_DEVICE_STACK_NEXT`). The board enumerates as a virtual COM port on the host. All characters sent to that port are echoed back, and the Zephyr console is routed through the same interface.

The `ra4e2_usbfs.overlay` configures:
- Port 8 I/O (P8.14 / P8.15 — USB FS differential pair, P4.7 — VBUS sense)
- PLL → 240 MHz, ICLK → 120 MHz, UCLK → 48 MHz (required for USB FS)
- `usbfs` peripheral with a single `cdc_acm_uart0` child node set as `zephyr,console`

> **Note:** The west manifest points to fork branches of both `zephyr` and `hal_renesas` that contain patches not yet merged upstream. See the comments in `west-manifest/west.yml` for the linked PRs.

## Prerequisites

- [west](https://docs.zephyrproject.org/latest/develop/west/index.html) installed (`pip install west`)
- A Zephyr-compatible toolchain (e.g. Zephyr SDK)
- J-Link or the on-board debug interface for flashing

## Workspace initialisation

```bash
# 1. Create a workspace directory and enter it
mkdir ra4e2-usb-ws && cd ra4e2-usb-ws

# 2. Initialise west from this repository's manifest
west init -m https://github.com/Hoog-V/RA4E2_USB_Example --mf west-manifest/west.yml

# 3. Fetch all projects listed in the manifest
west update
```

If you have already cloned this repository locally, point `west init` at the local path instead:

```bash
west init -l /path/to/RA4E2_USB_Example --mf west-manifest/west.yml
west update
```

## Building

```bash
west build -b ek_ra4e2 /path/to/RA4E2_USB_Example \
  -- -DDTC_OVERLAY_FILE=ra4e2_usbfs.overlay
```

The overlay is picked up automatically when building from inside the project directory because CMake searches the source root. If you build from a different directory, pass the absolute path to the overlay:

```bash
west build -b ek_ra4e2 /path/to/RA4E2_USB_Example \
  -- -DDTC_OVERLAY_FILE=/path/to/RA4E2_USB_Example/ra4e2_usbfs.overlay
```

## Flashing

```bash
west flash
```

## Running

1. Plug the board into a Linux host via USB.
2. Confirm it enumerated:

```
$ dmesg | tail
cdc_acm 1-1:1.0: ttyACM0: USB ACM device
```

3. Open the serial port (115200 8N1):

```bash
minicom --device /dev/ttyACM0
```

4. The board prints on startup:

```
Wait for DTR
```

5. After minicom raises DTR:

```
DTR set, start test
Baudrate detected: 115200
```

Characters typed in minicom are echoed back by the board.

## Troubleshooting

**ModemManager interference** — ModemManager probes CDC ACM devices and can send spurious `AT` commands. Add a udev rule to suppress it:

```
# /etc/udev/rules.d/99-ra4e2-cdc.rules
ATTRS{idVendor}=="8086", ATTRS{idProduct}=="f8a1", ENV{ID_MM_DEVICE_IGNORE}="1"
```

Reload rules with `sudo udevadm control --reload && sudo udevadm trigger`.
