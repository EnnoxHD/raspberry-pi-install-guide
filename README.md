# Raspberry Pi Installation
## SSH + Static IP
```bash
lsblk # nachfolgend X
mount /dev/sdX2 /mnt
mount /dev/sdX1 /mnt/boot
sudo nano /mnt/boot/ssh
umount /dev/sdX1
sudo nano /mnt/etc/dhcpcd.conf
umount /dev/sdX2
```

## Logindaten
| Eigenschaft     | Wert        |
| --------------- | ----------- |
| **user**        | `pi`        |
| **password**    | `raspberry` |

## WLAN
```bash
sudo nano /etc/network/interfaces
```
```text
auto wlan0
allow-hotplug wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
```
```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```
```text
network={
  ssid="<WLAN SSID>"
  proto=RSN
  key_mgmt=WPA-PSK
  pairwise=CCMP TKIP
  group=CCMP TKIP
  psk="<WLAN Passwort>"
}
```

## Optional: WLAN Powermanagement aus
```bash
sudo nano /etc/modprobe.d/8192cu.conf
```
```text
options 8192cu rtw_power_mgnt=0 rtw_enusbss=0 rtw_ips_mode=1
```
