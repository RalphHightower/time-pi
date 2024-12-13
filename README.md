# Time Pi

[![CI](https://github.com/geerlingguy/time-pi/actions/workflows/ci.yml/badge.svg)](https://github.com/geerlingguy/time-pi/actions/workflows/ci.yml)

TODO: Fancy picture goes here.

A Raspberry Pi stratum 1 time server.

Takes in GPS (or potentially other stratum 0 time sources), spits out NTP, PTP, etc.

## Setup

### Hardware

To run a decent time server (with high accuracy), you need a few things:

  - A computer running Linux (all the nice tooling for network time is availabe here)
  - A high quality time source (GPS is most commonly used these days)
  - A network adapter capable of hardware timestamping (Intel and ASIX make some good NICs for time-related applications)

For my own hardware configuration—which is the only one I've tested this repository with so far, I am using:

  - Computer: Raspberry Pi 5 model B
  - Time source: u-blox ZED-F9T-00B-01 (installed on [TimeHAT V2](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/674))
  - NIC: Intel i226-LM (installed on [TimeHAT V2](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/674))

GPS also requires a decent antenna with as clear a view of the sky as possible, to establish a precise GPS lock. Some GPS receivers are better than others, when it comes to timing accuracy, but almost any (even USB receivers) will do better than NTP!

I'm using a small [active GPS antenna](https://amzn.to/4gdhBj1), but for permanent installation it is best to mount a quality GPS antenna outside, clear of obstructions.

### Software

Make sure you have Ansible installed. Copy the `example.hosts.ini` file to `hosts.ini` and customize it so Ansible can connect to your Pi.

Because I can't quite get `cmdline.txt` changes automated the way I like, _manually_ edit `/boot/firmware/cmdline.txt` and remove the portion `console=serial0,115200`, so GPS can use the serial port.

Now run the Ansible playbook:

```
ansible-playbook main.yml
```

This playbook will configure:

  - [GPSd](https://gpsd.gitlab.io/gpsd/gpsd.html): interface with the u-blox GPS and provide GPS data to other applications
  - [Chrony](https://chrony-project.org): NTP server, which also sync GPS time (with Internet NTP server backup) to the system clock
  - [Linux PTP](https://linuxptp.nwtime.org): synchronize the system clock to the NIC PHC (Physical Hardware Clock), and set up the Pi as a PTP grandmaster clock

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
