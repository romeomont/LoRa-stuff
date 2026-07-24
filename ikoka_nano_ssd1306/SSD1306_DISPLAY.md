# Ikoka Nano — SSD1306 OLED display support

The Ikoka Nano ships with no onboard display, but its Qwiic (I2C) connector can
drive a standard 128x64 I2C SSD1306 OLED. This board now builds with
`DISPLAY_CLASS=SSD1306Display` by default across all `ikoka_nano_nrf_*` firmware
variants (companion radio BLE/USB, repeater, room server, kiss modem).

## Wiring

No soldering required if your OLED module has a Qwiic/STEMMA QT connector
(4-pin JST-SH plug) — just plug a Qwiic/STEMMA QT cable straight into the
Ikoka Nano's I2C port.

If your module only has bare pin headers, connect its 4 pins to the Ikoka
Nano's I2C port (soldered wires, or jumper cables if the pins line up with a
socket):

| OLED pin | Ikoka Nano |
|----------|------------|
| GND      | GND        |
| VCC      | 3V3        |
| SDA      | SDA (GPIO 4) |
| SCL      | SCL (GPIO 5) |

No reset pin is used (`PIN_OLED_RESET=-1` — most I2C SSD1306 breakouts don't
expose one).

## If the screen stays blank

The default I2C address is `0x3C`, which covers the large majority of 128x64
SSD1306 modules. If nothing shows up:

1. Open `variants/ikoka_nano_nrf/platformio.ini`.
2. Uncomment the line `; -D DISPLAY_ADDRESS=0x3D` in the `[ikoka_nano_nrf]`
   section (and remove the `0x3C` assumption if one was added).
3. Rebuild and reflash.

If your module isn't 128x64 (e.g. it's a smaller 128x32 panel), the image will
look corrupted/partial — that needs a small follow-up code change to
`src/helpers/ui/SSD1306Display.h` (resolution is currently hardcoded there for
all boards that use this display class), not just a build flag.

## Building

```
cd MeshCore
pio run -e ikoka_nano_nrf_22dbm_companion_radio_usb
```

Swap `22dbm` for `30dbm`/`33dbm` to match your E22 power amplifier module, and
`companion_radio_usb` for `companion_radio_ble`/`repeater`/`room_server`/
`kiss_modem` for the other build types. The output firmware appears at
`.pio/build/<env-name>/firmware.zip` (and `firmware.hex`).

## Flashing

The Ikoka Nano is an nRF52840 board flashed the same way as RAK4631/Heltec T114:

1. Put the board into DFU/bootloader mode: press the reset button once. If
   that doesn't work, quickly double-click reset, or unplug/replug USB
   (see MeshCore FAQ 6.7 for Xiao nRF52 boards).
2. Install the flashing tool (one-time): `pip install adafruit-nrfutil --break-system-packages`
3. Find the serial port the board enumerates as (`/dev/ttyACM0` on Linux, a
   `COMx` port on Windows).
4. Flash:
   ```
   adafruit-nrfutil --verbose dfu serial --package firmware.zip -p <port> -b 115200 --singlebank --touch 1200
   ```

On Windows, replace `<port>` with the COM port shown in Device Manager once the
board is in bootloader mode.
