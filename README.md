# RP2040 Minimal Development Board

English 

A minimal RP2040 board with a USB-C connector, designed in KiCad on a 4-layer PCB. This is my first SMD PCB design. I built it by following the "RP2040 Minimal Circuit Design" guide on [schemalyzer.com](https://www.schemalyzer.com/en/blog/microcontrollers/rp2040/minimal-circuit-design).

## Status

> **The board has not been manufactured or tested yet.**

The design currently passes KiCad's **ERC** (schematic) and **DRC** (PCB) checks only. Nothing has been verified on real hardware, so please run your own checks before ordering this design.

### Known DRC exceptions

A few DRC items are excluded on purpose:

- **USB-C connector (4 items):** The distance between the connector's two locating holes (NPTH) and the neighboring GND pads is 0.1944 mm, while my rule is 0.25 mm. This value comes from the connector footprint's own geometry. It is close to the NPTH clearance a PCB fab states, so it should be checked in the fab's DFM review before ordering.
- **Board edge (1 item):** A pad on the front copper layer (F.Cu) is 0.4 mm from the board edge, while my rule is 0.5 mm. This is above the fab's minimum for outer layers.

## Features

- **MCU:** RP2040 (QFN-56)
- **Flash:** Winbond W25Q32JVSS (4 MB, QSPI)
- **Clock:** 12 MHz crystal with two 33 pF load capacitors
- **Power:** USB-C VBUS (5 V) feeding an XC6206-series 3.3 V LDO. The LDO has a low current limit, so check its datasheet if you plan to power external loads.
- **USB:** USB-C connector, 27 Ω series resistors on D+/D−, separate 5.1 kΩ resistors on CC1 and CC2
- **Buttons:** BOOTSEL and RESET
- **Debug:** 3-pin SWD header
- **PCB:** 4 layers (F.Cu / In1.Cu GND plane / In2.Cu +3.3 V plane / B.Cu), 0603 passives

## Programming

Hold the BOOTSEL button while plugging the board into USB (or hold it while pressing RESET). The board appears as a USB mass-storage device, and you can drag and drop a `.uf2` file onto it. This flow has not been tried on a real board yet.

## Broken-out pins

Only **11 of the RP2040's 30 GPIOs** are routed to the headers. The remaining GPIOs are not connected and are left open for anyone who wants to extend the design (the corresponding header pins are unused).

Header pin numbers are the pad numbers on the PCB (pin 1 has a square pad).

**Top header**

| Header pin | GPIO |
|---|---|
| 9 | GPIO6 |
| 10 | GPIO5 |
| 11 | GPIO4 |
| 12 | GPIO3 |
| 13 | GPIO2 |
| 14 | GPIO1 |
| 15 | GPIO0 |

**Bottom header**

| Header pin | GPIO |
|---|---|
| 12 | GPIO20 |
| 13 | GPIO21 |
| 14 | GPIO25 |
| 15 | GPIO22 |

### SWD (debug) header

| Pin | Signal |
|---|---|
| 1 | SWCLK |
| 2 | GND |
| 3 | SWD (SWDIO) |

The pin order was chosen to match the Raspberry Pi Debug Probe cable. Please verify this against the probe's documentation before connecting it.

## Notes and verification against the datasheet

I compared the design with the RP2040 datasheet and Raspberry Pi's "Hardware design with RP2040" document:

- **Power and decoupling:** 100 nF on each IOVDD (6) and DVDD (2) pin, on USB_VDD and on ADC_AVDD, 1 µF on VREG_VIN and VREG_VOUT, TESTEN tied to GND and 27 Ω series resistors on USB_DP/USB_DM all follow the datasheet's requirements.
- **QSPI pin numbers:** This project uses the numbers from the datasheet pin table (QSPI_SD3 = 51, SCLK = 52, SD0 = 53, SD2 = 54, SD1 = 55, SS = 56, GND pad = 57). Most QSPI pin numbers in the guide's flash table do not match the datasheet (it lists QSPI_SS as pin 57, for example), and some pin ranges in its GPIO table are off as well.
- **Crystal circuit (deviation from the reference):** Raspberry Pi's reference design uses the Abracon ABM8-272-T3 (12 MHz, CL = 10 pF, ESR up to 50 Ω), two 15 pF load capacitors and a 1 kΩ series resistor on XOUT, and states that any deviation needs extensive testing. This board follows the guide instead: a 20 pF crystal with 2 × 33 pF load capacitors and no series resistor. The load capacitance math is consistent (33 pF / 2 + 3 pF stray ≈ 19.5 pF), but this is not the reference circuit and has not been tested.
- **Load capacitor values:** The guide's text and BOM disagreed (33 pF vs. 27 pF). This project uses 33 pF for a 20 pF crystal.
- **QSPI_SS pull-up:** There is no external pull-up on QSPI_SS. Raspberry Pi's reference design leaves it unpopulated for the W25Q flash, but it may matter if you use a different flash.

## Possible improvements

- Break out the remaining GPIOs to the headers
- ESD protection on the USB lines and protection on VBUS
- Test points and mounting holes
- Match Raspberry Pi's reference crystal circuit (ABM8-272-T3, 2 × 15 pF, 1 kΩ series resistor on XOUT), or characterize the current one over temperature
- A 3.3 V regulator with a higher current rating
- A BOM and production files (Gerber, pick-and-place)

## Tools

- KiCad (schematic and PCB)

## License

This work is dedicated to the public domain under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/). You can copy, modify and use it for any purpose, including commercially, without asking for permission. See the `LICENSE` file.

This design is provided **as is, without warranty of any kind**. The author takes no responsibility for any damage or loss resulting from its use. Build and use it at your own risk.
