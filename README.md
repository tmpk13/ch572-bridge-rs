# CH572D USB bridge

Firmware for the CH572D (U9) on the Forest Sensor Platform V1 board
([Forest-Sensor-Platform](https://github.com/tmpk13/Forest-Sensor-Platform),
KiCad). It makes the chip one composite USB device: a CDC-ACM serial port
bridged to the Wio-E5's UART and a CMSIS-DAP v2 probe for the Wio-E5's SWD
port, so one USB-C cable gives probe-rs flashing/RTT and the serial link.
The Wio-E5's own firmware is
[long-range-radio-rs](https://github.com/tmpk13/long-range-radio-rs).

- `mise run flash`: build and flash the bridge over USB; start it, then plug in a blank chip, or run `mise run boot` first
- `mise run boot`: restart a running bridge into its USB bootloader (`PORT=/dev/ttyACMn` if not ttyACM0)
- `mise run test`: host tests against simulated hardware, and the isp.py flasher's tests
- `mise run build`: build the firmware without flashing
- `mise run udev`: install the udev rules for the bootloader and the bridge, once per machine

First-time board setup: NOTES.md (First bring-up). Design notes: NOTES.md.
Components: ARCHITECTURE.md.

*Claude Code was utilized in the development process.*
