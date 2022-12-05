# Setting up Raspberry Pi 4

- Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- Run the app
- Select `Raspberry Pi OS (other)` option
- Select `Raspberry Pi OS Lite (64-bit)` with no desktop environment option
- Select `Storage`
- Select `Gear` button
- In `Advanced Options`, input
	- hostname
	- Check Enable SSH
		- Select `User password authentication`
	- Set username and password
	- Configure wireless LAN
		- SSID
		- Password
		- Wireless LAN Country
	- Set locale settings

- Click 'Write'
- Once finish eject the SD Card

# Boot Raspberry Pi
- Insert the SD card in the raspberry pi
- Wait a few minutes
- SSH into the pi, using `ssh` command.

```bash
ssh cranberry@10.0.0.104
```
- Update package lists

```bash
sudo apt update
```
- Remove network manager package, which may cause randomize of your MAC address

```bash
sudo apt remove network-manager
```

- Upgrade packages

```bash
sudo apt upgrade
```

* Install VIM

```bash
sudo apt install vim
```

* Get the list of network interfaces, using `ifconfig` command

```bash
ifconfig

eth0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether e4:5f:01:c2:36:e9  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 10  bytes 1588 (1.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10  bytes 1588 (1.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.104  netmask 255.255.255.0  broadcast 10.0.0.255
        inet6 fe80::8599:7c13:1030:bc48  prefixlen 64  scopeid 0x20<link>
        inet6 fd1c:bf21:3e42:1:486a:cb99:5af9:19a2  prefixlen 64  scopeid 0x0<global>
        ether e4:5f:01:c2:36:ea  txqueuelen 1000  (Ethernet)
        RX packets 98042  bytes 133970347 (127.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 37753  bytes 3683114 (3.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

`wlan0` is wireless network card and under it the value after `ether` is its MAC address `e4:5f:01:c2:36:ea`

## Set the static IP address to wireless network card

* Modify `/etc/dhcpcd.conf` file with `vim` and add the following content at the end of the file

```text
timeout 5
noipv6

interface wlan0
static ip_address=10.0.0.104/24
static routers=10.0.0.1
static domain_name_servers=1.1.1.1 1.0.0.1
```

* Reboot Raspberry PI

```bash
sudo reboot now
```

# Reboot Raspberry Pi

```bash
sudo shutdown -r now
```

# Shutdown Raspberry Pi

```bash
sudo shutdown -h now
```

# Troubleshooting

## Check boot time

Use `systemd-analyze` command

```bash
systemd-analyze blame

8.101s hciuart.service
5.853s dhcpcd.service
1.734s dev-mmcblk0p2.device
1.336s raspi-config.service
 918ms rpi-eeprom-update.service
 644ms systemd-udev-trigger.service
 608ms systemd-fsck@dev-disk-by\x2dpartuuid-286e6ee5\x2d01.service
...
```

```bash
systemd-analyze critical-chain

The time when unit became active or started is printed after the "@" character.
The time the unit took to start is printed after the "+" character.

multi-user.target @10.661s
└─dhcpcd.service @4.807s +5.853s
  └─basic.target @4.510s
    └─sockets.target @4.509s
      └─triggerhappy.socket @4.508s
        └─sysinit.target @4.488s
          └─systemd-timesyncd.service @3.959s +528ms
            └─systemd-tmpfiles-setup.service @3.753s +178ms
              └─local-fs.target @3.701s
                └─boot.mount @3.653s +46ms
                  └─systemd-fsck@dev-disk-by\x2dpartuuid-286e6ee5\x2d01.service @3.034s +608ms
                    └─dev-disk-by\x2dpartuuid-286e6ee5\x2d01.device @2.993s
```

