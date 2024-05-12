# debugger-adapters
Aliexpress STLINKv2 debugger to standard connector adapters

## Hardware
All part links are built into the KiCad parts, just generate the BOM from within KiCad and order. Miscellaneous parts are below.

- [Aliexpress STLINKv2](https://www.aliexpress.us/item/3256803389306042.html)
- [Cortex-debug Ribbon Cable](https://www.digikey.com/en/products/detail/cnc-tech/300-30-10-GR-0100F/5864892)

## Creating a Black Magic Probe
Black Magic Probe and Black Magic Debug are open-source SWD/JTAG debug projects for ARM Cortex microcontrollers. It gives you a bare GDB server and is platform-independent, so it will work on ARM Cortex microcontrollers regardless of brand or series.

To create a Black Magic Probe you need an existing debugger -- another Black Magic Probe, STLink (for ST-based debuggers), Atmel ICE (for Atmel chips), etc.

#### Resources
- [Black Magic Debug Website](https://black-magic.org/index.html)
- [Black Magic Debug GitHub](https://github.com/blackmagic-debug/blackmagic)
- [dfu-util Homepage](https://dfu-util.sourceforge.net/)
- [Zadig (Windows USB driver reinstaller)](https://zadig.akeo.ie/)

### Supported Hardware
- [Knockoff STLink v2 (Geehy MCU) - Aliexpress](https://www.aliexpress.us/item/3256803389306042.html)
- [Knockoff STLink v2 (STM32F101 MCU) - Aliexpress](https://www.aliexpress.us/item/3256803289344865.html)
- [STM32 "Blue Pill" (STM32F103C8T6) - Ebay](https://www.ebay.com/itm/292145343898)

### Wiring
Keep the SWD wires (between the BMP and the microcontroller) relatively short. SWD doesn't have a length spec, but under 30cm is a good rule of thumb. Slower baud rates will allow for longer lengths. It's recommended to extend the USB side instead of the SWD side.

### Reprogramming the BMP
The instructions below work on Linux/Mac. It works on Windows as well if you have `make` and `arm-none-eabi-gdb` installed, you can find these online.

1. Install `dfu-util`, if you don't have it already
   - Linux: `sudo apt install dfu-util`
   - OSX: `brew install dfu-util`
   - Windows (sigh):
     - Download [dfu-util](https://dfu-util.sourceforge.net/releases/dfu-util-0.11-binaries.tar.xz)
     - Unzip in Windows or using 7zip
     - The executable is `./dfu-util-0.11-binaries/dfu-util-0.11-binaries/win64/dfu-util.exe`, so anywhere you see `dfu-util` below substitute it with that path. Alternatively, add it PATH if you want this tool after this install.
2. Windows only: Download [Zadig](https://github.com/pbatard/libwdi/releases/download/v1.5.0/zadig-2.8.exe)
3. `git clone https://github.com/blackmagic-debug/blackmagic`
4. `cd` into the directory you cloned into
5. Make the binaries: `make -j8 PROBE_HOST=stlink`
   - Set `-j[jobs]` to the number of threads on your system. 8 works for most people.
6. Pull the metal shell off of your BMP-to-be by pulling it towards the USB port. Attach jumper wires to an existing BMP, and wire GND, VCC, SWDIO, and SWCLK to the pins/exposed pads on your BMP-to-be. You can wire it properly, or hold wires onto the pads until the next command is done.
7. Flash the bootloader: `arm-none-eabi-gdb -ex "tar ext [debugger device ID]" -ex "mon s" -ex "att 1" -ex "mon option erase" -ex "load" -batch src/blackmagic_dfu.elf`
   - Windows: Look in Device Manager for the device ID, use the lower of the 2 COM ports for the debugger. Ex. if the debugger shows up as COM4 and COM5, use COM4 for the commands. Run all commands in Powershell, it works on Windows 11 (no guarantees for 10 though). If you have the arm-none-eabi compiler installed, you should be able to just run arm-none-eabi-gdb .... in Terminal and it will work.
   - Mac/Linux: Find the debugger device ID in /dev. Use /dev/whatever for the following commands.
8. Plug your new BMP into your PC via USB, **unplug your existing BMP**.
9. Windows only: Reset USB drivers with Zadig. Launch Zadig (it's just an exe, no installation needed) and select *Options > List All Devices* and *Options > Ignore Hubs or Composite Parent*. Select the DFU device in the top dropdown. Select the **WinUSB** driver (not libusb-win32) using the up/down arrows on the right side. Then click Install Driver. Wait for it to finish.
10. Check that your new BMP shows up: `dfu-util -l`. It should say "found DFU" with some device information.
11. Flash the BMP firmware: `dfu-util -s 0x08002000:leave:force -D src/blackmagic.bin`

### Upgrading Your BMP
All you need to do is `git pull` the updated BMP firmware (or make any changes you need to), and re-`make`. Plug it in via USB and rerun step 11 above. The bootloader handles reflashing the firmware over USB.