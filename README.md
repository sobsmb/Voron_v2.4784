
# Voron V2.4 Configuration (4784)

Configuration repository for my **Voron V2.4 (V2.4784 / Blue)** running **Klipper**, with **Happy Hare v4.2.0** and a **12-gate ERCF v2 MMU**.

This repository stores the printer configuration, macros, and supporting service configuration used on the machine.

---

# Printer Details

| Component | Description |
|-----------|-------------|
| Printer | Voron V2.4 |
| Firmware | Klipper |
| MMU | 12-gate ERCF v2 with Happy Hare v4.2.0, Thumper Blocks, and servo-less Thumper bearing modification |
| Host | Raspberry Pi running MainsailOS |
| UI | Mainsail / KlipperScreen |
| Probe | Voron TAP |
| Chamber Filtration | Nevermore |
| Airflow | Intake / Exhaust / Cooling fans |

---

# Repository Layout

```
printer.cfg                Main Klipper entrypoint

hardware/                  Printer hardware configuration
features/                  Optional features (bed mesh, sensors, etc)
tuning/                    Tuning files (input shaper, accelerometer)
macros/                    Organized macros

services/                  Host-side service configuration
archive/                   Old / unused configs kept for reference
artifacts/                 Runtime generated files (ignored by git)

mmu/                       Customized Happy Hare v4.2 configuration and saved calibration
blue-v4-cutter.cfg         Blue-specific lift / native cut / Z-return overlay
```

### Important Notes

* The **`mmu/` directory is customized** for Blue's ERCF v2 hardware and patched Happy Hare v4.2 installation. Do not blindly update or regenerate it; preserve the local adaptations described below.
* Runtime files such as logs and Moonraker databases are excluded via `.gitignore`.
* The repository is automatically backed up to GitHub using the `BACKUP_CFG` macro.

---

# Happy Hare v4.2 / ERCF v2

Happy Hare **v4.2.0** is pinned to commit
`1b11270cbc16e9bc7921e8dad0fefdcc9577f074` with local adaptations for Blue.
The active source checkout is `/home/pi/Happy-Hare-v4.2-staging/`; despite its
name, it is the installed runtime source, not an unused preview. Its local
changes are not all committed upstream.

## Hardware and local adaptations

- **ERCF v2, 12 gates, Thumper Blocks**, with the
  [servo-less Thumper bearing modification](https://www.printables.com/model/1555814-ercf-v2-thumper-block-bearing-mod).
  This remains an ERCF v2 configuration, not Jack Rabbit / ERCF v3.
- The local `MOD_THUMPER_BEARING_SELECTOR` option uses `LinearSelector` with
  `filament_always_gripped=1`; no selector servo is generated. The gantry cutter
  and Blobifier tray retain their separate accessory servos.
- M220 speed-override handling is retained in the Moonraker preprocessing code.
- Encoder free-spin detection during final unload parking retains one bounded
  local recovery retry rather than silently declaring a successful unload.
- `blue-v4-cutter.cfg` wraps the native v4 cutter: lift Z by up to **10 mm**
  within available travel, perform the native cut, then return to the pre-cut Z.
  It adds no custom XY parking; the native cutter still performs its required
  XY approach and cut moves.
- Blue's existing custom PAUSE / CANCEL_PRINT macro precedence is preserved.

Review and preserve these configuration and source changes before updating
Happy Hare or regenerating its configuration. A stock upstream checkout is not
a drop-in replacement for this locally patched installation.

## Supervised validation and calibration

Post-upgrade checks completed successfully:

- Selector direction, home-switch press/release, and MMU homing.
- Selection, filament feeding, and parking at all **12 gates**.
- Physical 100 mm gear-travel checks on gates 0 and 1, followed by encoder-based
  gear calibration of gates 2-11 using the corrected reference.
- Fresh encoder and sensor-based Bowden calibration with the output tube connected.
- Blobifier tray extension/retraction, gantry-pin operation, and cold cutter actuation.
- Full XYZ homing, QGL, and Z re-homing; separate Z lift/return verification.
- Heated gate 0 load, Blobifier purge and brush cleaning, cutter lift/cut/Z-return,
  and unload/parking with motor-current restoration verified.

Saved runtime calibration is in `mmu/mmu_vars.cfg`. Bowden length and encoder
clog-detection values can change through calibration and autotuning; use the live
saved values rather than treating a historical measurement as fixed. Saving a
gate gear correction can also automatically rescale the shared Bowden length;
remeasure it after completing a set of gear calibrations.

The encoder's exposed filament-switch entity indicates **recent encoder motion**,
not static filament presence. Its idle state alone does not prove that filament
has cleared the selector; use the completed parking routine and appropriate checks.

## Backup and rollback

The original v3 source and pre-upgrade rollback copies are preserved separately.
A private post-validation archive also preserves the complete printer configuration,
patched v4.2 checkout (including local/untracked files), saved calibration, installed
Happy Hare module links, original v3 source, and existing v3 rollback files. Its
SHA-256 was verified on Blue and on an off-printer copy.

GitHub configuration backups are **not** a complete replacement for that private
archive: the active Happy Hare source lives outside this repository. Do not publish
private archives, credentials, or source/configuration snapshots without reviewing
their contents. This backup is not a full system-disk image.

Restoration requires stopping Klipper and Moonraker, restoring matching configuration
and source/link targets, and verifying them before a supervised restart. Accessory
servos may move during startup. Do not blindly mix v3 and v4 files or use upstream
uninstall as a rollback procedure.

---

# Hot Standby & Fast Restart System

This printer includes a **Hot Standby system** designed to drastically reduce the startup time between prints while maintaining safety.

Key features include:

- Hot Standby Mode
- Fast Restart (skip homing and QGL)
- TAP sanity probe validation
- Automatic standby timeout shutdown
- Chamber air purge safety cycle
- MMU‑friendly print ending
- Material‑aware fan behavior

---

# Hot Standby Mode

Hot standby allows the printer to remain in a **ready‑to‑print state** after a print finishes.

This is intended for workflows where multiple prints are started in sequence using the same material.

Hot standby is controlled with:

```
HEAT_HOLD_ON
HEAT_HOLD_OFF
```

When enabled, the printer enters standby automatically at the end of a print.

---

# Hot Standby Behavior

When **heat hold is enabled** and a print finishes the printer will:

- Turn **off the hotend**
- Keep the **bed temperature unchanged**
- Leave **Nevermore / intake / exhaust fan state unchanged**
- Leave the **toolhead where MMU_EJECT places it**
- Keep **steppers enabled**
- Perform a **TAP probe at bed center**
- Store the probe Z value
- Mark standby as **ready**
- Start a **60 minute standby timer**

This preserves the printer in a stable state so the next print can start quickly.

---

# Fast Restart System

If a new print begins while the printer is in hot standby the startup routine will attempt a **fast restart**.

Fast restart is allowed only when:

- Heat hold is enabled
- Standby is marked ready
- The material matches the previous print
- All axes are still homed
- The TAP sanity probe passes

If these conditions pass, the printer skips the heavy startup sequence.

---

# What Fast Restart Skips

Fast restart skips:

- Homing
- Quad Gantry Level (QGL)
- Nozzle cleaning
- Bed mesh reload
- Fan reconfiguration

The following still occur:

- Bed temperature set / wait
- Nozzle temperature set / wait
- Flow rate configuration
- Normal print start

This allows extremely fast restarts when printing multiple parts in sequence.

---

# TAP Sanity Probe

Before allowing fast restart the printer performs a **single TAP probe** at bed center.

The new probe result is compared to the stored standby probe value.

Tolerance:

±0.03 mm

If the difference exceeds this value, fast restart is **aborted** and a full startup sequence runs instead.

This protects against:

- Gantry drift
- Manual movement
- Belt relaxation
- Stepper disable
- Physical bumps to the printer

---

# Standby Safety Timer

Hot standby automatically starts a **60 minute timer**.

If no print begins before the timer expires the printer performs a safe shutdown.

Shutdown sequence:

1. Turn off bed heater
2. Turn off hotend heater
3. Turn off Nevermore
4. Run chamber air purge
5. Clear standby state
6. Disable heat hold

---

# Chamber Air Purge

When the standby timeout expires the printer runs a short purge cycle.

Fans activated:

- Intake fan
- Exhaust fan

Duration:

180 seconds

After the purge:

- Fans turn off
- Steppers disable

---

# CANCEL_PRINT Behavior

`CANCEL_PRINT` always performs a **full safe shutdown**.

Cancel intentionally **does not use hot standby**, since cancel usually indicates a failure or user interruption.

Cancel will:

- Clear standby state
- Cancel standby timers
- Eject filament
- Turn off heaters
- Lift Z
- Run enclosure fan cooldown timers
- Reset display
- Return printer to ready state

---

# Probe Reference Storage

When entering hot standby the printer performs a probe and stores:

```
HOT_STANDBY.last_probe_z
```

This value is used during fast restart sanity checks.

If standby ends due to timeout or cancel this value is cleared.

---

# Backing Up Configuration

A helper macro is provided to commit and push configuration changes:

```
BACKUP_CFG
```

This runs:

```
services/autocommit.sh
```

which commits and pushes the configuration to GitHub.

---

# External Components

This configuration integrates with several external projects:

- Klipper
- Moonraker
- Mainsail
- KlipperScreen
- Happy Hare ERCF

Those projects retain their respective licenses.

---

# License

This repository is released under the **MIT License**.

See the `LICENSE` file for details.

---

# Author

Jason S.
