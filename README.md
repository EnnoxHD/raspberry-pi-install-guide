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
sudo nano /etc/network/interfaces
```
WLAN über dhcpcd.conf:
```bash
sudo nano /etc/dhcpcd.conf
```
```text
interface wlan0
static ip_address=<IP-Adresse>/<Netmask Bits>
static routers=<Router IP>
static domain_name_servers=<DNS Server A> <DNS Server B>
```

> **Quelle: https://willhaley.com/blog/raspberry-pi-wifi-ethernet-bridge/** für Folgendes

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

## Festplatte einbinden
```bash
sudo apt-get install ntfs-3g
sudo fdisk -l
sudo ls -l /dev/disk/by-uuid
sudo mkdir -p /mnt/usb
sudo nano /etc/fstab
```
Hinzufügen:
```text
UUID=<UUID> /mnt/usb ntfs-3g uid=pi,gid=pi 0 0
```

## Funktion: PXE-Server
> **Quelle: https://www.youtube.com/watch?v=EVtBHgPHf30**

```bash
sudo apt-get install syslinux-common
sudo nano /etc/dnsmasq.conf
```
Anfügen:
```text
dhcp-boot=pxelinux.0,192.168.10.10
pxe-service=x86PC,"PXE Boot",pxelinux,192.168.10.10
enable-tftp
tftp-root=/mnt/usb/pxe
```
- Download von `pxelinux.0` über https://www.debian.org/distrib/netinst#netboot
- Download von `ipxe.lkrn` für ArchLinux über https://www.archlinux.org/releng/netboot/
- Download von `archlinux.iso` über https://www.archlinux.org/download/
```bash
cd /mnt/usb/pxe
curl -O http://ftp.nl.debian.org/debian/dists/buster/main/installer-amd64/current/images/netboot/pxelinux.0
sudo cp /usr/lib/syslinux/modules/bios/ldlinux.c32 ./
sudo cp /usr/lib/syslinux/modules/bios/libutil.c32 ./
sudo cp /usr/lib/syslinux/modules/bios/menu.c32 ./
sudo cp /usr/lib/syslinux/memdisk ./
mkdir -p ./archlinux-installer/bios/
mkdir -p ./archlinux-installer/iso/
cd ./archlinux-installer/bios/
curl -O https://www.archlinux.org/static/netboot/ipxe.419cd003a298.lkrn
mv ipxe.419cd003a298.lkrn ipxe.lkrn
cd ../iso
curl -O https://mirror.netcologne.de/archlinux/iso/2020.05.01/archlinux-2020.05.01-x86_64.iso
mv archlinux-2020.05.01-x86_64.iso archlinux.iso
cd ../..
sudo mkdir -p ./pxelinux.cfg
sudo nano ./pxelinux.cfg/default
```
```text
DEFAULT menu.c32
ALLOWOPTIONS 0
PROMPT 0
TIMEOUT 0

MENU TITLE PXE Boot Menu

LABEL archlinux
  MENU LABEL ArchLinux (over Internet, iPXE)
  KERNEL /archlinux-installer/bios/ipxe.lkrn

LABEL archlinux-iso
  MENU LABEL ArchLinux (local, ISO)
  KERNEL /memdisk
  APPEND iso initrd=/archlinux-installer/iso/archlinux.iso raw
```

## Funktion: Samba-Share
> **Quelle: https://raspberrypihq.com/how-to-share-a-folder-with-a-windows-computer-from-a-raspberry-pi/**

```bash
sudo apt-get install samba samba-common-bin
sudo nano /etc/samba/smb.conf
```
Share einrichten:
```text
[pxe]
 comment=PXE-Server Quellen
 path=/mnt/usb/pxe
 browseable=yes
 writeable=yes
 only guest=no
 create mask=0777
 directory mask=0777
 public=yes
```
