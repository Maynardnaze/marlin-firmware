# Andrew's SKR Pro / MMU3 / Fixed-Time baseline

This branch preserves the existing custom configuration from the local
`Marlin-oldtc-SKR-Pro-v1.1` working tree. It is a reproducible source snapshot,
not a declaration that tuning is complete or that this exact binary is flashed.

## Build

```sh
pio run -e BTT_SKR_PRO
```

The default environment is `BTT_SKR_PRO`. The included
`ini/skr-pro-peripheral-test.ini` defines an optional external-I2C-EEPROM test
environment; it is not the default or an instruction to use unidentified EEPROM.
Generated artifacts, preparation output, and compile databases stay local.

## Preserved configuration

- BTT SKR Pro V1.1; five-slot Prusa MMU3 protocol on custom BMG/Mosquito hardware.
- TMC2209 X/Y/E; TMC2208 Z/Z2. X/Y/E default to SpreadCycle.
- Fixed-Time motion: X 2HEI at 63 Hz; Y MZV at 54 Hz.
- Bilinear automatic bed leveling, not UBL; custom probe and geometry retained.
- MMU firmware compatibility version 3.0.3; extruder sequence values 43.0 / 24.7 mm.
- Existing thermal, sensorless homing, display, and peripheral settings retained.

## Source defaults are not live EEPROM

The following differences were observed in user-supplied runtime reports and
are intentionally NOT silently reconciled in this baseline:

| Setting | Source default | Reported runtime / print behavior |
|---|---|---|
| X/Y run current | 1000 / 1050 mA RMS | M122/M503: 900 / 1000 mA set current |
| Junction deviation | 0.30 mm | M503: 0.30 mm; 0.03 mm proposed only as a temporary diagnostic |
| Probe Z offset | -1.50 mm | M503: -1.50 mm; previously inspected slicer startup overrides it with -1.65 mm |
| Linear Advance | K0 | M503: K0; inspected PLA tool-start G-code sets K0.022 |

Do not issue M502 or M500 merely to make the printer match this branch.
A build does not verify EEPROM, wiring, installed hardware, or flashed identity.

## Open commissioning issues

- Two-color/two-object prints have shown repeated X/Y layer shifts. Cause is not
  established; collision, mechanics, motion settings, and firmware interactions
  remain hypotheses.
- User reported a cleaner wipe tower with slicer Extra loading distance -23 mm.
  This is user-tested slicer tuning, not a firmware setting or a proven final value.
- Slicer parking/cooling reference remains 82 / 33 / 15 mm; provenance is an
  owner-relayed Bondtech/Mosquito reference, not an independently verified publication.
- The raw MMU3 protocol trace is not gated by MMU_DEBUG in this source. No logging
  suppression patch is included in this baseline.
- Removal of the slicer's automatic M1 S10 priming pause is a separate PrusaSlicer
  G-code-substitution change; no slicer preset is bundled here.

## Verification at baseline publication

`pio run -e BTT_SKR_PRO` completed successfully before publication.
Reported RAM use: 47,696 / 196,608 bytes (24.3%); flash use:
294,000 / 1,015,808 bytes (28.9%). This verifies compilation, not physical operation.
Existing compiler warnings were retained: automatic Z2 pin assignment, the FT
M900 K advisory, and MMU_VERSION macro redefinition. No warning-suppression or
behavioral fix is folded into this snapshot. The staged diff passed whitespace
checks and a targeted credential-pattern scan (not a comprehensive security audit).
No firmware flash, motion test, or EEPROM write was performed for publication.

## Change workflow

Preserve this baseline before experiments. Use a focused branch/commit for each
firmware change; build the default environment and review the diff before pushing.
Record physical test results separately from compile success. Keep runtime-only
experiments and unresolved hypotheses labeled; do not promote them to defaults
without a deliberate decision. Never commit credentials, EEPROM dumps containing
private data, or unrelated commissioning artifacts.
