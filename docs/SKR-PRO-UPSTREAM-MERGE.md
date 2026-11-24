# Official bugfix integration for the custom SKR Pro

## Provenance and scope

- Preserved baseline: `29576d2d31d33104c0df4996b59f35621bc0fa8a`
  (`printer/skr-pro-mmu3-ft`).
- Official repository: https://github.com/MarlinFirmware/Marlin
- Resolved branch: `bugfix-2.1.x`
- Pinned integration target: `341a0d3b1948892e5aa07b64b96b795d8030f3bf`.
- Integration branch: `printer/skr-pro-mmu3-ft-upstream`.
- This is a real Git merge in a separate worktree. The original working folder,
  published baseline branch, and its generated artifacts are not replaced.

## Conflict resolution

- `Configuration.h`: retain upstream documentation/new feature sections while
  preserving custom values and the guarded external-EEPROM configuration. Remove
  a duplicated, disabled E_DUAL_STEPPER_DRIVERS block produced by the merge.
- `Configuration_adv.h`: retain X sensorless threshold 81 and Y threshold 85;
  accept upstream explanatory comment updates.
- `mmu3_supported_version.h`: keep minimum version 3.0.3. Upstream's new version
  policy requires matching major/minor and accepts patch levels at or above the
  minimum. This preserves compatibility with 3.0.3 but is not the previous exact
  patch-match policy.
- Preserve `BTT_SKR_PRO` as the default build and keep its optional peripheral-test
  INI include. Accept upstream build-system/toolchain updates as part of the merge.
- No current, junction-deviation, temperature, probe-offset, MMU geometry, or shaper
  tuning experiment is introduced here.

## Behavior changes to recognize

A merge can change behavior even when every configuration value is preserved.
Upstream includes FT 2HEI/MZV shaper fixes, implemented minimum-cruise-ratio handling,
MMU load-to-nozzle compensation fixes, reduced repetitive MMU logging, runout/error
handling fixes, and an STM32 build framework/toolchain upgrade. These are upstream
changes, not evidence that the printer's unresolved layer shifts are repaired.
`FTM_MINIMUM_CRUISE_RATIO` remains 0.5, but upstream now implements that setting in
the trapezoidal trajectory generator; the baseline generator did not apply it.
The LCD/explicit load-to-nozzle sequence now subtracts the MMU extra-load offset
from the first physical extrusion move instead of only adjusting the planner E
coordinate. The 43.0 / 24.7 mm macros are unchanged, but the effective movement may
change; recommission explicit/LCD loading conservatively before ordinary printing.
This does not establish the current slicer loading distance as correct for the new build.

## EEPROM / flashing warning

**Do not flash this build without recording the current M503 and M493 output and
planning how to restore the desired runtime calibration.**
The preserved internal-flash configuration enables `EEPROM_INIT_NOW` and
`EEPROM_AUTO_INIT`. A new build can initialize saved settings on first boot.
The EEPROM schema also changes from V90 to V91, so the old stored layout is not
compatible even aside from the build-hash initialization policy. Source
preservation does not guarantee EEPROM/runtime calibration survives. Runtime X/Y
current (900/1000 mA) differs from source defaults (1000/1050 mA); plate Z offset
and filament K can also be changed by G-code. Do not use M502 or M500 blindly.

No flashing, EEPROM write, or physical printer test is authorized by this merge.

## Validation interpretation

Header-retention checks compare all active textual #define entries in both headers
against the baseline, including definitions in conditional sections. They are not a
substitute for compiler preprocessing, firmware compilation, or hardware validation.
## Validation results

- `pio run -e BTT_SKR_PRO`: SUCCESS, 503.288 seconds for the fresh worktree,
  including dependency installation. RAM 48,852 / 196,608 bytes (24.8%);
  flash 298,256 / 1,015,808 bytes (29.4%).
- `pio run -e BTT_SKR_PRO -t compiledb`: SUCCESS, 28.453 seconds. This is
  compilation-database generation, not a second firmware build.
- Header retention: baseline's 508 Configuration.h macro names and 877
  Configuration_adv.h macro names retain their textual values. Resolved-header
  markers and Git unmerged-path checks are clear.
- Effective GCC preprocessing using the generated FT translation-unit command:
  FT motion/default enabled; X 2HEI 63 Hz, Y MZV 54 Hz; bilinear enabled,
  UBL/standard X/Y shaping absent; five MMU tools, 115200 baud, 43.0/24.7 geometry.
  Internal flash EEPROM and both automatic initialization flags are effective.
- ELF verification: M493, FT menu, M701, M702, and MMU3 tool-change symbols present;
  legacy M593 absent. Vector table address is 0x08008000.
- Independent bounded source review: no concrete configuration/source blocker;
  EEPROM and changed MMU behavior are commissioning advisories, not resolved risks.
- Custom paths pass staged whitespace checks. A whole-merge whitespace check
  reports trailing whitespace in untouched upstream files; those are not reformatted.
- Remaining compiler warnings: automatic Z2 pin assignment and FT K uses M900.
  The baseline's MMU_VERSION redefinition warning is not present in this build log.
- Final tree differs from pinned upstream only in the existing seven baseline
  customization/documentation/build paths and this integration note.

These are source/build checks, not a claim of working printer motion or a fix for
layer shifts. Generated binaries, ELF and compilation databases are not committed.
A separate local artifact archive records the exact source commit and SHA-256 values.
