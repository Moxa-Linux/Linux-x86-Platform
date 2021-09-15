# Ubuntu 20.04

- Kernel version: `5.4.0-70-generic`
- ISO: `ubuntu-20.04.2-live-server-amd64.iso`
- Download Link: https://releases.ubuntu.com/20.04/ubuntu-20.04.2-live-server-amd64.iso

## Prerequisite
- Setup network

- Update system packages to latest version
```bash=
apt-get update
```

- Install packages
```bash=
apt-get install build-essential linux-headers-`uname -r` -y
```

## Issue: Device or resource busy when load gpio-it87 driver
- Add `acpi_enforce_resources=lax` on boot parameter
```bash=
# Edit grub config
vi /etc/default/grub
GRUB_CMDLINE_LINUX="[...] acpi_enforce_resources=lax"

# If you change this file, run 'update-grub' afterwards to update
update-grub
```

## Enforce to use it87 series driver instead of iTCO driver (if needed)
```bash=
vi /lib/modprobe.d/iTCO-blacklist.conf

blacklist iTCO_wdt
blacklist iTCO_vendor_support
```

## Build necessary drivers (please refer to product page)
[Product Pages Link](/products/)

---

# LTE cellular module dial up on Ubuntu 20.04 (Telit LE910C4 modules)
[Telit/LE910C4](/cellular/telit/LE910C4.md)

---

# Wi-Fi dial up on Ubuntu 20.04
[Sparklan/WPEQ-261ACNI](/wifi/sparklan/WPEQ-261ACNI.md)
