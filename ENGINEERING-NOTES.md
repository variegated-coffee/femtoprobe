# Femtoprobe Engineering Notes

Things a carrier-board designer or firmware author needs to know when using the Femtoprobe (R2A). How to build and bring up the module is in [MANUFACTURING-NOTES.md](MANUFACTURING-NOTES.md).

## Pinout

The module has eight castellated pins, four on each short edge, on a 2.54 mm pitch. Positions below are viewed from the top (component side), with the "Femtoprobe" silkscreen text upright: J3 is on the left edge and J4 on the right.

"Carrier pad" is the pad number in the carrier-side `Femtoprobe` symbol and footprints (see [Mounting on a carrier](#mounting-on-a-carrier)).

| Edge | Position | Module pin | Carrier pad | Signal |
| ---- | -------- | ---------- | ----------- | ------ |
| Left | 1 (top) | J3 pin 4 | 1 | GND |
| Left | 2 | J3 pin 3 | 2 | USB D− |
| Left | 3 | J3 pin 2 | 3 | USB D+ |
| Left | 4 (bottom) | J3 pin 1 | 4 | +3V3 (supply input) |
| Right | 1 (top) | J4 pin 1 | 5 | UART RX (probe input) |
| Right | 2 | J4 pin 2 | 6 | UART TX (probe output) |
| Right | 3 | J4 pin 3 | 7 | SWDIO |
| Right | 4 (bottom) | J4 pin 4 | 8 | SWCLK |

The front silkscreen labels the groups (GND, USB, 3V3, UART, SWD); the back silkscreen labels each pin.

## Power

* **Supply:** 3.3 V on +3V3 (J3 pin 1). The module has no regulator; the RP2040, its USB PHY and the flash all run directly from this rail.
* **Decoupling on the module:** ten 100 nF and four 1 µF capacitors. There is no bulk capacitance; provide it on the carrier.
* **Core supply:** the RP2040's internal regulator makes the 1.1 V core rail (VREG_VOUT to DVDD) on the module. It isn't brought out. The `LV_PWR_DIST` netclass applies to it.

## USB

* D+ is on J3 pin 2 and D− on J3 pin 3, each through a 27 Ω series resistor (R1, R2) on the module.
* The interface is USB full speed (12 Mbit/s).
* There is **no ESD protection and no VBUS sensing on the module**. Put ESD protection on the carrier at the connector, and route D+/D− as a 90 Ω differential pair.

## SWD and UART

* The target-side signals all have 100 Ω series resistors on the module (R4–R7).
* They are **3.3 V logic with no level shifting**. A target running at another I/O voltage needs level shifters on the carrier.
* UART directions are from the probe's point of view: connect UART TX to the target's RX, and UART RX to the target's TX.

| Signal | Module pin | RP2040 GPIO | Series resistor |
| ------ | ---------- | ----------- | --------------- |
| UART RX | J4 pin 1 | GPIO21 (UART1 RX) | R7 |
| UART TX | J4 pin 2 | GPIO20 (UART1 TX) | R6 |
| SWDIO | J4 pin 3 | GPIO11 | R5 |
| SWCLK | J4 pin 4 | GPIO10 | R4 |

## RP2040

* **Flash and clock:** 2 MB Winbond W25Q16JVUXIQ flash and a 12 MHz crystal (12 pF load capacitors), the same parts as on the Raspberry Pi Pico.
* **Reset:** RUN has a 4.7 kΩ pull-up (R3). The RESET pads (SW1) short RUN to GND.
* **Bootloader:** the BOOT pads (SW2) short QSPI_SS to GND. Hold them shorted while releasing reset, or while powering up, to start the USB bootloader.
* **The RP2040's own SWD port isn't connected.** The module is programmed over USB only.
* **GPIO usage:** only GPIO10, GPIO11, GPIO20 and GPIO21 are used. All other GPIOs, including GPIO25, are unconnected.

## Using it with the APEC SoM

The Femtoprobe is designed for [APEC SoM](https://github.com/variegated-coffee/apec-som) carrier boards. On such a carrier, wire it up like this (APEC pin numbers are the module's castellation numbers, from the APEC SoM's ENGINEERING-NOTES.md):

* **USB:** connect D+/D− to one of the APEC SoM's two spare USB hub downstream ports: USB_PORT3 (D+ on pin 91, D− on pin 92) or USB_PORT4 (D+ on pin 94, D− on pin 95). The Femtoprobe then shows up on the same USB connection as the rest of the controller, so one cable is enough.
* **SWD:** connect SWDIO to APEC pin 67 and SWCLK to APEC pin 68, the RP2350's SWD port.
* **Power:** feed +3V3 from the APEC SoM's +3V3 rail (pins 76–77).
* **UART (optional):** connecting it to the ESP32-C6's UART0 is recommended. That puts the ESP32's serial console on the same USB connection. Cross the lines: Femtoprobe UART TX to ESP32_U0RXD (APEC pin 3), and Femtoprobe UART RX to ESP32_U0TXD (APEC pin 4).

## Mounting on a carrier

* The module is meant to be fitted on a pin header during development and removed when it's no longer needed.
* The carrier-side parts are in the `APEC_SoM` library of the [APEC SoM](https://github.com/variegated-coffee/apec-som) repository: the `Femtoprobe` symbol in `APEC_SoM.kicad_sym`, plus two footprints in `APEC_SoM.pretty`. They use the carrier pad numbers in the table above.
  * `Femtoprobe`: through-hole footprint for 2.54 mm pin headers, so the module can be plugged in and removed.
  * `Femtoprobe SMT`: land pattern for soldering the castellations directly to the carrier.
* Each pin has a second 1.0 mm plated hole 2.54 mm in from the edge, in the same copper land as the castellation. These inner holes sit on a 2.54 mm grid with the two rows 22.86 mm apart, which is what lets the module sit on ordinary pin headers.
* The underside has the bottom ends of the castellation and pin holes exposed. Keep other exposed carrier copper out from under the module.
