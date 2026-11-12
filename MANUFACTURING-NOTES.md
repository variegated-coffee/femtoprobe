# Femtoprobe Manufacturing Notes

How to fabricate, assemble and bring up the Femtoprobe, revision R2A. All values below come from the R2A design files (KiCad 10). Order parameters are given in JLCPCB's terms, since the project's production files are set up for JLCPCB, but any fab and assembler with the same capabilities will do.

For the pinout and carrier design, see [ENGINEERING-NOTES.md](ENGINEERING-NOTES.md).

## PCB fabrication

| Parameter | Value |
| --------- | ----- |
| Layers | 4 |
| Size | 27.93 × 12.13 mm |
| Thickness | 1.6 mm |
| Material | FR-4 |
| Copper weight | 1 oz on all layers |
| Surface finish | ENIG |
| Castellated holes | **Yes**, on both short edges |
| Vias | 47, through-hole only, 0.6 mm pad and 0.3 mm drill |
| Smallest hole | 0.3 mm |
| Smallest track | 0.2 mm |
| Smallest clearance | 0.15 mm |
| Impedance control | Not specified in the design |

**Stackup** (from the board file, top to bottom):

| Layer | Use | Thickness |
| ----- | --- | --------- |
| F.Cu | Components, signals | 35 µm |
| Prepreg | | 0.1 mm |
| In1.Cu | Power routing | 35 µm |
| Core FR-4 | | 1.24 mm |
| In2.Cu | Ground | 35 µm |
| Prepreg | | 0.1 mm |
| B.Cu | Signals | 35 µm |

This is KiCad's generic 4-layer, 1.6 mm stackup. Nothing in the design depends on the dielectric thicknesses, so the fab's standard 4-layer 1.6 mm stackup will do.

**Castellations.**
* J3 and J4 are the eight edge pins, four on each short edge.
* Each pin has a 1.0 mm plated hole centred on the board edge, which becomes a plated half-hole when the outline is milled. A second 1.0 mm plated hole sits 2.54 mm further in, in the same copper land.
* Ask the fab for castellated (plated half) holes. The pads aren't marked as castellated in the design files, so the Gerbers won't flag them on their own.
* ENIG is the surface finish JLCPCB [recommends for castellated boards](https://jlcpcb.com/blog/castellated-pcbs-introduction-and-design-requirements).
* A custom rule in `femtoprobe.kicad_dru` exempts the J3/J4 pads from the copper-to-edge clearance check.

**Tooling holes.**
* H1 and H2 are 1.152 mm non-plated holes in opposite corners, matching JLCPCB's assembly tooling-hole size.
* They exist only on the PCB, not in the schematic, and are excluded from the BOM and placement files.

## Assembly

* **Top side only.** The bottom has no components, only routing and silkscreen (logo, URL and pin labels), so no bottom-side assembly or stencil is needed.
* **Parts:** 26 to place, all with LCSC part numbers in the schematic.
  * U1: RP2040
  * U2: W25Q16JVUXIQ flash
  * Y2: 12 MHz crystal
  * 16 capacitors and 7 resistors, all 0402
* **Not placed:** J3 and J4 (the castellations), SW1 (RESET) and SW2 (BOOT). SW1 and SW2 are pairs of 1.0 mm pitch through-hole pads, meant to be bridged with tweezers. None of these four have LCSC numbers; leave them out of the assembly order.
* **Finest features:**
  * RP2040 (U1): QFN-56, 0.4 mm pitch, with an exposed pad
  * W25Q16JVUXIQ (U2): USON-8, 3 × 2 mm, 0.5 mm pitch, with an exposed pad
  * Y2: 2.5 × 2.0 mm crystal
  * 0402 passives
* **Production files:** generated with the JLCPCB tools plugin for KiCad, which writes them to `jlcpcb/` (not tracked in git).
  * The plugin takes LCSC numbers from the footprints on the PCB. Sync them from the schematic first; see the checklist below.
* **Order number:** the back silkscreen has a `JLCJLCJLCJLC` marker, which JLCPCB replaces with its order number. Other fabs will print it as it is, so remove it before ordering elsewhere.
* **Rotations:** check the placement preview for rotation errors before confirming the order, especially U1, U2 and Y2.

## Before ordering

1. **Sync part numbers:** in the PCB editor, run Tools → Update PCB from Schematic with "Update footprint fields" enabled, so the LCSC numbers on the footprints match the schematic.
2. **DRC:** refill all zones and run DRC. It should report no errors and no unconnected items. The remaining warnings are known:
   * silkscreen text below the 0.8 mm minimum;
   * the J3/J4 footprints differing from the library;
   * a few thermal-relief and silkscreen-clipping warnings.
3. **Outputs:** generate the production files. Check the castellation half-holes in a Gerber viewer before uploading.
4. **Order notes:** mention castellated holes in the order notes, and choose ENIG.

## Bring-up and programming

The module has no connectors of its own. Power and program it through a carrier board, or a fixture that contacts the castellations. Pin numbers below are module pins; see the pinout in ENGINEERING-NOTES.md.

1. **First power-up.** Feed 3.3 V into +3V3 (J3 pin 1) from a current-limited supply, with GND on J3 pin 4.
2. **USB.** Connect D+ (J3 pin 2) and D− (J3 pin 3) to a USB host.
3. **Bootloader.** Short the BOOT pads (SW2) while powering up, or while shorting and releasing the RESET pads (SW1). The RP2040 then enumerates as a USB mass-storage bootloader, and firmware can be copied to it as a UF2 file.
   * The flash and crystal are the same as on the Raspberry Pi Pico, so the Pico's flash and clock settings work unchanged. Probe firmware still has to use the module's own pin assignments; see ENGINEERING-NOTES.md.
   * The RP2040's own SWD port isn't connected, so USB is the only way to program the module.
