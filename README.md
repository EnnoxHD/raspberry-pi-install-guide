# Raspberry Pi
## Initial setup
1. [Download](https://www.raspberrypi.com/software/operating-systems/) the `Raspberry Pi OS Lite` image
2. Insert SD card into PC
3. Use [Etcher](https://www.balena.io/etcher/) to [apply image](https://www.balena.io/etcher/#faq) to SD card
4. (Auto-)mount the partitions
5. Create an empty file `/boot/ssh` for SSH functionality at first start
6. Set a static IP in `/rootfs/etc/dhcpcd.conf` with:
```text
# ...
# Example static IP configuration:
interface eth0
static ip_address=192.168.10.10/24
# ...
```
7. Umount the partitions
8. Insert SD card into Raspberry Pi
9. Power the Rasperry Pi on
10. Wait a while
11. Configure a static IP on the PC Lan interface with
    an IP of `192.168.10.20` and a `255.255.255.0` subnet
12. Connect a Cross-Over-Cable / Switch between the PC and Raspberry Pi
13. Check if a connection is established (if not: restart the PC LAN interface)
14. Open Terminal on PC and SSH into the Raspberry Pi with `ssh pi@192.168.10.10`
15. On the first time allow establishing a ssh connection and type `yes`
16. Login as the user `pi` with the default password `raspberry`
17. Permanently enable the SSH server via `sudo raspi-config` then `3 Interface Options > P2 SSH > YES > OK > Finish > No`
18. Shutdown the Raspberry Pi `sudo poweroff`

## Change password
1. Connect to the Raspberry Pi
2. Change the password via `sudo passwd pi`
3. Type the new password and then retype again
4. The password is now changed
5. Shutdown the Raspberry Pi

## Enable Wi-Fi connection
1. Connect to the Raspberry Pi
2. Set the Wi-Fi country via `sudo raspi-config` then `5 Localisation Options > L4 WLAN Country`
3. Choose the country you are in e.g. `DE Germany > OK > OK`
4. Setup Wi-Fi connection (also via `sudo raspi-config`) `1 System Options > S1 Wireless LAN`
5. Enter your Wi-Fi name (`SSID > OK`)
6. Enter the `passphrase` for your Wi-Fi then `OK > Finish > No`
7. Shutdown the Raspberry Pi

## Update the software
1. Connect to the Raspberry Pi
2. Update the package list via `sudo apt update`
3. Upgrade the packages and any dependency changes via `sudo apt full-upgrade` then `Y`
4. Shutdown the Raspberry Pi

## Install DHCP server
1. Connect to the Raspberry Pi
2. Update the package list via `sudo apt update`
3. Install `sudo apt install dnsmasq` then `Y`
4. Configure the DHCP server in `/etc/dnsmasq.d/bridge.conf` with:
```text
interface=eth0
domain-needed
bogus-priv
dhcp-range=192.168.10.100,192.168.10.199,255.255.255.0,12h
```
5. Shutdown the Raspberry Pi
6. Configure DHCP on the PC Lan interface
7. Start the raspberry
8. Prove that DHCP works on the PC Lan interface with `ip address`
9. Shutdown the Raspberry Pi

## Configure IPv4 packet forwarding
1. Connect to the Raspberry Pi
2. Configure IPv4 packet forwarding in `/etc/sysctl.conf` with `net.ipv4.ip_forward=1`
3. Shutdown the Raspberry Pi

## Configure the firewall and routing
1. Connect to the Raspberry Pi
2. Update the package list via `sudo apt update`
3. Install `sudo apt install iptables-persistent` then `Y`
4. During install save the current IPv4 and IPv6 rules each with `Yes`
5. Configure iptables with the following commands:
```text
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -t filter -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -t filter -A FORWARD -i eth0 -o wlan0 -j ACCEPT
```
6. Save the iptables configuration with `sudo netfilter-persistent save`
7. Shutdown the Raspberry Pi
