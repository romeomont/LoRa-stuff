# LoRa stuff

## Ikoka Nano — SSD1306 OLED display

The Ikoka Nano (MeshCore firmware, nRF52840) ships with no onboard display,
but its Qwiic (I2C) connector can drive a standard 128x64 I2C SSD1306 OLED.
Everything you need is in [`ikoka_nano_ssd1306/`](ikoka_nano_ssd1306).

You have two ways to get this running: **flash a ready-made build** (fastest,
no tools to install beyond the flasher) or **build it yourself from source**.

### What's in this folder

```
ikoka_nano_ssd1306/
├── SSD1306_DISPLAY.md          Wiring + troubleshooting notes
├── prebuilt_firmware/          Ready-to-flash firmware (pick one)
└── variant_changes/            Modified MeshCore source files (target.h, platformio.ini)
```

### 1. Wire up the OLED

Connect the SSD1306 module's 4 wires to the Ikoka Nano's Qwiic/I2C connector:

| OLED pin | Ikoka Nano  |
|----------|-------------|
| GND      | GND         |
| VCC      | 3V3         |
| SDA      | SDA (GPIO 4)|
| SCL      | SCL (GPIO 5)|

No reset pin is used — most SSD1306 breakout boards don't expose one.

### 2. Pick a firmware file to flash

Inside [`prebuilt_firmware/`](ikoka_nano_ssd1306/prebuilt_firmware), pick the
`.zip` matching your setup:

| File | Use this if... |
|------|-----------------|
| `ikoka_nano_nrf_22dbm_companion_radio_usb.zip` | Your radio module is the plain E22-900M22 (no PA) **and** you'll pair with the MeshCore app over USB serial |
| `ikoka_nano_nrf_22dbm_companion_radio_ble.zip` | Same radio module, but you'll pair over Bluetooth |
| `ikoka_nano_nrf_30dbm_companion_radio_usb.zip` | You have the E22-900M30 PA module, USB pairing |
| `ikoka_nano_nrf_30dbm_companion_radio_ble.zip` | E22-900M30 PA module, Bluetooth pairing |
| `ikoka_nano_nrf_33dbm_companion_radio_usb.zip` | You have the E22-900M33 PA module, USB pairing |
| `ikoka_nano_nrf_33dbm_companion_radio_ble.zip` | E22-900M33 PA module, Bluetooth pairing |

Not sure which E22 power module you have? Check the label printed on the small
metal-can RF module soldered to the board (22/30/33 refers to its dBm output).
If in doubt, **use the 22dbm build** — it just means lower transmit power, it
won't damage anything.

### 3. Flash it

The Ikoka Nano is an nRF52840 board, flashed the same way as RAK4631/Heltec
T114:

1. **Put the board into bootloader/DFU mode**: press the reset button once.
   If the board doesn't show up as a new serial port, quickly double-click
   reset instead, or unplug/replug the USB cable.
2. **Install the flashing tool** (one-time):
   ```
   pip install adafruit-nrfutil --break-system-packages
   ```
3. **Find the serial port** the board enumerates as:
   - Windows: check Device Manager → Ports (COM & LPT) for a new `COMx`
   - Linux: usually `/dev/ttyACM0`
   - macOS: usually `/dev/tty.usbmodemXXXX`
4. **Flash** (replace `<port>` and the filename with your actual values):
   ```
   adafruit-nrfutil --verbose dfu serial --package ikoka_nano_nrf_22dbm_companion_radio_usb.zip -p <port> -b 115200 --singlebank --touch 1200
   ```
5. Wait for it to finish — it'll print progress and end with a success
   message. The board will reboot automatically into the new firmware.

### 4. Screen stays blank?

The firmware assumes the most common I2C address, `0x3C`. If your specific
module uses `0x3D` instead:

1. Open [`variant_changes/platformio.ini`](ikoka_nano_ssd1306/variant_changes/platformio.ini)
2. Find the commented line `; -D DISPLAY_ADDRESS=0x3D` and uncomment it
3. Rebuild from source (see below) and reflash

If the image looks scrambled/partial rather than blank, your module is
probably not 128x64 (e.g. a smaller 128x32 panel) — that needs a small code
change, not just a build flag. Let us know and we can add it.

### Building from source instead (VS Code + PlatformIO)

If you'd rather build it yourself (e.g. to change the I2C address, or to pull
in newer MeshCore changes), and you're using **VS Code with the PlatformIO
extension**:

1. Clone MeshCore: `git clone https://github.com/meshcore-dev/MeshCore.git`,
   then open that folder in VS Code (`File → Open Folder...`). PlatformIO
   should auto-detect it and show its own sidebar icon (the little alien
   head) — give it a minute the first time to install toolchains.
2. Copy [`variant_changes/target.h`](ikoka_nano_ssd1306/variant_changes/target.h)
   and [`variant_changes/platformio.ini`](ikoka_nano_ssd1306/variant_changes/platformio.ini)
   over the matching files in `MeshCore/variants/ikoka_nano_nrf/`, overwriting
   the originals.
3. In the PlatformIO sidebar (click the alien head → **PROJECT TASKS**), find
   the environment you want, e.g. `ikoka_nano_nrf_22dbm_companion_radio_usb`
   (swap `22dbm` for `30dbm`/`33dbm` and `companion_radio_usb` for
   `companion_radio_ble`/`repeater`/`room_server`/`kiss_modem` to match your
   hardware/use case — same guidance as the table in step 2 above).
4. Expand that environment and click **Build** (checkmark icon) first to
   confirm it compiles.
5. Put the board into bootloader mode (press reset once; double-click reset
   or unplug/replug USB if that doesn't work), then click **Upload** (arrow
   icon) under the same environment. PlatformIO already knows how to flash
   this board (it's configured to use `nrfutil` internally) — you don't need
   to install or run anything separately.
6. If Upload can't find the board, check the COM port: PlatformIO usually
   auto-detects it, but you can also select it manually via the PlatformIO
   sidebar → **Devices**, or `File → Preferences → PlatformIO: Force Upload
   Port` if it picks the wrong one.

The built firmware also lands at `.pio\build\<env-name>\firmware.zip` inside
the MeshCore folder, in case you want it for reference or to share.

### What actually changed

This enables `DISPLAY_CLASS=SSD1306Display` on the Ikoka Nano board (it
defaulted to no display at all) and wires it to the I2C bus already used by
the onboard sensor/RTC — the same pattern MeshCore already uses for the
RAK4631 and Ikoka Handheld boards. See
[`SSD1306_DISPLAY.md`](ikoka_nano_ssd1306/SSD1306_DISPLAY.md) for more detail.
