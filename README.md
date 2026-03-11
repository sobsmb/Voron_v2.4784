
# Voron V2.4 Configuration (4784)

Configuration repository for my **Voron V2.4** running **Klipper**, with **Happy Hare ERCF MMU**.

This repository stores the printer configuration, macros, and supporting service configuration used on the machine.

---

# Printer Details

| Component | Description |
|-----------|-------------|
| Printer | Voron V2.4 |
| Firmware | Klipper |
| MMU | ERCF with Happy Hare |
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

mmu/                       Happy Hare ERCF configuration (not modified)
```

### Important Notes

* The **`mmu/` directory is left untouched** so that Happy Hare updates can be applied safely.
* Runtime files such as logs and Moonraker databases are excluded via `.gitignore`.
* The repository is automatically backed up to GitHub using the `BACKUP_CFG` macro.

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
