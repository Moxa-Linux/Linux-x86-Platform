# MC-3201-TGL

- Support Model List:
  - MC-3201-TL1-S-S
  - MC-3201-TL7-M-S

- Support Linux Dist:
  - [Debian-11 (bullseye)](/dist/Debian-11.md)
  - [Ubuntu-20.04-HWE](/dist/Ubuntu-20.04-HWE.md)

- Necessary drivers for controlling Super IO:
  - [moxa-it87-serial-driver](https://github.com/Moxa-Linux/moxa-it87-serial-driver/tree/master)
  - [moxa-it87-gpio-driver](https://github.com/Moxa-Linux/moxa-it87-gpio-driver/tree/master)
  - moxa-it87-wdt-driver
    - For Ubuntu 20.04/Debian 11 (kernel ver >= 4.19): [moxa-it87-wdt-driver(5.2)](https://github.com/Moxa-Linux/moxa-it87-wdt-driver/tree/5.2/master)

After `make && make install` the upon drivers,  
please run `depmod -a` command to generate modules.dep and map files in /lib/modules/version.

---

## Example code
- [mx-gpio-ctl](https://github.com/Moxa-Linux/Linux-x86-Platform/blob/main/tools/mx-gpio-ctl)
- [mx-cp2112-gpio-ctl](https://github.com/Moxa-Linux/Linux-x86-Platform/blob/main/tools/mx-cp2112-gpio-ctl)

---

## UART
UART modes are controled by 3 gpio pins and it87_serial:

- Super IO (IT8786) PIN table

| Device Node | GPIO#0 | GPIO#1 | GPIO#2 | it87_serial |
| ----------  | ------ | ------ | ------ | ----------- |
| /dev/ttyS0  |  GP13  |  GP11  |  GP12  |      1      |
| /dev/ttyS1  |  GP16  |  GP14  |  GP15  |      2      |

- GPIO value of UART modes

|   UART mode    | GPIO#0 | GPIO#1 | GPIO#2 | it87_serial |
| -------------- | ------ | ------ | ------ | ----------- |
| RS232          |   1    |    0   |    0   |      0      |
| RS485-2W       |   0    |    1   |    0   |      1      |
| RS422/RS485-4W |   0    |    0   |    1   |      1      |

- Example code for port 1
  1. select port 1 (/dev/ttyS0) as RS232 mode
  ```bash=
  mx-gpio-ctl set 13 1
  mx-gpio-ctl set 11 0
  mx-gpio-ctl set 12 0
  echo 0 > /sys/class/misc/it87_serial/serial1/serial1_rs485
  ```
  2. select port 1 (/dev/ttyS0) as RS485-2W mode
  ```bash=
  mx-gpio-ctl set 13 0
  mx-gpio-ctl set 11 1
  mx-gpio-ctl set 12 0
  echo 1 > /sys/class/misc/it87_serial/serial1/serial1_rs485
  ```
  3. select port 1 (/dev/ttyS0) as RS422/RS485-4W mode
  ```bash=
  mx-gpio-ctl set 13 0
  mx-gpio-ctl set 11 0
  mx-gpio-ctl set 12 1
  echo 1 > /sys/class/misc/it87_serial/serial1/serial1_rs485
  ```
  4. get port 1 (/dev/ttyS0) mode
  ```bash=
  mx-gpio-ctl get 13
  mx-gpio-ctl get 11
  mx-gpio-ctl get 12
  cat /sys/class/misc/it87_serial/serial1/serial1_rs485
  ```
  Result:
  ```text
  0
  1
  0
  1
  ```
  
  Then look up the table, port 1 is on RS485-2W mode.

- Example code for port 2
	1. select port 2 (/dev/ttyS1) as RS232 mode
	```bash=
	mx-gpio-ctl set 16 1
	mx-gpio-ctl set 14 0
	mx-gpio-ctl set 15 0
	echo 0 > /sys/class/misc/it87_serial/serial2/serial2_rs485
	```
	2. select port 2 (/dev/ttyS1) as RS485-2W mode
	```bash=
	mx-gpio-ctl set 16 0
	mx-gpio-ctl set 14 1
	mx-gpio-ctl set 15 0
	echo 1 > /sys/class/misc/it87_serial/serial2/serial2_rs485
	```
	3. select port 2 (/dev/ttyS1) as RS422/RS485-4W mode
	```bash=
	mx-gpio-ctl set 16 0
	mx-gpio-ctl set 14 0
	mx-gpio-ctl set 15 1
	echo 1 > /sys/class/misc/it87_serial/serial2/serial2_rs485
	```
	4. get port 2 (/dev/ttyS1) mode
	```bash=
	mx-gpio-ctl get 16
	mx-gpio-ctl get 14
	mx-gpio-ctl get 15
	cat /sys/class/misc/it87_serial/serial2/serial2_rs485
	```
	Result:
	```text
	0
	1
	0
	1
	```

	Then look up the table, port 2 is on RS485-2W mode.

---

## NMEA
- FT4232 UART chip

| Port |  Device Node  |
| ---- | ------------  |
|  1   | /dev/ttyUSB0  |
|  2   | /dev/ttyUSB1  |
|  3   | /dev/ttyUSB2  |
|  4   | /dev/ttyUSB3  |

---

## DIO
- DIO status will be kept when warn (soft) reboot
- DO = HIGH (Relay, Close)  -->  DI = LOW
- DO = LOW (Relay, Open) -->  DI = HIGH
- CP2112 PIN table

| DIO index   | GPIO PIN |
| ----------  | -------- |
| DI 0        |    0     |
| DI 1        |    1     |
| DI 2        |    2     |
| DI 3        |    3     |
| DO 0        |    4     |
| DO 1        |    5     |
| DO 2        |    6     |
| DO 3        |    7     |

- Example code
  1. get DI 0 state: `mx-cp2112-gpio-ctl get 0`
  2. get DO 0 state: `mx-cp2112-gpio-ctl get 4`
  3. set DO 0 state as low: `mx-cp2112-gpio-ctl set 0 4`

---

## Cellular: mPCIe slot power control
- Default status is according to BIOS setting

### 1st-cut setting
- Set GP41 & GP46 Low = Power off
- Set GP41 & GP46 High = Power on

| MPCIE PWR CTL |     GPIO PIN    |
| ------------  | --------------- |
| mPCIe Slot    |   GP41 & GP46   |

- For power on mPCIe slot
  - Set High on GP41, wait 100ms, and Set High on GP46

- For power off mPCIe slot
  - Set Low on GP41, wait 100ms, and Set Low on GP46

- For reset module
  - Set High, wait 100ms, and Low on GP46

- Example code
  1. Power on slot: `mx-gpio-ctl set 41 1; sleep 0.1; mx-gpio-ctl set 46 1`
  2. Power off slot: `mx-gpio-ctl set 41 0; sleep 0.1; mx-gpio-ctl set 46 0`
  3. Get power status on slot: `mx-gpio-ctl get 41`
  4. Get reset pin status on slot: `mx-gpio-ctl get 46`

---

## Cellular: M.2 Key B slot

---

## Cellular: SIM card select

| SIM SEL          | GPIO PIN | SLOT Type |
| ---------------  | -------- | --------- |
| SIM Slot #1/#2   |   GP80   | mPCIe     |
| SIM Slot #3/#4   |   GP82   | M.2 Key B |

- Default status
  - mPCIe: SIM slot #1 (Low)
  - M.2 Key B: SIM slot #3 (Low)

- Select SIM slot
  - mPCIe:
    - Set GP80 Low = Select SIM slot #1
    - Set GP80 High = Select SIM slot #2
  - M.2 Key B:
    - Set GP82 Low = Select SIM slot #3
    - Set GP82 High = Select SIM slot #4

- Example code
  1. Select SIM slot #1: `mx-gpio-ctl set 80 0`
  2. Select SIM slot #2: `mx-gpio-ctl set 80 1`
  3. Select SIM slot #3: `mx-gpio-ctl set 82 0`
  4. Select SIM slot #4: `mx-gpio-ctl set 82 1`
  5. Get SIM slot #1/#2 status: `mx-gpio-ctl get 80`
  6. Get SIM slot #3/#4 status: `mx-gpio-ctl get 82`

---

## Cellular: LTE Telit module dial-up and connection guide
[Telit/LE910C4](/cellular/telit/LE910C4.md)

---

## LAN interface

| LAN Slot |  NIC |  renamed NIC  |
| -------- | ---- | ------------- |
| LAN #1   | eth0 |   enp0s31f6   |
| LAN #2   | eth1 |   enp6s0      |
| LAN #3   | eth2 |   enp7s0      |
| LAN #4   | eth3 |   enp8s0      |

- For Ubuntu 20.04 LAN setting:
I219 LAN chip: only support on **Ubuntu 20.04 HWE** kernel version:  
```
apt update
apt install --install-recommends linux-generic-hwe-20.04

# after reboot, check kernel version and NIC list
uname -a
# Linux moxa 5.11.0-34-generic #36~20.04.1-Ubuntu SMP Fri Aug 27 08:06:32 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux

ip a | grep enp
# 2: enp6s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
# 3: enp7s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
# 4: enp8s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
#    inet 10.123.8.84/23 brd 10.123.9.255 scope global enp8s0
# 5: enp0s31f6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
```
- Ref
https://askubuntu.com/questions/1344156/ubuntu-20-04-2-and-onboard-intel-i219-v

---

## Watchdog  
- Build [moxa-it87-wdt-driver](https://github.com/Moxa-Linux/moxa-it87-wdt-driver) on your host  

### Example for probing `it87_wdt` driver on boot  
- Edit `/lib/modprobe.d/watchdog.conf`
```
# timeout:Watchdog timeout in seconds, default=60 (int)
# nowayout:Watchdog cannot be stopped once started, default=0 (bool)
options it87_wdt nowayout=1 timeout=60
```

### Disable `iTCO_wdt` driver  
- Edit `/lib/modprobe.d/iTCO-blacklist.conf`
```
blacklist iTCO_wdt
blacklist iTCO_vendor_support
```

### Disalbe `NMI watchdog` driver (for Ubuntu)
- First edit the /etc/default/grub file as root and then add nmi_watchdog=0 to the line starting with GRUB_CMDLINE_LINUX_DEFAULT
```
GRUB_CMDLINE_LINUX_DEFAULT="nmi_watchdog=0"
```
- After adding the kernel parameter, don't forget to update the GRUB
```
sudo update-grub
```

### Watchdog daemon package (Debian/Ubuntu)  
- Install package  
```bash
sudo apt-get update
sudo apt-get install watchdog
```
- Edit `/etc/default/watchdog`  
```text
# Start watchdog at boot time? 0 or 1
run_watchdog=1
# Start wd_keepalive after stopping watchdog? 0 or 1
run_wd_keepalive=0
# Load module before starting watchdog
watchdog_module="it87_wdt"
# Specify additional watchdog options here (see manpage).
```
- Edit `/etc/watchdog.conf`
To uncomment this to use the watchdog device driver access "file"
```text
watchdog-device                = /dev/watchdog
```

### Watchdog example code  
https://github.com/torvalds/linux/tree/master/tools/testing/selftests/watchdog
