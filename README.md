
# Voron V2.4 – Blue Printer Configuration

This repository contains the full Klipper configuration for my **Voron V2.4** printer (nicknamed **Blue**).

This configuration includes several custom macro systems designed to make multi-print workflows much faster while still maintaining safety.

## Key Features

- Hot Standby Mode
- Fast Restart (skip homing and QGL)
- TAP sanity probe validation
- Automatic standby timeout shutdown
- Chamber air purge safety cycle
- MMU-friendly print ending
- Material-aware fan behavior

---

# Printer Hardware

| Component | Details |
|---|---|
| Printer | Voron V2.4 |
| Probe | Voron TAP |
| Toolhead | TAP compatible |
| MMU | ERCF |
| Chamber | Fully enclosed |
| Filtration | Nevermore |
| Airflow | Intake / Exhaust / Cooling fans |
| Firmware | Klipper |

---

# Major Macro Systems

## Hot Standby Mode

Hot standby allows the printer to remain in a **ready-to-print state** after a print finishes.

This is intended for workflows where multiple prints are started in sequence using the same material.

Hot standby is controlled with:

```
HEAT_HOLD_ON
HEAT_HOLD_OFF
```

When enabled, the printer will enter standby at the end of a print.

---

# Hot Standby Behavior

When **heat hold is enabled** and a print finishes, the printer will:

- Turn **off the hotend**
- Keep the **bed temperature unchanged**
- Leave **Nevermore / intake / exhaust fan state unchanged**
- Leave the **toolhead where MMU_EJECT places it**
- Keep **steppers enabled**
- Perform a **TAP probe at bed center**
- Store the probe Z value
- Mark standby as **ready**
- Start a **60 minute standby timer**

This preserves the printer in a stable state for the next print.

---

# Fast Restart System

If a new print begins while the printer is in hot standby, the startup sequence will attempt a **fast restart**.

Fast restart is allowed only when all conditions are true:

- Heat hold is enabled
- Standby is marked ready
- The material is the **same as the previous print**
- All axes are still **homed**
- The **sanity probe passes**

If all conditions are satisfied, the printer skips the heavy startup sequence.

---

# What Fast Restart Skips

Fast restart skips:

- Homing
- Quad Gantry Level (QGL)
- Nozzle cleaning
- Bed mesh reload
- Fan reconfiguration

The following still occur:

- Bed temperature set and wait
- Nozzle temperature set and wait
- Flow rate configuration
- Normal print start

---

# TAP Sanity Probe

Before allowing fast restart, the printer performs a **single TAP probe** at bed center.

The new probe result is compared to the stored standby probe value.

Tolerance:

```
±0.03 mm
```

If the difference exceeds this threshold, fast restart is **aborted** and a full startup sequence runs instead.

---

# Standby Safety Timer

Hot standby automatically starts a **60 minute safety timer**.

If no print begins before the timer expires, the printer performs a safe shutdown.

Shutdown sequence:

1. Turn off bed heater
2. Turn off hotend heater
3. Turn off Nevermore
4. Run chamber air purge
5. Clear standby state
6. Disable heat hold

---

# Chamber Air Purge

When the standby timeout expires, the printer performs a short purge cycle.

Fans activated:

- Intake fan
- Exhaust fan

Duration:

```
180 seconds
```

After the purge:

- Fans shut off
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

When entering hot standby, the printer performs a probe and stores:

```
HOT_STANDBY.last_probe_z
```

This value is used during fast restart sanity checks.

If standby ends due to timeout or cancel, this value is cleared.

---

# Example Workflow

Typical multi-print workflow:

1. Enable heat hold
2. Start a print
3. Print finishes and printer enters hot standby
4. Start next print
5. Fast restart triggers automatically

---

# Repository Structure

```
macros/
  print/
    print_start.cfg
    print_end.cfg
    cancel_print.cfg

  system/
    heat_hold.cfg
    hot_standby.cfg
```

---

# Credits

This configuration builds on the standard Voron Klipper ecosystem and extends it with a hot standby / fast restart system designed for high-throughput printing workflows.
