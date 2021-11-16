# DA-681A (CN)
- Support Model List:
  - DA-681A-I-DPP-WL1 (CN)
  - DA-681A-I-SP-WL1 (CN)

- Necessary drivers for controlling Super IO and UART (P1~P2):
  - [moxa-it87-serial-driver](https://github.com/Moxa-Linux/moxa-it87-serial-driver/tree/master)
  - [moxa-it87-gpio-driver](https://github.com/Moxa-Linux/moxa-it87-gpio-driver/tree/master)
  - moxa-it87-wdt-driver
    - For Ubuntu 20.04/Debian 10 (kernel ver >= 4.19): [moxa-it87-wdt-driver(5.2)](https://github.com/Moxa-Linux/moxa-it87-wdt-driver/tree/5.2/master)
    - For CentOS 7.9//Debian 9 (kernel ver < 4.19): [moxa-it87-wdt-driver(4.9.x)](https://github.com/Moxa-Linux/moxa-it87-wdt-driver/tree/4.9.x/master)

- Necessary drivers for controlling UART (P3~P12):
  - [mxu11x0](https://github.com/Moxa-Linux/mxu11x0/tree/buster-4.19.0-amd64/master)

After `make && make install` the above drivers,  
please run `depmod -a` command to generate modules.dep and map files in /lib/modules/version.

## Example code
- [mx-gpio-ctl](https://github.com/Moxa-Linux/Linux-x86-Platform/blob/main/tools/mx-gpio-ctl)

## UART (P1~P2, DB)
UART modes are controled by 3 gpio pins and it87_serial:

- Super IO (IT8786) PIN table

| DEVICE NODE | GPIO#0 | GPIO#1 | GPIO#2 | it87_serial |
| ----------  | ------ | ------ | ------ | ----------- |
| /dev/ttyS0  |  GP13  |  GP11  |  GP12  |      1      |
| /dev/ttyS1  |  GP16  |  GP14  |  GP15  |      2      |

- GPIO value of UART modes

|   UART MODE    | GPIO#0 | GPIO#1 | GPIO#2 | it87_serial |
| -------------- | ------ | ------ | ------ | ----------- |
| RS232          |   1    |    0   |    0   |      0      |
| RS485-2W       |   0    |    1   |    0   |      1      |
| RS422/RS485-4W |   0    |    0   |    1   |      1      |

- Example code
	1. select port 1 (/dev/ttyS0) as RS232/RS485-2W/RS422/RS485-4W mode
	```bash=
	# set port 1 as RS232 mode
	mx-gpio-ctl set 13 1
	mx-gpio-ctl set 11 0
	mx-gpio-ctl set 12 0
	echo 0 > /sys/class/misc/it87_serial/serial1/serial1_rs485

	# set port 1 as RS485-2W mode
	mx-gpio-ctl set 13 0
	mx-gpio-ctl set 11 1
	mx-gpio-ctl set 12 0
	echo 1 > /sys/class/misc/it87_serial/serial1/serial1_rs485

	# set port 1 as RS422/RS485-4W mode
	mx-gpio-ctl set 13 0
	mx-gpio-ctl set 11 0
	mx-gpio-ctl set 12 1
	echo 1 > /sys/class/misc/it87_serial/serial1/serial1_rs485
	```
	2. select port 2 (/dev/ttyS1) as RS232/RS485-2W/RS422/RS485-4W mode
	```bash=
	# set port 1 as RS232 mode
	mx-gpio-ctl set 16 1
	mx-gpio-ctl set 14 0
	mx-gpio-ctl set 15 0
	echo 0 > /sys/class/misc/it87_serial/serial2/serial2_rs485

	# set port 2 as RS485-2W mode
	mx-gpio-ctl set 16 0
	mx-gpio-ctl set 14 1
	mx-gpio-ctl set 15 0
	echo 1 > /sys/class/misc/it87_serial/serial2/serial2_rs485

	# set port 2 as RS422/RS485-4W mode
	mx-gpio-ctl set 16 0
	mx-gpio-ctl set 14 0
	mx-gpio-ctl set 15 1
	echo 1 > /sys/class/misc/it87_serial/serial2/serial2_rs485
	```
	3. get port 2 (/dev/ttyS1) mode
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
	# Then look up the above table, port 2 is on RS485-2W mode.
	```

## UART (P3~P12, TB)
UART modes are controled by ioctl (default is RS485-2W):

- Driver: [mxu11x0](https://github.com/Moxa-Linux/mxu11x0/tree/buster-4.19.0-amd64/master)
- Utility: [setserial](http://sourceforge.net/projects/setserial/)
```
# install package on ubuntu 20.04
apt-get update && apt-get install setserial -y

# install package on centOS 7.9
yum install setserial
```
- Due to TUSB3410 chips are partially connected to two USB hubs, the `ttyUSB` ports number are not increased correctly from driver.  
Add udev rules to create `ttyM` symlinks: edit `/lib/udev/rules.d/10-serial.rules` 
 
```
KERNEL=="ttyS[0-1]*", KERNELS=="ttyS0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM0"
KERNEL=="ttyS[0-1]*", KERNELS=="ttyS1", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM1"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-7:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM2"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-8:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM3"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-9.1:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM4"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-9.2:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM5"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-9.3:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM6"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-9.4:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM7"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-10.1:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM8"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-10.2:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM9"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-10.3:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM10"
KERNEL=="ttyUSB[0-9]*", KERNELS=="1-10.4:1.0", SUBSYSTEM=="tty", ACTION=="add", SYMLINK+="ttyM11"
```

After reboot, the `/dev/ttyM*` device nodes are created:
```
# ls /dev/ttyM* -al
lrwxrwxrwx 1 root root 5 May 21 03:00 /dev/ttyM0 -> ttyS0
lrwxrwxrwx 1 root root 5 May 21 03:00 /dev/ttyM1 -> ttyS1
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM10 -> ttyUSB8
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM11 -> ttyUSB9
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM2 -> ttyUSB0
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM3 -> ttyUSB1
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM4 -> ttyUSB2
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM5 -> ttyUSB3
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM6 -> ttyUSB5
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM7 -> ttyUSB7
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM8 -> ttyUSB4
lrwxrwxrwx 1 root root 7 May 21 03:00 /dev/ttyM9 -> ttyUSB6
```

- UART ports mapping (after adding udev rules):

| Port Num | Symlink Device Node |
| -------- | ------------------  |
|    P1    |    /dev/ttyM0       | 
|    P2    |    /dev/ttyM1       | 
|    P3    |    /dev/ttyM2       | 
|    P4    |    /dev/ttyM3       | 
|    P5    |    /dev/ttyM4       | 
|    P6    |    /dev/ttyM5       | 
|    P7    |    /dev/ttyM6       | 
|    P8    |    /dev/ttyM7       | 
|    P9    |    /dev/ttyM8       | 
|    P10   |    /dev/ttyM9       | 
|    P11   |    /dev/ttyM10      | 
|    P12   |    /dev/ttyM11      | 

- Get P3~P12 (/dev/ttyM2 ~ /dev/ttyM11) UART mode (default is RS-485 2W mode)
```
### Get UART mode
### port 0x0000: RS-232
### port 0x0001: RS-485 2W
### port 0x0002: RS-422
### port 0x0003: RS-485 4W
$ setserial -G /dev/ttyM2
/dev/ttyM2 uart 16550A port 0x0001 irq 0 baud_base 9600 spd_normal low_latency
# [...]
$ setserial -G /dev/ttyM11
/dev/ttyM11 uart 16550A port 0x0001 irq 0 baud_base 9600 spd_normal low_latency
```

- Set P3~P12 (/dev/ttyM2 ~ /dev/ttyM11) UART mode (default is RS-485 2W mode)
```
### Set UART mode
### $ setserial /dev/ttyMx port [mode]
### mode:
### 0: RS-232
### 1: RS-485 2W
### 2: RS-422
### 3: RS-485 4W

# Set P3 as RS-232 mode
$ setserial /dev/ttyM2 port 0
$ setserial -G /dev/ttyM2
/dev/ttyM2 uart 16550A port 0x0000 irq 0 baud_base 9600 spd_normal low_latency

# Set P3 as RS-485 2W mode
$ setserial /dev/ttyM2 port 1
$ setserial -G /dev/ttyM2
/dev/ttyM2 uart 16550A port 0x0001 irq 0 baud_base 9600 spd_normal low_latency

# Set P3 as RS-422 mode
$ setserial /dev/ttyM2 port 2
$ setserial -G /dev/ttyM2
/dev/ttyM2 uart 16550A port 0x0002 irq 0 baud_base 9600 spd_normal low_latency

# Set P3 as RS-485 4W mode
$ setserial /dev/ttyM2 port 3
$ setserial -G /dev/ttyM2
/dev/ttyM2 uart 16550A port 0x0003 irq 0 baud_base 9600 spd_normal low_latency

# [...]
$ setserial /dev/ttyM11 port 3
$ setserial -G /dev/ttyM11
/dev/ttyM11 uart 16550A port 0x0003 irq 0 baud_base 9600 spd_normal low_latency
```

## Relay control
- Set GPIO Low = Close
- Set GPIO High = Open

(Notice: System-On GPIO High=Open, System-Off GPIO Low=Close)

|    RELAY CTL   | GPIO PIN |
| -------------- | -------- |
|  Relay Slot #1 |   GP65   |

- Example code
	1. Set 'Open' status on relay slot #1: `mx-gpio-ctl set 65 1`
	2. Set 'Close' status on relay slot #1: `mx-gpio-ctl set 65 0`
	3. Get relay status on slot #1: `mx-gpio-ctl get 65`

## Programmable LED
- Set GPIO Low = LED light ON
- Set GPIO High = LED light OFF

(Notice: the Programmable LED is `active low`, thus set `high` means `light OFF`, set `low` means `light ON`)

| MPCIE PWR CTL | GPIO PIN |
| ------------  | -------- |
| LED index #1  |   GP70   |
| LED index #2  |   GP71   |
| LED index #3  |   GP72   |
| LED index #4  |   GP73   |
| LED index #5  |   GP74   |
| LED index #6  |   GP75   |

- Example code
	1. Light ON LED index #1: `mx-gpio-ctl set 70 0`
	2. Light OFF LED index #1: `mx-gpio-ctl set 70 1`
	3. Get status of LED index #1: `mx-gpio-ctl get 70`

## Watchdog  

- moxa-it87-wdt-driver
  - For Ubuntu 20.04 (kernel ver >= 4.19) [moxa-it87-wdt-driver](https://github.com/Moxa-Linux/moxa-it87-wdt-driver/tree/buster/master)
  - For CentOS 7.9 (kernel ver < 4.19) [moxa-it87-wdt-driver](https://github.com/Moxa-Linux/moxa-it87-wdt-driver/tree/stretch/master)

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

### Watchdog example code  
https://github.com/torvalds/linux/tree/master/tools/testing/selftests/watchdog

---

## LAN interface (Notice: NIC naming rules are depended on the setting of Linux distribution)
| LAN SLOT |  NIC |  renamed NIC  |
| -------- | ---- | ------------- |
| LAN #1   | eth0 |   enp2s0      |
| LAN #2   | eth1 |   enp3s0      |
| LAN #3   | eth2 |   enp4s0      |
| LAN #4   | eth3 |   enp5s0      |
| LAN #5   | eth4 |   enp6s0      |
| LAN #6   | eth5 |   enp7s0      |

---

