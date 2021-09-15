# CentOS 7.9

- Kernel version: `3.10.0-1160.21.1.el7.x86_64`
- Download Link: http://isoredirect.centos.org/centos/7/isos/x86_64/

## Prerequisite

- Update system packages to latest version
```bash=
yum distro-sync
```

- Install packages
```bash=
# install kernel devel and header packages
yum install "kernel-devel-$(uname -r)"
yum install "kernel-headers-$(uname -r)"
yum install gcc
```

- After installing the kernel packages, you can find all the kernel headers files in /usr/src/kernels directory using following command. 
```bash=
ls /lib/modules/$(uname -r)
ls /usr/src/kernels/$(uname -r)
```

## Issue: Device or resource busy when load gpio-it87 driver
- Add `acpi_enforce_resources=lax` on boot parameter
```bash=
# Edit grub config
vi /etc/default/grub
GRUB_CMDLINE_LINUX="[...] acpi_enforce_resources=lax"

# If you change this file, run 'grub2-mkconfig' afterwards to update
grub2-mkconfig -o /etc/grub2.cfg
grub2-mkconfig -o /etc/grub2-efi.cfg
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

# LTE cellular module dial up on CentOS 7.9 (Telit LE910C4 modules)
[Telit/LE910C4](/cellular/telit/LE910C4.md)

---

# Wi-Fi dial up on CentOS 7.9
[Sparklan/WPEQ-261ACNI](/wifi/sparklan/WPEQ-261ACNI.md)

