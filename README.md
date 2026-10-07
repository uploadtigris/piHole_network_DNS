# Pi-hole Network DNS

A Raspberry Pi running [Pi-hole](https://pi-hole.net/) as the DNS server for my
home network, filtering ads and trackers for every device before queries ever
leave the LAN. It's the first service in my homelab and the one that taught me
the most about DNS, DHCP, and troubleshooting infrastructure you actually depend
on every day.

- **Part of:** [my_home_lab](https://github.com/uploadtigris/my_home_lab)
- **Portfolio:** [uploadtigris.github.io](https://uploadtigris.github.io)

---

## What it does

Pi-hole is a DNS sinkhole: it sits between every device on the network and the
upstream resolvers, answering queries for ad/tracker domains with a dead end and
forwarding everything else on. One box, configured once, cleans up browsing for
every phone, laptop, and TV on the network — no per-device client needed.

- Network-wide ad and tracker blocking at the DNS layer
- Upstream resolution via Cloudflare (`1.1.1.1`), with Google (`8.8.8.8` / `8.8.4.4`) as fallback
- Block lists: StevenBlack's Unified Hosts list plus [HaGeZi's Pro list](https://github.com/hagezi/dns-blocklists) (~170k+ additional domains)
- Query logging and the Pi-hole admin dashboard for visibility into what's being asked for

---

## Hardware & software

| | |
|---|---|
| **Device** | Raspberry Pi 2 Model B |
| **OS** | Raspberry Pi OS Lite (32-bit), headless |
| **Connection** | Wired Ethernet to the router for stability |
| **Addressing** | Static IP so every client can rely on it as DNS |

Full step-by-step build (imaging the SD card, enabling SSH, installing Pi-hole,
setting the static IP, and pointing the router at it) is written up here:
**[Pi-hole Setup Guide →](docs/01_setup-guide.md)**

---

## How it fits the network

The router hands out Pi-hole's address as the DNS server for the whole LAN, so
every device resolves through it by default. After the home network is segmented
into VLANs, Pi-hole moves to the **Servers** VLAN and serves DNS to every other
VLAN through a single documented firewall exception (*"any VLAN may reach Pi-hole
on port 53"*). See
[network-segmentation-ids](https://github.com/uploadtigris/network-segmentation-ids).

---

## Troubleshooting write-ups

Running DNS for the whole house means when Pi-hole breaks, the internet "breaks"
for everyone — which makes it a great teacher. Two real failures I diagnosed and
wrote up as I went:

1. **The DNS server that quietly stopped resolving.**
   After a restart, the Pi came up on a *new DHCP address* instead of holding its
   static IP, which broke DNS network-wide — the tell was exactly one query per
   hour showing up in the logs. I tracked it down, pinned the address properly,
   and wrote up the fix.
   → [LinkedIn write-up](https://www.linkedin.com/feed/update/urn:li:activity:7434388898182594561/)

2. **The "404" that turned out to be a corrupted install.**
   The admin console started returning a 404 and `apt upgrade` threw package
   errors. I power-cycled, confirmed the Pi was reachable with `arp -a`, forced a
   filesystem check (`sudo touch /forcefsck`), and worked through
   `apt --fix-broken install` and `dpkg --configure -a`. The root cause was a
   corrupted Pi-hole install, not a dying SD card, so reinstalling from the
   official script brought it back — no reflash needed.
   → [LinkedIn write-up](https://www.linkedin.com/feed/update/urn:li:activity:7418760395584155648/)

Both are examples of how I document every fix: what broke, how I found it, what I
changed, and what I'd watch next time.

---

## What's in this repo

| Path | What it holds |
|---|---|
| `README.md` | What it does and how it fits the network |
| [`docs/01_setup-guide.md`](docs/01_setup-guide.md) | Full build: SD card, SSH, install, static IP, router DNS |
| [`docs/02_reinstall-after-power-outage.md`](docs/02_reinstall-after-power-outage.md) | Rebuilding the Pi after a power outage |
| [`images/`](images/) | Screenshots |

---

## Everyday commands

```bash
# Find the Pi on the network (ping sweep, no port scan)
sudo nmap -sn 192.168.1.0/24

# Update Pi-hole
pihole -up

# Live-tail the DNS query log
pihole -t

# Set / reset the admin password
sudo pihole setpassword
```

Admin dashboard: `http://<pi-hole-ip>/admin`

---

## Skills this project exercises

`DNS` · `DHCP & static addressing` · `nmap` · `arp` · `SSH` · `Raspberry Pi` ·
`Linux (Debian) administration` · `troubleshooting & documentation`
