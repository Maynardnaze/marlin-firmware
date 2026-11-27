# Standard-motion ZV and junction-deviation diagnostic variant

## Purpose and provenance

Parent: `6b84647b20033ce8f5c2be6646dc0368da5a124b`, official upstream target
`341a0d3b1948892e5aa07b64b96b795d8030f3bf`.

The user's corrected observation is that a clean print had M493 S0 in its start
G-code; M493 S1 still produces layer shifting. This motivates a separate standard
motion diagnostic, not a claim that the precise FT defect is identified. The
front-right pause was attributed by the user to a loose filament sensor, now fixed.

## Changes

- Disable FT_MOTION in this variant only; preserve the prior FT worktrees/branches.
- Enable standard X/Y ZV input shaping and the LCD shaping menu.
- Default X/Y frequencies to zero (shaping initially OFF).
- Set SHAPING_MIN_FREQ to 20 Hz for explicit buffer sizing with zero defaults.
- Keep stock standard damping 0.15 as a provisional value, not a measurement.
- Preserve Configuration.h byte-for-byte apart from Git line-ending conversion:
  hardware, bilinear leveling, calibration, current defaults, and JD default 0.30
  are not silently retuned. No MMU geometry/source or slicer presets changed.
- Use M593 for this variant. M493 is not compiled; remove both M493 startup lines
  from a separately named standard-motion slicer preset. Do not accidentally use
  that preset to enable FT on the comparison build.

## Controlled calibration sequence (not yet physically executed)

1. Record current M503 and M493 separately BEFORE any user-authorized flash.
   This new build's EEPROM_INIT_NOW and automatic initialization remain enabled;
   runtime settings can reset. Restore the intended calibrated settings explicitly
   before motion, including the correct plate offset and X/Y currents.
2. With standard firmware, read M503 and M593 separately. Confirm no motion is
   queued before setting M593 F0. Both shaping frequencies must report zero.
3. First JD candidate: M205 J0.03 (millimeters). This is a conservative experiment,
   not a calibrated value or proof the previous 0.30 caused the shifts. Use the
   same geometry/material/speed/acceleration, with M593 F0 throughout. Do not M500.
4. Compare fixed-JD prints at 0.02, 0.03, 0.04 and, only if still clean, 0.05. Do not
   change current/acceleration/loading distances concurrently. Inspect corner
   ringing, rounding/bulges, and position loss. Select a clean repeatable result,
   not the largest accepted number. Corner bulges can also involve flow/advance.
   Keep filament K unchanged during the first comparison.
5. Hold JD and acceleration fixed while tuning ZV one axis at a time. Existing FT
   2HEI 63 Hz / MZV 54 Hz are optimizer settings for different shapers, not proven
   standard-ZV optima. Prefer the retained raw resonance spectra if available;
   reacquire only if evidence is missing or the mechanics/mount changed. Rigid
   toolhead mounting for X and bed mounting for Y on this bed-slinger are required.
   M593 X F<hz> D<zeta> and M593 Y F<hz> D<zeta> set axes independently; M593 reports.
6. Validate with a fixed-frequency print before a separately authorized M500.
   A source build and a test plan cannot establish calibrated physical values.

A candidate JD can be tried on the current FT-capable firmware WITHOUT flashing:
when idle, issue M400, M493 S0, M205 J0.03, and M205 on separate lines. The actual
print file must keep M493 S0 and contain no M205/EEPROM reload overrides. Its last
supplied v5 export uses PLA 215/60, plate offset -1.625 and extra loading -30;
retain these deliberate file values unless the user supplies a newer baseline.

## Build invocation

Use BTT_SKR_PRO. The isolated build reuses the existing pinned dependency cache:

`PLATFORMIO_LIBDEPS_DIR=C:/Users/AndrewMaynard/Downloads/Marlin-SKR-Pro-upstream-merge/.pio/libdeps pio run -e BTT_SKR_PRO`

The cache override does not change firmware settings. Local artifacts and a
manifest record the actual build result. No flashing, EEPROM writes or test
motions were performed by the assistant.

## Verification

- Fresh BTT_SKR_PRO build: SUCCESS, 215.598 s. RAM 47,888 / 196,608 bytes
  (24.4%); flash 281,272 / 1,015,808 bytes (27.7%).
- Compilation database generation: SUCCESS, 21.866 s; not a firmware build.
- Effective GCC preprocessing: FT absent, standard X/Y shaping enabled, X/Y
  frequencies 0, damping 0.15, buffer minimum 20 Hz; bilinear and MMU retained;
  CLASSIC_JERK absent, junction-deviation source default remains 0.30 mm.
- ELF: M593, menu_advanced_input_shaping, MMU3 toolchange, M701 and M702 present;
  M493 absent. Vector table remains 0x08008000.
- Failing-before / passing-after feature acceptance test and whitespace check pass.
- The only firmware-file difference from the parent is Configuration_adv.h.
- These checks do not identify the cause of FT shifts or establish physical calibration.
