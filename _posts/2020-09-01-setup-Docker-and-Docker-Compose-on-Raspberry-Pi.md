---
published: true
title: "Setup Docker and Docker Compose on a Raspberry Pi "
excerpt: "Setting up Docker and Docker Compose on a Raspberry Pi"
date: 2020-09-01T00:00:00-04:00
tags:
  - Raspberry Pi
  - Raspbian
  - Docker
  - Docker Compose
---

## Introduction

This post will show you how to install Docker and Docker-Compose on a clean Raspberry Pi:

- `docker` cli is typically used to manage **individual containers**
- `docker-compose` cli on the other hand is used to manage **multi-container** applications

Full definition of *Docker Compose*:
> Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.

![image-center](/assets/images/docker_rpi.png){: .align-center}

## Raspberry Pi OS setup

### O/S installation

Refer to the official 📖 [Raspberry Pi](https://www.raspberrypi.org/documentation/) documentation which goes through all required steps to get your RPi up & running. I'll quickly summarize the steps which I've gone through.

- Download & install [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/)
- Select **Raspberry Pi OS (32-bit) Lite** if you'll be running a [headless](https://en.wikipedia.org/wiki/Headless_software) install. Alternatively, use the "Full" version if you want the desktop version.
- Add an empty `ssh` file to the root of SD card. This enables SSH on first boot.
- Connect via SSH using the default credentials:
  - user: `pi`
  - password: `raspberry`

### Upgrade packages

By now you should've succesfully connected to your RPi via SSH. Next up, we'll update our packages:

Update package list:

```bash
sudo apt update
```

Check which packages can be upgraded:

```bash
apt list --upgradable
```

Upgrade installed packages to latest version:

```bash
sudo apt full-upgrade
```

## Docker installation

Docker itself provides a so called [*convenience script*](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script) which automatically detects your distribution and takes care of all dependencies for you. Before blindly downloading & running it, have a 🕵 look at what's inside [here](https://github.com/docker/docker-install/blob/master/install.sh). These are the steps I used to install Docker:

Download Docker's install script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

Run it 🚀 and let it work its magic:

```bash
sudo sh get-docker.sh
```

We won't be needing this script anymore, so delete it:

```bash
rm get-docker.sh
```

### Add non-root user to the Docker group

In order to run Docker commands without adding `sudo` to all your commands, we'll add the current user to the `docker` security group.

This will add the `pi` user to the `docker`group:

```bash
sudo usermod -aG docker pi
```

We'll have to logout and log back in before this will actually work, so type:

```bash
logout
```

### Verify Docker install

Show the Docker version information:

```bash
docker version
```

Run a test container:

```bash
docker run hello-world
```

This should output something similar:

```text
...

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

...
```

Remove the image:

```bash
docker image rm hello-world
```

## Docker Compose install

Installing `docker-compose` using the `apt` package manager will fail as Raspberry Pi's `arm` architecture is currently (August 2020) not supported 🤷‍♂.

If you do feel brave and try and try using `apt-get`, you'll most likely end up with the following error when you try running `docker-compose`:

```bash
/usr/local/bin/docker-compose: line 1: Not: command not found
```

Use the below `pip` installation method instead:

```bash
sudo apt-get install -y libffi-dev libssl-dev
sudo apt-get install -y python3 python3-pip
sudo apt-get remove python-configparser
sudo pip3 -v install docker-compose
```

### Verify Docker Compose install

Running `docker-compose version` should output something like this:

```bash
pi@raspberrypi:~/docker $ docker-compose version
docker-compose version 1.26.2, build unknown
docker-py version: 4.3.1
CPython version: 3.7.3
OpenSSL version: OpenSSL 1.1.1d  10 Sep 2019
```

You can now run & manage Docker containers using Docker and/or Docker Compose 🐋!
