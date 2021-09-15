# Linux-x86-Platform
Documents for setup x86 platform on Linux Distributions

## Products Page Links
[DA-681A-I-WL (CN)](/products/DA-681A-I-WL.md)<br>

## Linux Distribution Setup
[CentOS-7.9](/dist/CentOS-7.9.md)<br>
[Ubuntu-20.04](/dist/Ubuntu-20.04.md)<br>
[Debian-11 (bullseye)](/dist/Debian-11.md)

## RF Modules
### Cellular
[Telit LE910C4](/cellular/telit/LE910C4.md)
### Wi-Fi
[Sparklan WPEQ-261ACNI(BT)](/wifi/sparklan/WPEQ-261ACNI.md)

## How to export super IO (IT87 series) GPIO pin

- Check drivers are loaded
    1. Ensure `gpio_it87` driver is loaded on Linux OS
    2. Ensure `it87_serial` driver is loaded on Linux OS

- Convert GPIO pin index to gpio sysfs:
    1. Check IT87 gpio chip index
    ```bash=
    # cat /sys/class/gpio/gpiochip448/label
    gpio_it87
    # it means IT87 gpio chip base index is started from 448
    ```
    2. Convert GPIO pin index to gpio-sysfs index
        - `GP[n][m] = base + (n - 1) * 8 + m`
        - e.g.
            - GP35 on IT87 gpio chip base 448:
              gpio-sysfs index = 448 + (3 - 1) * 8 + 5 = `469`
    3. Export gpio-sysfs index 469
    ```bash=
    echo 469 > /sys/class/gpio/export
    ```
    4. Set GPIO value (high/low)
    ```bash=
    # Set High value
    echo "high" > /sys/class/gpio/gpio469/direction
    # Set Low value
    echo "low" > /sys/class/gpio/gpio469/direction
    ```
    4. Get GPIO value
    ```bash=
    cat /sys/class/gpio/gpio469/value
    # if return 0 = low
    # if return 1 = high
    ```

- An example code for set/get Super IO gpio-sysfs value from GPIO pin

[mx-gpio-ctl](tools/mx-gpio-ctl)

	- Set GP14 value as 0: `mx-gpio-ctl set 14 0`
	- Get GP14 value: `mx-gpio-ctl get 14`
