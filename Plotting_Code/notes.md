## AQ Raspberry Pi

### Password

Password for `pi` user changed to:

```
kpaqrpi4
```

### GPS Time

Use GPS device as a time source.

`gpsd` already installed and running.

Config file `/etc/chrony/chrony.conf` already contains:

```
refclock SHM 0 offset 0.5 delay 0.2 refid NMEA
```

System time is set from GPS once up and running / signal received.

GPS signal can be monitored with:

```
gpsmon
```

`chrony` time source can be viewed with:

```
chronyc sources
```

GPS source appears as `NMEA`.

### DHCP Server

Raspberry Pi has wired network interface st to use address:

```
192.168.4.1
```

Configured in `/etc/dhcpcd.conf`:

```
interface eth0
inform 192.168.4.1
noipv6
```

Pi runs `dnsmasq` for DHCP services. Config file is at:

```
/etc/dnsmasq.conf
```

Relevant bits:

```
dhcp-mac=set:client_is_a_pi,B8:27:EB:*:*:*
dhcp-reply-delay=tag:client_is_a_pi,2

interface=eth0
listen-address=192.168.4.1
bind-interfaces
bogus-priv
dhcp-range=192.168.4.10,192.168.4.19,2h
dhcp-option=3
```

Service is set to try and restart every 5 seconds, as it will fail to run when a network cable is not connected.

Service file is copied in to `/etc/systemd/service/`:

```
cp /lib/systemd/system/dnsmasq.service /etc/systemd/system/
```

Then, adjusted to always restart:

```
Restart=always
RestartSec=5
```

Service enabled / started via `systemd` with:

```
systemctl daemon-reload # re-reads service files
systemctl enable dnsmasq
systemctl start dnsmasq
```

Status can be checked with:

```
systemctl status dnsmasq
```

### System Updates

System fully updated with:

```
apt-get update
apt-get upgrade
apt-get dist-upgrade
apt-get autoremove
reboot
```

### Sensor Device Naming

Usb port numbers from udev:

```
           1-1.1.2    1-1.3
           |-----|  |-----|
           |-----|  |-----|
-------
| eth |    |-----|  |-----|
|  0  |    |-----|  |-----|
-------    1-1.1.3    1-1.2
```

Alternative port number for different Raspberry Pi models:

```
           1-1.2      1-1.4
           |-----|  |-----|
           |-----|  |-----|
-------
| eth |    |-----|  |-----|
|  0  |    |-----|  |-----|
-------    1-1.3      1-1.5
```

`udev` rules set to assign device names `/dev/ttySDS01` (top left), `/dev/ttySDS02` (bottom left), `/dev/ttySDS03` (top right), `/dev/ttySDS04` (bottom right).

Rules file created at `/etc/udev/rules.d/99-sensors.rules`, containing:

```
# Port 1-1.2 = SDS01:
KERNEL=="ttyUSB*", KERNELS=="1-1.2", ATTRS{idProduct}=="7523", ATTRS{idVendor}=="1a86", SYMLINK+="ttySDS01"
# Port 1-1.3 = SDS02:
KERNEL=="ttyUSB*", KERNELS=="1-1.3", ATTRS{idProduct}=="7523", ATTRS{idVendor}=="1a86", SYMLINK+="ttySDS02"
# Port 1-1.4 = SDS03:
KERNEL=="ttyUSB*", KERNELS=="1-1.4", ATTRS{idProduct}=="7523", ATTRS{idVendor}=="1a86", SYMLINK+="ttySDS03"
# Port 1-1.5 = SDS04:
KERNEL=="ttyUSB*", KERNELS=="1-1.5", ATTRS{idProduct}=="7523", ATTRS{idVendor}=="1a86", SYMLINK+="ttySDS04"
# Port 1-1.1.3 = SDS03:
KERNEL=="ttyUSB*", KERNELS=="1-1.1.3", ATTRS{idProduct}=="7523", ATTRS{idVendor}=="1a86", SYMLINK+="ttySDS03"
# Port 1-1.1.2 = SDS04:
KERNEL=="ttyUSB*", KERNELS=="1-1.1.2", ATTRS{idProduct}=="7523", ATTRS{idVendor}=="1a86", SYMLINK+="ttySDS04"
```

When devices are connected, they are assigned a standard `/dev/ttyUSB` name, with the `/dev/ttySDS` name being created as a symlink, e.g.:

```
crw-rw---- 1 root dialout 188,  0 Jun 29 14:31 /dev/ttyUSB0
lrwxrwxrwx 1 root root          7 Jun 29 14:31 /dev/ttySDS04 -> ttyUSB0
crw-rw---- 1 root dialout 188,  1 Jun 29 14:31 /dev/ttyUSB1
lrwxrwxrwx 1 root root          7 Jun 29 14:31 /dev/ttySDS01 -> ttyUSB1
crw-rw---- 1 root dialout 188,  2 Jun 29 14:31 /dev/ttyUSB2
lrwxrwxrwx 1 root root          7 Jun 29 14:31 /dev/ttySDS03 -> ttyUSB2
```

### Existing Logging Processes

Disabled by commenting the following lines in `~pi/.bashrc`:

```
#if [[ ! $(pgrep -f pm2pt5_monitoring_script.sh) ]]; then
#    ./pm2pt5_monitoring_script.sh
#fi
```

`cron` job disabled for `pi` user:

```
###* * * * * /home/pi/pm2pt5_monitoring_script.sh
```

