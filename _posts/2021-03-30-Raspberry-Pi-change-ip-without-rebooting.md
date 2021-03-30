---
published: true
title: "Raspberry Pi change IP without rebooting"
excerpt: "Raspberry Pi change IP without rebooting"
date: 2021-03-30T00:00:00-04:00
show_date: true
tags:
  - networking
  - Raspbian
  - Raspberry Pi
---

💡 Even though the article below is targeted specifically at *Raspberry Pi OS (formerly called Raspbian)* the steps below should work perfectly fine for *any Debian based O/S*, as well as any other Linux distros which use *dhcpcd*.
{: .notice--success}

Configuring a static IP address in Raspberry Pi OS is pretty straight forward.
To do so we'll need to disable the default automatic configuration ([DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)) for the network interface in question.

## Adding a static configuration

Raspberry Pi OS *-like many other Linux distros-* uses [dhcpcd](https://wiki.archlinux.org/index.php/Dhcpcd) as its default DHCP client.  
The configuration file of **dhcpcd** is located at `/etc/dhcpcd.conf`.

Let's say you want to use a static IP address of `192.168.0.150`, simply add the following lines:

```shell
interface eth0
static ip_address=192.168.0.150/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.150 1.1.1.1 1.0.0.1
```

- `interface`: network interface for which you want to set a static IP (usually `eth0` or `wlan0`).
- `static ip_address`: your desired static IP in [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) notation. `/24` refers to netmask `255.255.255.0`.
- `static routers`: IP of your router / gateway.
- `static domain_name_servers`: your DNS server(s) of choice. As I'm running a [Pi-hole](https://pi-hole.net/) instance on the same device, I'm specifying the same IP followed by [Cloudflare](https://1.1.1.1/dns/)'s DNS servers which will serve as a backup should my Pi-hole instance be unreachable.

This is the part you would normally reboot your Raspberry Pi (by running `reboot` unsurprisingly 🤷‍♂️), in order to apply your changes.
Thankfully there *is* an easier way which doesn't require a reboot.

## Apply changes without rebooting

The command below will bring down the `eth0` interface, ⏸️ pause for exactly 5 seconds and automatically bring it back up afterwards:

```shell
sudo ifconfig eth0 down && sleep 5 && sudo ifconfig eth0 up
```

Verify whether the network interface has been configured by running `ifconfig`:

```shell
...
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    👉 inet 192.168.0.150  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 2a02:1811:c401:4d00:1eb1:5550:706c:b5d  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::df3d:ab2:eb31:e33d  prefixlen 64  scopeid 0x20<link>
        ether dc:a6:32:22:2d:fc  txqueuelen 1000  (Ethernet)
        RX packets 788462  bytes 137882009 (131.4 MiB)
        RX errors 65535  dropped 65535  overruns 0  frame 0
        TX packets 452092  bytes 85263482 (81.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...
```

You should now be able to reach your Raspberry Pi using the static IP you configured 👏.
