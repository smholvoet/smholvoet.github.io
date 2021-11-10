---
published: true
title: "Upgrading your Raspberry Pi to Bullseye"
excerpt: "Upgrading your Raspberry Pi to Bullseye"
date: 2021-11-10T00:00:00-04:00
show_date: true
tags:
  - Raspberry Pi
  - Raspbian
  - buster
  - bullseye
---

Raspberry Pi recently released a new version of their Raspberry Pi OS, namely [Bullseye](https://www.raspberrypi.com/news/raspberry-pi-os-debian-bullseye/).
As you can see below, this upgrade is in line with Raspberry Pi's release policy where the OS receives a major update every 2 years:

| Release date   | Debian version    |
|----------------|------------|
| 📅 September 2013 | 📦 Wheezy     |
| 📅 September 2015 | 📦 Jessie     |
| 📅 August 2017    | 📦 Stretch    |
| 📅 June 2019      | 📦 Buster     |
| 📅 **November 2021**  | 🆕 **Bullseye** 👈 |

This gets the OS back on track with the latest [Debian](https://wiki.debian.org/DebianReleases#Production_Releases) release (*Debian 11* aka *Debian bullseye*), which was released 2 months ago.

## Checking current version

My Raspberry Pi 3 Model B was still running the previous version (*Debian buster*):

```bash
lsb_release -a
```

```bash
No LSB modules are available.
Distributor ID: Raspbian
Description:    Raspbian GNU/Linux 10 (buster)
Release:        10
Codename:       buster
```

This also meant that the Linux kernel version also seemed to be pretty outdated 😱:

```bash
uname -r
```

```bash
5.4.79-v7+
```

This means that in my case I will be upgrading from *buster* to *bullseye*. I haven't tried upgrading from any older versions.

⚠️ **Warning:** Raspberry Pi [recommends](https://www.raspberrypi.com/news/raspberry-pi-os-debian-bullseye/) downloading the [newest](https://www.raspberrypi.com/software/operating-systems/) image and reinstalling instead of *manually* upgrading your system. Follow the below steps at your own risk and make sure you've backed up your SD card before getting started.
{: .notice--warning}

> With a major upgrade, we recommend downloading a new image, reinstalling any applications, and moving your data across from your current image. Debian major version upgrades contain a lot of changes, and it is very easy for some small tweak made somewhere in the system to be incompatible with some change you have made, and you can end up with a broken system and a Raspberry Pi that won’t boot.
{: .notice--warning}

## Upgrading

### Update package sources

Let's first update the [Apt](https://wiki.debian.org/Apt) package sources. In my case they were still pointing to the *buster* ones. We need to make sure new and already installed packages use the *bullseye* version.

Edit the `/etc/apt/sources.list` file and replace `buster` by `bullseye`:

```bash
deb http://raspbian.raspberrypi.org/raspbian/ buster main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://raspbian.raspberrypi.org/raspbian/ buster main contrib non-free rpi
```

**old:**

```bash
deb http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi
```

**new:**

```bash
deb http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://raspbian.raspberrypi.org/raspbian/ buster main contrib non-free rpi
```

Repeat the same steps for any other `.list` files located in the `/etc/apt/sources.list.d/` directory. In my case I had to make changes in 3 files:

```text
...
📂etc
 └── 📂apt
      ├── 📝sources.list 👈
      └── 📂sources.list.d
          ├── 📝docker.list 👈
          └── 📝raspi.list 👈
```

## Update packages

Update the package list from the repository and check for any new versions. This will show you how many packages *can* be updated, but won't actually update them:

```bash
sudo apt update
```

```bash
Get:1 http://raspbian.raspberrypi.org/raspbian bullseye InRelease [15.0 kB]
Get:2 http://archive.raspberrypi.org/debian bullseye InRelease [23.5 kB]
Get:3 https://download.docker.com/linux/raspbian bullseye InRelease [26.7 kB]
Get:4 http://raspbian.raspberrypi.org/raspbian bullseye/main armhf Packages [13.2 MB]
Get:5 http://archive.raspberrypi.org/debian bullseye/main armhf Packages [200 kB]
Get:6 https://download.docker.com/linux/raspbian bullseye/stable armhf Packages [5,486 B]
Get:7 http://raspbian.raspberrypi.org/raspbian bullseye/contrib armhf Packages [60.2 kB]
Get:8 http://raspbian.raspberrypi.org/raspbian bullseye/non-free armhf Packages [106 kB]
Get:9 http://raspbian.raspberrypi.org/raspbian bullseye/rpi armhf Packages [1,360 B]
Fetched 13.7 MB in 9s (1,561 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
443 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

## Install packages

Install 2 additional packages, which I suppose are needed to successfully complete the upgrade:

- 📦 [libgcc-8-dev](https://packages.debian.org/buster/libgcc-8-dev)
- 📦 [gcc-8-base](https://packages.debian.org/buster/gcc-8-base)

Run the following command to do so:

```bash
sudo apt install libgcc-8-dev gcc-8-base
```

## Full upgrade

During this step we'll actually upgrade the O/S, update our packages and remove any currently installed packages which are needed to successfully complete the upgrade.

```bash
sudo apt full-upgrade
```

This process might take a while, and you will most likely get a couple of warnings like the one below:

![bullseye-update-window](/assets/images/bullseye-update-window.png)

Go through any warnings and make sure you understand them before hitting `YES`.

## Final touches

Once the actual upgrade has been completed we still need to modify the `/boot/config.txt` file:

```bash
sudo nano /boot/config.txt
```

Scroll down to the bottom of the file and do the following:

- comment out the line containing `dtoverlay=vc4-fkms-v3d`
- add a line containing `dtoverlay=vc4-kms-v3d` at the very bottom of the file

**old:**

```bash
...

[pi4]
# Enable DRM VC4 V3D driver on top of the dispmanx display stack
dtoverlay=vc4-fkms-v3d
max_framebuffers=2

[all]
#dtoverlay=vc4-fkms-v3d
```

**new:**

```bash
...

[pi4]
# Enable DRM VC4 V3D driver on top of the dispmanx display stack
#dtoverlay=vc4-fkms-v3d
max_framebuffers=2

[all]
#dtoverlay=vc4-fkms-v3d
dtoverlay=vc4-kms-v3d
```

Save your changes and finally reboot your device:

```bash
sudo shutdown -r now
```

Wait a few moments for your Raspberry Pi to finish rebooting. If all went well, your Raspberry Pi should come up again and you'll now be running 🎯 *bullseye*:

```bash
lsb_release -a
```

```bash
No LSB modules are available.
Distributor ID: Raspbian
Description:    Raspbian GNU/Linux 11 (bullseye)
Release:        11
Codename:       bullseye
```

```bash
uname -r
```

```bash
5.10.63-v7+
``````
