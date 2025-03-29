# Time Pi

[![CI](https://github.com/geerlingguy/time-pi/actions/workflows/ci.yml/badge.svg)](https://github.com/geerlingguy/time-pi/actions/workflows/ci.yml)

<p align="center"><img alt="Raspberry Pi 5 with TimeHAT V2" src="/resources/time-pi.jpeg" height="auto" width="600"></p>

A Raspberry Pi stratum 1 time server.

Takes in GPS (or potentially other stratum 0 time sources), spits out NTP, PTP, etc.

## Setup

### Hardware

To run a decent time server (with high accuracy), you need a few things:

  - A computer running Linux (all the nice tooling for network time is availabe here)
  - A high quality time source (GPS is most commonly used these days)
  - A network adapter capable of hardware timestamping (Intel and ASIX make some good NICs for time-related applications)

There are many options for each of the above items—for example, many use [Adafruit's Ultimate GPS HAT](https://www.adafruit.com/product/2324) or its [USB equivalent](https://www.adafruit.com/product/4279) for GPS acquisition, and for PTP, you can use a Compute Module 4 or 5's built-in NIC (with [PPS in or out](https://www.jeffgeerling.com/blog/2022/ptp-and-ieee-1588-hardware-timestamping-on-raspberry-pi-cm4)), or add on a compatible NIC on the Pi 5 with something like the [uPCIty Lite](https://amzn.to/4iUn9ke).

My own hardware configuration—which is the basis for the code in this repository, consists of:

  - Computer: Raspberry Pi 5 model B
  - Time source: u-blox ZED-F9T-00B-01 (installed on [TimeHAT V4](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/674))
  - NIC: Intel i226-LM (installed on [TimeHAT V4](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/674))

A precise GPS signal for nanosecond-accurate time requires a decent antenna with as clear a view of the sky as possible. Some GPS receivers are better than others, but even USB receivers will do better than NTP!

I use a small [active GPS antenna](https://amzn.to/4gdhBj1) for testing, but for permanent installation it is best to mount a [higher-quality GPS antenna](https://www.meinbergglobal.com/english/products/gps-glonass-l1-antenna.htm) outside, clear of obstructions.

### Software

Make sure you have Ansible installed. Copy the following example files and customize them according to your setup:

  - `example.hosts.ini` to `hosts.ini`
  - `example.config.yml` to `config.yml`

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

# Check on NTP service from another computer
ntpdate -q [ip of grandmaster]

# Check on PTP clocks and offsets (assuming ptp4l is running)
wget https://tsn.readthedocs.io/_downloads/f329e8dec804247b1dbb5835bd949e6f/check_clocks.c
gcc -o check_clocks check_clocks.c
sudo ./check_clocks -d eth0  # or eth1 (the interface you're using for PTP)
```

Much of the work that went into this project was documented in [this thread on the TimeHat v2](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/674).

## Slave / Client Setup

For PTP, you need to install and configure PTP for Linux on slave/client machines, and synchronize them to the master/server node as well.

An example configuration for a slave/client node is set up in `ptp-client-node.yml`, and further examples may be provided in the future.

## Other Hobbyist Time Servers

  - [Austin's Nerdy Things: Nanosecond accurate PTP Pi server](https://austinsnerdythings.com/2025/02/18/nanosecond-accurate-ptp-server-grandmaster-and-client-tutorial-for-raspberry-pi/)
  - [Andreas Spiess: NTP Server from GPS Satellites](https://www.youtube.com/watch?v=RKRN4p0gobk)
  - [Jeff Geerling: Time Card mini for GPS and OXCO on Pi](Time Card mini adds Pi, GPS, and OCXO to your PC)

## License

GPLv3 or Later

## Author

[Jeff Geerling](https://www.jeffgeerling.com), with assistance from Ahmad Byagowi and Oleg Obleukhov from Meta.
