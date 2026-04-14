# debugger-adapters
Aliexpress STLINKv2 debugger to standard connector adapters

## Hardware
All part links are built into the KiCad parts, just generate the BOM from within KiCad and order. Miscellaneous parts are below.

- [Aliexpress STLINKv2](https://www.aliexpress.us/item/3256803389306042.html)
- [Cortex-debug Ribbon Cable](https://www.digikey.com/en/products/detail/cnc-tech/300-30-10-GR-0100F/5864892)
- [Pre-assembled Cortex-debug Cable](https://www.aliexpress.us/item/3256805917162371.html)

## Assembly
The ribbon cable IDC connectors for the Cortex-Debug adapter are quite difficult to get exactly right. If the wire goes in at slightly the wrong angle, the conductors will bridge or connect to the wrong pins. After each connector is put on, check each pin with a multimeter to make sure it's not bridged to anything it shouldn't be.

## Creating a Black Magic Probe
Black Magic Probe and Black Magic Debug are open-source SWD/JTAG debug projects for ARM Cortex microcontrollers. It gives you a bare GDB server and is platform-independent, so it will work on ARM Cortex microcontrollers regardless of brand or series.

To create a Black Magic Probe you need an existing debugger -- another Black Magic Probe, STLink (for ST-based debuggers), Atmel ICE (for Atmel chips), etc.

#### Resources
- [Black Magic Debug Website](https://black-magic.org/index.html)
- [Black Magic Debug Codeberg](https://codeberg.org/blackmagic-debug/blackmagic)
- [dfu-util Homepage](https://dfu-util.sourceforge.net/)
- [Zadig (Windows USB driver reinstaller)](https://zadig.akeo.ie/)

### Supported Hardware
- [Knockoff STLink v2 (Geehy MCU) - Aliexpress](https://www.aliexpress.us/item/3256803389306042.html)
- [Knockoff STLink v2 (STM32F101 MCU) - Aliexpress](https://www.aliexpress.us/item/3256803289344865.html)
- [STM32 "Blue Pill" (STM32F103C8T6) - Ebay](https://www.ebay.com/itm/292145343898)

### Wiring
Keep the SWD wires (between the BMP and the microcontroller) relatively short. SWD doesn't have a length spec, but under 30cm is a good rule of thumb. Slower baud rates will allow for longer lengths. It's recommended to extend the USB side instead of the SWD side.

### Reprogramming the BMP
The Black Magic Probe project switched to Meson in current releases, which makes installation a little harder on Windows. On Windows, your best move is load up a Linux VM using VirtualBox or something similar. You can try to get MSYS2 to work, but no guarantees. Builds and uploads on Windows may need [Zadig](https://zadig.akeo.ie/) to manually assign Windows drivers to your BMP, you want to assign it to **WinUSB**.

Instructions for Linux/Mac:

1. Install dependencies
   - Linux: `sudo apt install meson ninja gcc-arm-none-eabi gdb-multiarch binutils-arm-none-eabi dfu-util libhidapi-dev libftdi1-dev libusb-1.0-0-dev`
   - OSX: `brew install meson ninja arm-none-eabi-gcc arm-none-eabi-binutils arm-none-eabi-gdb dfu-util hidapi libftdi libusb`
3. `git clone https://codeberg.org/blackmagic-debug/blackmagic`
4. `cd` into the directory you cloned into
5. Set up the build: `meson setup build --reconfigure --cross-file cross-file/stlink.ini -Dprobe=stlink -Dtargets=cortexar,cortexm,sam,stm,nxp,ti`
   - Which targets you choose are up to you and your use case. The ones shown above are a general recommendation. See [supported targets](https://black-magic.org/supported-targets.html) for others.
6. Build the binary: `meson compile -C build`
7. Pull the metal shell off of your BMP-to-be by pulling it towards the USB port. Attach jumper wires to an existing BMP, and wire GND, VCC, SWDIO, and SWCLK to the pins/exposed pads on your BMP-to-be. You can wire it properly, or hold wires onto the pads until the next command is done.
8. Flash the bootloader: `arm-none-eabi-gdb -ex "tar ext [debugger device ID]" -ex "mon s" -ex "att 1" -ex "mon option erase" -ex "load" -batch build/blackmagic_stlink_firmware.elf`
   - Find the debugger device ID in /dev. Use /dev/whatever for the following commands.
9. Plug your new BMP into your PC via USB, **unplug your existing BMP**.
10. Check that your new BMP shows up: `dfu-util -l`. It should say "found DFU" with some device information.
11. Flash the BMP firmware: `dfu-util -s 0x08002000:leave:force -D src/blackmagic.bin`

### Upgrading Your BMP
All you need to do is `git pull` the updated BMP firmware (or make any changes you need to), and rebuild. Plug it in via USB and rerun step 11 above. The bootloader handles reflashing the firmware over USB.
