### Summary

Adds support for the NASPIER 6C, an FMUv6C-class flight controller with an STM32H743 FMU and an STM32F103 IOMCU.

### Classification & Testing (check all that apply and add your own)

- [ ] Checked by a human programmer
- [ ] Non-functional change
- [ ] No-binary change
- [ ] Infrastructure change (e.g. unit tests, helper scripts)
- [ ] Automated test(s) verify changes (e.g. unit test, autotest)
- [ ] Tested manually, description below (e.g. SITL)
- [ ] Tested on hardware
- [ ] Logs attached
- [ ] Logs available on request

DRAFT: no hardware exists yet, so no hardware tests have been done. The hwdef files pass `chibios_hwdef.py` and the README passes markdownlint.

Tests to list here once hardware is available:

- Boots and connects over USB (both USB interfaces)
- IIM-42653 and BMI088 detected, each IMU tested on its own, HUD horizon correct
- IMU heater holds `BRD_HEAT_TARG`
- MS5611 barometer and IST8310 compass detected; external compass on GPS1 and GPS2
- RC input: PPM, SBUS, DSM (including bind) on the IOMCU; CRSF on TELEM2
- All 16 outputs tested with servos; DShot on the FMU outputs, `BRD_IO_DSHOT` on the IO outputs
- GPS tested on every UART; TELEM1 and TELEM2 tested with RTS/CTS
- CAN1 and CAN2 tested with a DroneCAN GPS
- Battery voltage checked against a multimeter on both inputs
- microSD logging, and flashing from the SD card in the bootloader

### Description

The NASPIER 6C follows the Pixhawk FMUv6C reference design. Its FMU pinout is the same as `Pixhawk6C`. The differences are:

- IMUs: IIM-42653 and BMI088 on a separate, heated sensor board, both mounted `ROTATION_ROLL_180`
- MS5611 barometer at 0x76 only, IST8310 compass at 0x0C
- Direct battery input (XT30, 4S to 12S) with voltage sensing only, plus a Brick input for an analog power module (disabled by default)
- USB power status taken from the LTC4417 power mux
- Spektrum power switched by the IOMCU
- No PX4 bootloader compatibility (`USE_ALT_RAM_MAP` removed)

The IOMCU uses the stock `iofirmware_f103_lowpolh.bin` (24 MHz crystal, standard IOMCU pinout).

Schematics and board documentation: <https://github.com/naspaw/NASPIER_6C>

Commits:

1. `hwdef: NASPIER6C`
2. `bootloaders: NASPIER6C`

The board ID is not allocated yet; that needs a separate PR against `Tools/AP_Bootloader/board_types.txt` first.
