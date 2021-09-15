# DA-682C-KL
- Support Model List:
  - DA-682C-KL1-H-T
  - DA-682C-KL1-HH-T
  - DA-682C-KL3-H-T
  - DA-682C-KL3-HH-T
  - DA-682C-KL5-H-T
  - DA-682C-KL5-HH-T
  - DA-682C-KL7-H-T
  - DA-682C-KL7-HH-T

- Suppoer Linux Dist:
  - Debian 9

- Necessary drivers for controlling Super IO:
  - [moxa-it87-serial-driver](https://github.com/Moxa-Linux/moxa-it87-serial-driver/tree/master)
  - [moxa-it87-gpio-driver](https://github.com/Moxa-Linux/moxa-it87-gpio-driver/tree/master)
  - moxa-it87-wdt-driver
    - For Debian 9 (kernel ver < 4.19): [moxa-it87-wdt-driver(4.9.x)](https://github.com/Moxa-Linux/moxa-it87-wdt-driver/tree/4.9.x/master)

After `make && make install` the above drivers,  
please run `depmod -a` command to generate modules.dep and map files in /lib/modules/version.

## Example code
- [mx-gpio-ctl](https://github.com/Moxa-Linux/Linux-x86-Platform/blob/main/tools/mx-gpio-ctl)

## UART
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
	1. select port 1 (/dev/ttyS0) as RS232 mode
	```bash=
	mx-gpio-ctl set 13 1
	mx-gpio-ctl set 11 0
	mx-gpio-ctl set 12 0
	echo 0 > /sys/class/misc/it87_serial/serial1/serial1_rs485
	```
	2. select port 2 (/dev/ttyS1) as RS485-2W mode
	```bash=
	mx-gpio-ctl set 16 0
	mx-gpio-ctl set 14 1
	mx-gpio-ctl set 15 0
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
	```

	Then look up the table, port 2 is on RS485-2W mode.

## DIO
- Super IO (IT8786) PIN table

(Notice: Default status depends on BIOS setting)

| DIO INDEX   | GPIO PIN |
| ----------  | -------- |
| DI 0        |   GP82   |
| DI 1        |   GP83   |
| DI 2        |   GP84   |
| DI 3        |   GP85   |
| DI 4        |   GP36   |
| DI 5        |   GP37   |
| DO 0        |   GP33   |
| DO 1        |   GP34   |

- Example code
	1. set DO 0 value as low: `mx-gpio-ctl set 33 0`
	2. get DI 0 value: `mx-gpio-ctl get 82`

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

(Notice: the Programmable LED is active low, thus set high means light OFF, set low means light ON)

| MPCIE PWR CTL | GPIO PIN |
| ------------  | -------- |
| LED index #1  |   GP70   |
| LED index #2  |   GP71   |
| LED index #3  |   GP72   |
| LED index #4  |   GP73   |
| LED index #5  |   GP74   |
| LED index #6  |   GP75   |
| LED index #7  |   GP76   |
| LED index #8  |   GP77   |

- Example code
	1. Light ON LED index #1: `mx-gpio-ctl set 70 0`
	2. Light OFF LED index #1: `mx-gpio-ctl set 70 1`
	3. Get status of LED index #1: `mx-gpio-ctl get 70`

## USB power controll
- Set GPIO Low = Power OFF USB port
- Set GPIO High = Power ON USB port

(Notice: Default status is Power ON, depends on BIOS setting)

| USB PORT LOCATION | GPIO PIN |
| ----------------  | -------- |
|    USB2 Wafer     |   GP41   |
|    USB2 Front     |   GP46   |
|    USB3 REAR      |   GP27   |

- Example code
	1. Power ON USB2 Wafer: `mx-gpio-ctl set 41 1`
	2. Power OFF USB3 REAR: `mx-gpio-ctl set 27 0`
	3. Get status of USB2 Front: `mx-gpio-ctl get 46`

## 12V IN Power Supply Monitor
- Get GPIO Low = 12V IN Power Supply status is ON
- Get GPIO High = 12V IN Power Supply status is OFF

|    12V IN Power SLOT  | GPIO PIN |
| --------------------- | -------- |
|  Power SLOT #1 (PWR1) |   GP80   |
|  Power SLOT #2 (PWR2) |   GP81   |

- Example code
	1. Get Power Supply SLOT #1 status: `mx-gpio-ctl get 80`
	2. Get Power Supply SLOT #2 status: `mx-gpio-ctl get 81`

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
| LAN #1   | eth0 |   enp0s31f6   |
| LAN #2   | eth1 |   enp3s0      |
| LAN #3   | eth2 |   enp4s0      |
| LAN #4   | eth3 |   enp5s0      |
| LAN #5   | eth4 |   enp6s0      |
| LAN #6   | eth5 |   enp9s0      |

---

## Expansion Module - DN-SP08-I-TB/DN-SP08-I-DB
- Necessary drivers for controlling `DN-SP08-I-TB/DN-SP08-I-DB` module
  - [moxa-hid-ft260-driver](https://github.com/Moxa-Linux/moxa-hid-ft260-driver)
  - [moxa-it87-gpio-driver](https://github.com/Moxa-Linux/moxa-gpio-pca953x-driver)

After `make && make install` the above drivers, <br>
please run `depmod -a` command to generate modules.dep and map files in '/lib/modules/version'.

- Edit udev rule for re-bind FT260 device
e.g. `/lib/udev/rules.d/11-ft260-pca9535.rules` (depending on your Linux setting)
```
ACTION=="add", KERNEL=="0003:0403:6030.*", SUBSYSTEM=="hid", DRIVERS=="hid-generic", \
RUN+="/bin/bash -c 'echo $kernel > /sys/bus/hid/drivers/hid-generic/unbind'", \
RUN+="/bin/bash -c 'echo $kernel > /sys/bus/hid/drivers/ft260/bind'"
```

- Bind `FT260` USB-to-I2C driver for `PCA9535`
```bash=
# search ft260 device, e.g. assume i2c bus index 6
cat /sys/bus/i2c/devices/i2c-6/name | grep "FT260"

# get result:
# FT260 usb-i2c bridge on hidraw2

# then bind pca9535 device on FT260, using addresses '0x26' and '0x27'
echo "pca9535 0x26" > /sys/bus/i2c/devices/i2c-6/new_device
echo "pca9535 0x27" > /sys/bus/i2c/devices/i2c-6/new_device

# then check pca9535 is registered on gpio chip sysfs
# notice: gpiochip index 432/416 is dynamic generated from sysfs
cat /sys/class/gpio/gpiochip432/label | grep pca9535
cat /sys/class/gpio/gpiochip416/label | grep pca9535

# get result:
# pca9535
# pca9535

# finally, user can select UART mode via `/sys/class/gpio/gpio[n]/direction`
```

## One module card plugged
- Select UART mode

**Assume gpiochip base is `416` and `432`**: 
<br>
**(Notice: gpiochip base number are dynamic generated, please check base number from the above description)**  

| Port Num | Device Node   | GPIO#0 | GPIO#1 | GPIO#2 | GPIO#3 |
| -------- | ------------  | ------ | ------ | ------ | ------ |
|    P1    | /dev/ttyUSB0  |   432  |  433   |   434  |   435  |
|    P2    | /dev/ttyUSB1  |   436  |  437   |   438  |   439  |
|    P3    | /dev/ttyUSB2  |   440  |  441   |   442  |   443  |
|    P4    | /dev/ttyUSB3  |   444  |  445   |   446  |   447  |
|    P5    | /dev/ttyUSB4  |   416  |  417   |   418  |   419  |
|    P6    | /dev/ttyUSB5  |   420  |  421   |   422  |   423  |
|    P7    | /dev/ttyUSB6  |   424  |  425   |   426  |   427  |
|    P8    | /dev/ttyUSB7  |   428  |  429   |   430  |   431  |

**GPIO value of UART modes:**  

|   UART mode    | GPIO#0 | GPIO#1 | GPIO#2 | GPIO#3 |
| -------------- | ------ | ------ | ------ | ------ |
| RS232          |   1    |    1   |    0   |    0   |
| RS485-2W       |   0    |    0   |    0   |    1   |
| RS422/RS485-4W |   0    |    0   |    1   |    0   |

- Example code

First step, user needs to export all `pca9535` gpio path:  
```bash=
#!/bin/bash
#
# An example code to export pca9535 gpio
#

TARGET_GPIOCHIP="pca9535"
GPIOCHIP_NAME=gpiochip
GPIO_FS_PATH=/sys/class/gpio

# Export GPIOs
ls $GPIO_FS_PATH | grep $GPIOCHIP_NAME | while read -r chip ; do
	GPIO_LABEL=$(cat $GPIO_FS_PATH/$chip/label)
	if [[ "$GPIO_LABEL" != *"$TARGET_GPIOCHIP"* ]]; then
		continue
	fi

	pinstart=$(echo $chip | sed s/$GPIOCHIP_NAME/\\n/g)
	count=$(cat $GPIO_FS_PATH/$chip/ngpio)
	for (( i=0; i<${count}; i++ )); do
        echo $((${pinstart}+${i})) > "/sys/class/gpio/export" 
	done
done
```

Then, to control UART modes via sysfs gpio direction:  
```bash=
# Example code
# to switch expansion module card P1 (/dev/ttyUSB0) to RS232 mode
echo "high" > /sys/class/gpio/gpio432/direction
echo "high" > /sys/class/gpio/gpio433/direction
echo "low" > /sys/class/gpio/gpio434/direction
echo "low" > /sys/class/gpio/gpio435/direction

# to switch expansion module card P2 (/dev/ttyUSB1) to RS485-2W mode
echo "low" > /sys/class/gpio/gpio436/direction
echo "low" > /sys/class/gpio/gpio437/direction
echo "low" > /sys/class/gpio/gpio438/direction
echo "high" > /sys/class/gpio/gpio439/direction

# to switch expansion module card P8 (/dev/ttyUSB7) to RS422/RS485-4W mode
echo "low" > /sys/class/gpio/gpio428/value
echo "low" > /sys/class/gpio/gpio429/value
echo "high" > /sys/class/gpio/gpio430/value
echo "low" > /sys/class/gpio/gpio431/value
```
