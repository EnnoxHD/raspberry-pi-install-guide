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

Passwort ändern:
```bash
sudo passwd pi
<Passwort>
<Passwort bestätigen>
```

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

## Softwareupdate
```bash
sudo apt-get update
sudo apt-get --with-new-pkgs upgrade
```

## Funktion: "WLAN-to-LAN"-Bridge mit eigenem Subnetz
Änderungen entfernen:
```bash
sudo nano /etc/dhcpcd.conf
```

> **Quelle: https://willhaley.com/blog/raspberry-pi-wifi-ethernet-bridge/**

```bash
sudo apt-get install dnsmasq
sudo mkdir -p /etc/iptables
sudo nano /etc/iptables/rules.v4
```
```text
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o wlan0 -j MASQUERADE
COMMIT

*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i eth0 -o wlan0 -j ACCEPT
COMMIT
```
```bash
sudo nano /etc/network/if-up.d/iptables
```
```text
#!/bin/sh
iptables-restore < /etc/iptables/rules.v4
```
```bash
sudo chmod +x /etc/network/if-up.d/iptables
sudo nano /etc/sysctl.conf
```
Zeile ändern:
```text
net.ipv4.ip_forward=1
```
```bash
sudo nano /etc/network/interfaces.d/eth0
```
```text
auto eth0
allow-hotplug eth0
iface eth0 inet static
  address 192.168.10.10
  netmask 255.255.255.0
  gateway 192.168.10.10
```
```bash
sudo nano /etc/dnsmasq.d/bridge.conf
```
```text
interface=eth0
bind-interfaces
server=1.1.1.1
domain-needed
bogus-priv
dhcp-range=192.168.10.100,192.168.10.199,12h
```
