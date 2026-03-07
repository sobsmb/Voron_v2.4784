# Voron V2.4 Configuration (4784)

Configuration repository for my **Voron V2.4** running **Klipper**, with **Happy Hare ERCF MMU**.

This repository stores the printer configuration, macros, and supporting service configuration used on the machine.

---

## Printer Details

| Component | Description                     |
| --------- | ------------------------------- |
| Printer   | Voron V2.4                      |
| Firmware  | Klipper                         |
| MMU       | ERCF with Happy Hare            |
| Host      | Raspberry Pi running MainsailOS |
| UI        | Mainsail / KlipperScreen        |

---

## Repository Layout

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

## Backing Up Configuration

A helper macro is provided to commit and push configuration changes:

```
BACKUP_CFG
```

This runs the script:

```
services/autocommit.sh
```

which commits and pushes the configuration to GitHub.

---

## External Components

This configuration integrates with several external projects:

* Klipper
* Moonraker
* Mainsail
* KlipperScreen
* Happy Hare ERCF

Those projects retain their respective licenses.

---

## License

This repository is released under the **MIT License**.

See the `LICENSE` file for details.

---

## Author

Jason S.
