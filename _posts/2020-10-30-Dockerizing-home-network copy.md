---
published: false
title: "Dockerizing my home network: pihole, Unifi, portainer & watchtower"
excerpt: "Dockerizing my home network: pihole, Unifi, portainer & watchtower"
date: 2020-10-30T00:00:00-04:00
tags:
  - networking
  - UniFi
  - pihole
  - DNS
---

## Introduction

I recently moved over all of my networking related applications (mainly pihole & Unifi) to Docker, which runs on top of my [Raspberry Pi 4 Model B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/). I've created a `docker-compose` file which throws together all my required applications. Make sure you have Docker Docker-Compose installed (see [Setup Docker and Docker Compose on a Raspberry Pi
](https://sanderh.dev/setup-Docker-and-Docker-Compose-on-Raspberry-Pi/))

[`docker-compose.yaml`](github.com)

```yaml
version 3.8
....
```

- [Watchtower](#watchtower)
- [Portainer](#portainer)
- [Pi-hole](#pi-hole)
- [Unifi](#unifi)
- [RPI-monitor](#rpi-monitor)

## Watchtower

- 📝 https://github.com/containrrr/watchtower
- 🐋 https://hub.docker.com/r/containrrr/watchtower

## Portainer

- 📝 https://github.com/portainer/portainer
- 🐋 https://hub.docker.com/r/portainer/portainer

## pi-hole

- 📝 https://github.com/pi-hole/pi-hole
- 🐋 https://hub.docker.com/r/pihole/pihole/

## Unifi

- 📝 https://github.com/jacobalberty/unifi-docker
- 🐋 https://hub.docker.com/r/jacobalberty/unifi

## Rpi-monitor

- 📝 https://github.com/XavierBerger/RPi-Monitor
- 🐋 https://hub.docker.com/r/michaelmiklis/rpi-monitor
