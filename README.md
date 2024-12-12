# Time Pi

TODO: Fancy picture goes here.

A Raspberry Pi stratum 1 time server.

Takes in GPS (or potentially other stratum 0 time sources), spits out NTP, PTP, etc.

## Setup

### Hardware

TODO.

### Software

Make sure you have Ansible installed. Copy the `example.hosts.ini` file to `hosts.ini` and customize it so Ansible can connect to your Pi.

Because I can't quite get `cmdline.txt` changes automated the way I like, _manually_ edit `/boot/firmware/cmdline.txt` and remove the portion `console=serial0,115200`, so GPS can use the serial port.

Now run the Ansible playbook:

```
ansible-playbook main.yml
```

> **Intel i226 Notes**: Currently I can't get the i226 to work with DHCP at all, so I have to manually set an IP address using `nmtui`. It also [doesn't work at 2.5 Gbps currently](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/674#issuecomment-2533117275), and it can't be overridden via Linux, so I make sure to plug it into a 1 Gbps port on my network.

## Usage

Some handy commands:

```
# GPS-related debugging
sudo systemctl status gpsd  # check gpsd status
gpsmon -n  # monitors gpsd output
cgps -s  # also monitors gps output

# PTP timestamping debugging
ethtool -T eth0  # or eth1, lists hardware clock info

# Chrony debugging
chronyc sources -v  # shows sources with documentation of fields
chronyc -n tracking  # shows detailed timing data

# Checking on NTP service from another computer
ntpdate -q [ip of grandmaster]
```

Much of the work that went into this project was documented in [this thread on the TimeHat v2](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/674).

## License

GPLv3 or Later

## Author

[Jeff Geerling](https://www.jeffgeerling.com), with assistance from Ahmad Byagowi and Oleg Obleukhov from Meta.
