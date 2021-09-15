# Wi-Fi dial on Sparklan WPEQ-261ACNI
- Support Module List:
  - [WPEQ-261ACNI(BT)](https://www.sparklan.com/product/wpeq-261acnibt-qca6174a-mu-mimo-industrial-grade-module/)

- Ensure DIP switch status is 'OFF' status on target slot

## Build atheros-ath-driver
- For kernel 4.9
  - https://github.com/Moxa-Linux/moxa-atheros-ath-driver/tree/4.9.198/master
- For kernel 4.19
  - https://github.com/Moxa-Linux/moxa-atheros-ath-driver/tree/4.19.171/master

- Build drivers
```
apt update && apt install build-essential linux-headers-`uname -r`
cd moxa-atheros-ath-driver
make -C /lib/modules/$(uname -r)/build M=$(pwd)/ath modules
cp ath/ath10k/ath10k_core.ko /lib/modules/$(uname -r)/kernel/drivers/net/wireless/ath/ath10k/
cp ath/ath10k/ath10k_pci.ko /lib/modules/$(uname -r)/kernel/drivers/net/wireless/ath/ath10k/
depmod -a
```

## check ath10k driver is loaded

```bash=
$ dmesg | grep ath
[    6.704652] ath10k_pci 0000:03:00.0: irq 134 for MSI/MSI-X
[    6.704664] ath10k_pci 0000:03:00.0: pci irq msi oper_irq_mode 2 irq_mode 0 reset_mode 0
[    6.931468] ath10k_pci 0000:03:00.0: qca6174 hw3.2 target 0x05030000 chip_id 0x00340aff sub 168c:3363
[    6.931471] ath10k_pci 0000:03:00.0: kconfig debug 0 debugfs 1 tracing 0 dfs 0 testmode 0
[    6.931841] ath10k_pci 0000:03:00.0: firmware ver WLAN.RM.4.4.1-00140-QCARMSWPZ-1 api 6 features wowlan,ignore-otp,mfp crc32 29eb8ca1
[    6.999859] ath10k_pci 0000:03:00.0: board_file api 2 bmi_id N/A crc32 4ac0889b
[    7.569153] ath10k_pci 0000:03:00.0: Unknown eventid: 3
[    7.585056] ath10k_pci 0000:03:00.0: Unknown eventid: 118809
[    7.587960] ath10k_pci 0000:03:00.0: Unknown eventid: 90118
[    7.588799] ath10k_pci 0000:03:00.0: htt-ver 3.60 wmi-op 4 htt-op 3 cal otp max-sta 32 raw 0 hwcrypto 1
[    7.658686] ath: EEPROM regdomain: 0x6c
[    7.658689] ath: EEPROM indicates we should expect a direct regpair map
[    7.658692] ath: Country alpha2 being used: 00
[    7.658693] ath: Regpair used: 0x6c
```

## Dial up with wpa_supplicant
https://wiki.centos.org/zh-tw/HowTos/Laptops/WpaSupplicant

- Install necessary packages:
* For [Ubuntu 20.04] / [Debian 9/10/11]
```bash=
apt-get install net-tools wpa_supplicant -y
```
* For [CentOS 7.9]
```bash=
yum install net-tools wpa_supplicant -y
```

```bash=
# check Wi-Fi module interface name, e.g. 'wlp3s0'
ip a | grep wlp
# 4: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
#    inet 192.168.126.241/24 brd 192.168.126.255 scope global dynamic wlp3s0

# edit /etc/wpa_supplicant/wpa_supplicant.conf, e.g.
vim /etc/wpa_supplicant/wpa_supplicant.conf

network={
        ssid="SSID"
        psk="12345678"
        proto=RSN
        key_mgmt=WPA-PSK
        pairwise=CCMP
}

# then start wpa_supplicant and use dhclient to get ip
ifconfig wlp3s0 up
wpa_supplicant -B -i wlp3s0 -Dnl80211 -c /etc/wpa_supplicant/wpa_supplicant.conf
dhclient -v wlp3s0
```
