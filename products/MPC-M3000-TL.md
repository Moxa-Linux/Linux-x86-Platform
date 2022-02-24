# MPC-M3000-TL

- Support Model List:
  - MPC-M3190S-TG1-DC
  - MPC-M3190S-TG3-DC
  - MPC-M3190S-TG7-DC
  - MPC-M3190S-TG1-AC
  - MPC-M3190S-TG3-AC
  - MPC-M3190S-TG7-AC
  - MPC-M3240W-TG1-DC
  - MPC-M3240W-TG3-DC
  - MPC-M3240W-TG7-DC
  - MPC-M3240W-TG1-AC
  - MPC-M3240W-TG3-AC
  - MPC-M3240W-TG7-AC

- Support Linux Dist:
  - [Debian-11 (bullseye)](/dist/Debian-11.md)

- Necessary drivers for controlling Super IO:
  - [moxa-it87-gpio-driver](https://github.com/Moxa-Linux/moxa-it87-gpio-driver/tree/master)
  - [moxa-it87-wdt-driver](https://github.com/Moxa-Linux/moxa-it87-wdt-driver/tree/5.2/master)

After `make && make install` the upon drivers,  
please run `depmod -a` command to generate modules.dep and map files in /lib/modules/version.


---

## Example code
- [mx-gpio-ctl](https://github.com/Moxa-Linux/Linux-x86-Platform/blob/main/tools/mx-gpio-ctl)

---

## UART
UART modes are controled by 4 gpio pins :

- Build [moxa-gpio-pca953x-driver](https://github.com/Moxa-Linux/moxa-gpio-pca953x-driver)

- PCA9535 PIN table

| Device Node | GPIO#0 | GPIO#1 | GPIO#2 | GPIO#3 |
| ----------  | ------ | ------ | ------ | -------|
| /dev/ttyUSB0  |    0   |    1   |    2   |    3   |
| /dev/ttyUSB1  |    4   |    5   |    6   |    7   |
| /dev/ttyUSB2  |    8   |    9   |    10  |    11  |
| /dev/ttyUSB3  |    12  |    13  |    14  |    15  |
- GPIO value of UART modes

|   UART mode    | GPIO#0 | GPIO#1 | GPIO#2 | GPIO#3 |
| -------------- | ------ | ------ | ------ | ------ |
| RS232          |   1    |    1   |    0   |    0   |
| RS485-2W       |   0    |    0   |    0   |    1   |
| RS422/RS485-4W |   0    |    0   |    1   |    0   |

- Example code for port 1
  1. Bind pca9535 device on cp2112
  ```bash=
  # Search for cp2112 device
  cat /sys/bus/i2c/devices/i2c-15/name
  # Probe & bind pca9535 to cp2112
  modprobe gpio-pca953x
  echo "pca9535 0x26" > /sys/bus/i2c/devices/i2c-15/new_device
  # Check the result
  cat /sys/class/gpio/gpiochip128/label
  ```
  ```bash=
  # Script for exporting all pca9535 pins
  TARGET_GPIOCHIP=pca9535 
  GPIOCHIP_NAME=gpiochip 
  GPIO_FS_PATH=/sys/class/gpio 
  ls $GPIO_FS_PATH | grep $GPIOCHIP_NAME | while read -r chip ; do
	  if [ x"$TARGET_GPIOCHIP"!= x"" ]; then
		  if [ x"$TARGET_GPIOCHIP"!= x" $(cat $GPIO_FS_PATH/$chip/label)" ]
		  	continue
		  fi
	  fi
	  pinstart=$(echo $chip | sed s/$GPIOCHIP_NAME/\\n/g) 
	  count=$(cat$GPIO_FS_PATH/$chip/ngpio)
	  for (( i=0; i<${count}; i++ )); do
		  echo $((${pinstart}+${i})) > $GPIO_FS_PATH/export
	  done
  done
  ```
  2. select port 1 (/dev/ttyS0) as RS232 mode
  ```bash=
  echo "high" > /sys/class/gpio/gpio128/direction
  echo "high" > /sys/class/gpio/gpio129/direction
  echo "low" > /sys/class/gpio/gpio130/direction
  echo "low" > /sys/class/gpio/gpio131/direction
  ```
  3. select port 1 (/dev/ttyS0) as RS485-2W mode
  ```bash=
  echo "low" > /sys/class/gpio/gpio128/direction
  echo "low" > /sys/class/gpio/gpio129/direction
  echo "low" > /sys/class/gpio/gpio130/direction
  echo "high" > /sys/class/gpio/gpio131/direction
  ```
  4. select port 1 (/dev/ttyS0) as RS422/RS485-4W mode
  ```bash=
  echo "low" > /sys/class/gpio/gpio128/direction
  echo "low" > /sys/class/gpio/gpio129/direction
  echo "high" > /sys/class/gpio/gpio130/direction
  echo "low" > /sys/class/gpio/gpio131/direction
  ```

---

## NMEA
- FT4232 UART chip

| Port |  Device Node  |
| ---- | ------------  |
|  1   | /dev/ttyUSB4  |
|  2   | /dev/ttyUSB5  |
|  3   | /dev/ttyUSB6  |
|  4   | /dev/ttyUSB7  |
|  1   | /dev/ttyUSB8  |
|  2   | /dev/ttyUSB9  |
|  3   | /dev/ttyUSB10  |
|  4   | /dev/ttyUSB11  |
---

## DIO
- DIO status will be kept when warn (soft) reboot
- DO = HIGH (Relay, Close)  -->  DI = LOW
- DO = LOW (Relay, Open) -->  DI = HIGH
- PCA9535 PIN table

| DIO index   | GPIO PIN |
| ----------  | -------- |
| DI 0        |    0     |
| DI 1        |    1     |
| DI 2        |    2     |
| DI 3        |    3     |
| DI 4        |    4     |
| DI 5        |    5     |
| DI 6        |    6     |
| DI 7        |    7     |
| DO 0        |    8     |
| DO 1        |    9     |
| DO 2        |    10     |
| DO 3        |    11     |
| DO 4        |    12     |
| DO 5        |    13     |
| DO 6        |    14     |
| DO 7        |    15     |

- Example code for DIO
  ```bash=
  # Search for cp2112 device
  cat /sys/bus/i2c/devices/i2c-15/name
  # Probe & bind pca9535 to cp2112
  modprobe gpio-pca953x
  echo "pca9535 0x27" > /sys/bus/i2c/devices/i2c-15/new_device
  ```

---

## Cellular: mPCIe slot power control
- Default status is according to BIOS setting
- Power control:

| MPCIE PWR CTL |     GPIO PIN    |
| ------------  | --------------- |
|   SIO PIN     |       GP85      |

- Reset control:

| MPCIE RST CTL |     GPIO PIN    |
| ------------  | --------------- |
|   SIO PIN     |       GP86      |

- For power on mPCIe slot
  - Power ON -> wait 200 ms -> and RST ON
  - Set GP85 High, wait 200 ms, and Set GP86 High

- For power off mPCIe slot
  - RST OFF -> wait 200 ms -> and Power OFF
  - Set GP86 Low, wait 200 ms, and Set GP85 Low

- Example code
  1. Power on mPCIe slot: `mx-gpio-ctl set 85 1; sleep 0.2; mx-gpio-ctl set 86 1`
  2. Power off mPCIe slot: `mx-gpio-ctl set 86 0; sleep 0.2; mx-gpio-ctl set 85 0`

---

## Cellular: M.2 B Key B 

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

### Disalbe `NMI watchdog` driver
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
