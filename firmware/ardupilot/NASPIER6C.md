# NASPIER 6C: ArduPilot board port (draft)

This folder is a **local draft** of what an ArduPilot pull request for the NASPIER 6C would contain. Nothing here has been pushed, forked or submitted. The paths below `firmware/ardupilot/` mirror the upstream [ArduPilot](https://github.com/ArduPilot/ardupilot) tree, so the files could be copied into an ArduPilot checkout unchanged.

Deliberately left out of this draft:

- **Board ID**: `APJ_BOARD_ID AP_HW_NASPIER6C` is used by name, but no number is allocated in `Tools/AP_Bootloader/board_types.txt`.
- **USB VID/PID**: `USB_VENDOR` / `USB_PRODUCT` are not set, so ArduPilot's generic IDs would be used.

Based on ArduPilot `master` at `aa788b0` (2026-09-30).

## PR structure

```text
firmware/ardupilot/
├── NASPIER6C.md                        this file, not part of the PR
├── PR_DESCRIPTION.md                   draft PR body, not part of the PR
├── libraries/AP_HAL_ChibiOS/hwdef/NASPIER6C/
│   ├── hwdef.dat                       main firmware hardware definition
│   ├── hwdef-bl.dat                    bootloader hardware definition
│   ├── README.md                       board documentation, becomes the wiki page
│   ├── NASPIER6C_top.png               pinout image (from hardware/baseboard)
│   ├── NASPIER6C_bottom.png            power layout image
│   └── NASPIER6C_dimensions.png        mechanical drawing
└── Tools/bootloaders/                  NOT CREATED, see below
    ├── NASPIER6C_bl.bin
    └── NASPIER6C_bl.hex
```

No `defaults.parm` is needed. ArduPilot's rules say battery and serial defaults go in `hwdef.dat`, and every other board default is set there with `define` lines.

### Commit plan (as ArduPilot expects it)

| # | Commit title | Files | Status |
| --- | --- | --- | --- |
| 0 | `Tools: add board ID for NASPIER6C` (separate, earlier PR) | `Tools/AP_Bootloader/board_types.txt` | skipped on purpose (draft) |
| 1 | `hwdef: NASPIER6C` | everything in `libraries/AP_HAL_ChibiOS/hwdef/NASPIER6C/` | ready (draft) |
| 2 | `bootloaders: NASPIER6C` | `Tools/bootloaders/NASPIER6C_bl.bin`, `.hex` | needs a board ID and an ARM toolchain build |

Each commit may touch only one subsystem (CI checks this), and the PR must not contain merge commits.

## Current upstream requirements

From `dev/source/docs/porting.rst`, `readme_file.rst` and the CI rules on ArduPilot master:

| Requirement | Status |
| --- | --- |
| Board ID allocated through a PR to `board_types.txt` (blocks of at most 10 IDs) | skipped (draft). The 7160 to 7169 range is currently free |
| `APJ_BOARD_ID` uses a name from `board_types.txt`, not a number (CI rejects numbers) | done |
| Board name 13 characters or less (`NASPIER6C` is 9) | done |
| `hwdef.dat` and `hwdef-bl.dat` with the same board ID | done |
| Bootloader: `FLASH_RESERVE_START_KB 0`, `FLASH_BOOTLOADER_LOAD_KB` equal to the main `FLASH_RESERVE_START_KB` (128) | done |
| Parameter storage declared (FRAM, `HAL_WITH_RAMTRON`) | done |
| Battery monitor and serial defaults in `hwdef.dat`, not in `defaults.parm` | done |
| `README.md` with the required sections (intro and photo, where to buy, specs, pinout, UART mapping, RC input, PWM, GPIOs, battery, compass, firmware, loading) | done |
| README passes markdownlint (no embedded HTML, markdown tables) | done, 0 issues with ArduPilot's `.markdownlint-cli2.jsonc` |
| Bootloader binaries built with `Tools/scripts/build_bootloaders.py NASPIER6C` | not done |
| Firmware builds (`./waf configure --board NASPIER6C && ./waf copter`) | not done, needs the ARM toolchain |
| Hardware tested; tests listed in the PR description | not possible yet, no hardware |

### Acceptance risk

ArduPilot does not accept every board. The porting guide lists reasons a board may be rejected, and three apply to the NASPIER 6C today:

- It is "excessively similar to an existing board". The FMU pinout is the Pixhawk 6C pinout.
- "Sufficient testing on real hardware has not been performed, or cannot yet be done."
- It is not yet manufactured in quantity or for sale.

The open hardware documentation counts in its favour. Asking on the ArduPilot Discord hardware channel before a real submission is recommended.

## Differences from `Pixhawk6C`

The FMU pinout matches Pixhawk6C for every UART, SPI, I2C, SDMMC, CAN, PWM, ADC, LED, heater and power pin. The port is therefore a small, reviewable diff against `libraries/AP_HAL_ChibiOS/hwdef/Pixhawk6C/hwdef.dat`:

| Item | Pixhawk6C | NASPIER6C | Source |
| --- | --- | --- | --- |
| USB VID/PID | Holybro `0x3162/0x0053` | not set (draft) | - |
| Board ID | `TARGET_HW_PX4_FMU_V6C` | `AP_HW_NASPIER6C`, not allocated | - |
| `USE_ALT_RAM_MAP` | yes, for the PX4 bootloader | removed, ArduPilot bootloader only | - |
| IMU on `PC13` CS | ICM-42688-P or LSM6DSV | IIM-42653 (Invensensev3 driver, WHOAMI 0x56 is supported upstream) | IMU-01 schematic |
| IMU on `PC14`/`PC15` CS | BMI055 or BMI088 | BMI088 | IMU-01 schematic |
| IMU rotation | `PITCH_180_YAW_90` / `PITCH_180` | `ROLL_180` for both | IMU-01 orientation drawing |
| Barometers probed | MS5611 0x76 and 0x77, BMP388 0x77 | MS5611 0x76 only (CSB pulled high by R76) | baseboard page 8 |
| `HAL_BARO_ALLOW_INIT_NO_BARO` | yes | removed | - |
| Compasses probed | IST8310 0x0C, RM3100 | IST8310 0x0C, `ROTATION_NONE` (CAD0 and CAD1 pulled low) | IMU-01 schematic |
| `HAL_HEATER_MAG_OFFSET` | Holybro-measured offset | removed, must be measured on this board | - |
| Battery 1 | analog V+I, 18.18 V/V | voltage only by default, 17.12 V/V; INA168 is not populated | baseboard page 3 |
| Battery 2 | pins set in `defaults.parm` | pins and scales in `hwdef.dat`, disabled by default | baseboard page 3 |
| USB power flag | from `VBUS` | from LTC4417 `VALID3#` (`PE15 VBUS_nVALID`) | baseboard page 5 |
| Spektrum power | FMU pin `PH2` | removed; the IOMCU switches it (IO `PC13` into MIC5332 EN1) | baseboard pages 2 and 6 |

IOMCU: the STM32F103 has a 24 MHz crystal and its pinout matches ArduPilot's `iomcu/hwdef.inc`, so the stock `iofirmware_f103_lowpolh.bin` and the DShot variant are used, as on Pixhawk6C. IO `PB14` (the blue IO LED) is the IOMCU heater/heartbeat pin, so that LED blinks as the IO heartbeat.

## Hardware findings from the schematics

These came up while mapping the schematics to ArduPilot. They affect firmware behaviour or the documentation.

### Needs a hardware change

1. **IMU heater cannot reach full power (NASPIER_IMU-01, Q4).** The AO3400 N-channel MOSFET is on the high side: drain to VDD_5V, source to the two 51 Ω heater resistors. With a 3.3V gate it works as a source follower, so the resistors get about 2V (roughly 0.15 to 0.2 W instead of about 1 W). Fix: use a low-side switch. Put the resistors between VDD_5V and the drain, connect the source to GND, and move R6 so it pulls the gate down to GND. The sensor board README says "ready for production", so this is worth fixing first.
2. **SBUS output is not inverted (baseboard pages 9 and 14).** The RC IN connector pin 4 carries `IO_SBUS_OUTPUT`, the raw USART3 TX from the IOMCU. The inverted copy from U24 (`SBUS_OUT_RSSI_IN`) is not routed to any connector. SBUS is an inverted protocol and the STM32F103 cannot invert in hardware, so SBUS servos will not understand `BRD_SBUS_OUT`. Fix: route `SBUS_OUT_RSSI_IN` (U24 2Y) to RC IN pin 4.
3. **Check LTC4417 EN pin 22 (baseboard page 5).** It is marked no-connect. Confirm with the datasheet that leaving EN floating keeps the mux enabled.

### Documentation mismatches (repository README vs schematic)

1. **GPS1 pins 8 to 10**: the schematic has pin 8 = 3.3V (safety switch supply), pin 9 = Buzzer -, pin 10 = GND. This is the Pixhawk standard. The README says pin 8 = Buzzer, pins 9 and 10 = GND.
2. **Brick input is analog, not I2C**: CN7 pins 3 and 4 go to ADC pins `PA2` (BAT2_I) and `PB1` (BAT2_V). The README describes the Brick input as "I2C-based, reports voltage and current over I2C".
3. **RC IN pins 3 to 5**: the schematic has pin 3 = not connected, pin 4 = SBUS out, pin 5 = GND. The README says pin 3 = SBUS out, pins 4 and 5 = GND.
4. **Main and Aux output naming**: the README calls the FMU header "Main" and the IO header "Aux". ArduPilot (and PX4) always number the IOMCU outputs first, so SERVO1 to SERVO8 are on the header the README calls "Aux". Consider swapping the silkscreen labels to match Pixhawk 6C (IO = MAIN, FMU = AUX).
5. **"Independently switchable sensor rails"**: all four sensor LDO enables (S_EN_1 to S_EN_4) connect to one GPIO (`PB2`) through 0 Ω resistors. Firmware can only switch them together, unless the 0 Ω links are changed.

### Assumptions that need hardware confirmation

- **IMU and compass rotations** assume the sensor board is mounted top side up with its arrow pointing towards vehicle forward. The flex routing and mounting are not documented. Check with the HUD horizon, the accelerometer calibration and `COMPASS_AUTO_ROT`.
- **Heater gains** (`HAL_IMUHEAT_P_DEFAULT 50`, `HAL_IMUHEAT_I_DEFAULT 0.07`) come from Pixhawk6C. Tune them once the heater circuit is fixed.
- **Battery 2 scales** (18.18 V/V, 36.36 A/V) match a Holybro PM02-style module. Set them for whichever module is used.
- **Direct battery current scale** (20.08 A/V) is calculated for the INA168 option: 1 mΩ shunt, 249 kΩ gain resistor, 200 µA/V transconductance.

## Validation done

- `libraries/AP_HAL_ChibiOS/hwdef/scripts/chibios_hwdef.py` from ArduPilot master was run on both `hwdef.dat` and `hwdef-bl.dat`. Both generated `hwdef.h`, the DMA map and `ldscript.ld` with no errors. For this run only, a temporary board ID line and an empty bootloader file were added to the scratch copy of ArduPilot.
- The generated `hwdef.h` shows the expected probe lists: IIM-42653 and BMI088 with `ROTATION_ROLL_180`, MS5611 at 0x76, IST8310 at 0x0C. All UARTs have TX and RX DMA; the USART1, USART2 and USART3 TX streams are shared.
- `README.md` was checked with `markdownlint-cli2` and ArduPilot's config: 0 issues.

Not done: compiling the firmware, building the bootloader, and any hardware tests.

## Hardware test plan (from the ArduPilot porting guide)

- [ ] Boots and connects to a GCS over USB (Mission Planner, QGC, MAVProxy)
- [ ] GPS and compass detected; external compass on GPS1 and GPS2
- [ ] RC input detected and follows the transmitter: PPM, SBUS, DSM (including bind), CRSF on a UART
- [ ] HUD horizon follows board movement, testing each IMU on its own with `INS_USE`, `INS_USE2`
- [ ] IMU heater reaches `BRD_HEAT_TARG` and holds it
- [ ] Arms with the default arming checks
- [ ] Battery voltage on the Direct battery input matches a multimeter; Brick input voltage and current
- [ ] Each output (SERVO1 to SERVO16) drives a servo in MANUAL mode; DShot on the FMU outputs; `BRD_IO_DSHOT` on the IO outputs
- [ ] GPS moved to each UART in turn (with the other UARTs set to none) and works
- [ ] TELEM1 and TELEM2 work with a flow-control radio and `BRD_SER1_RTSCTS=1`, `BRD_SER2_RTSCTS=1`
- [ ] CAN1 and CAN2 with a DroneCAN device
- [ ] microSD logging, and firmware flashing from the SD card in the bootloader
- [ ] Safety switch, buzzer and LEDs
- [ ] Power flags: unplug and replug USB, Brick and battery while powered, check `POWER_STATUS`

## Trying it locally

On Linux or WSL with the ArduPilot build environment installed:

```bash
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot
cp -r <NASPIER_6C>/firmware/ardupilot/libraries/AP_HAL_ChibiOS/hwdef/NASPIER6C libraries/AP_HAL_ChibiOS/hwdef/
# local only: give the board a temporary ID so the hwdef scripts can resolve it
echo "AP_HW_NASPIER6C 7160" >> Tools/AP_Bootloader/board_types.txt
Tools/scripts/build_bootloaders.py NASPIER6C
./waf configure --board NASPIER6C
./waf copter
```

Output: `build/NASPIER6C/bin/arducopter.apj` and `arducopter_with_bl.hex`.
