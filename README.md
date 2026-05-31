# HomeServer
Hello people, this is a repository in which i would share how i implemented my home server with all the explaination and config files.
Please feel free to add any suggestions by sending a pull request to make it better or if you have any questions please open an issue. 

Personal Raspberry Pi based homelab/server setup built around Docker, self-hosting, remote access, automation, and backups.

---

# Index

* [Overview](#overview)
* [Hardware](#hardware)
* [Software Stack](#software-stack)
* [Network Configuration](#network-configuration)
* [Docker Setup](#docker-setup)
* [Directory Structure](#directory-structure)
* [Services](#services)
* [Tailscale](#Tailscale)
* [SSH Configuration](#ssh-configuration)
* [Storage & NAS](#storage--nas)
* [Backups](#backups)
* [Home Assistant](#home-assistant)
* [Plex Media Server](#plex-media-server)
* [Security](#security)
* [Monitoring & Maintenance](#monitoring--maintenance)
* [Troubleshooting](#troubleshooting)
* [Future Improvements](#future-improvements)

---

# Overview

This repository contains the full documentation and configuration files for my personal homelab server.

The project focuses on:

* self-hosting
* secure remote access
* automation
* backups
* Docker containerization
* home integration
* media services

Each component is documented separately inside its dedicated folder and markdown file.

---

# Hardware

To build my homme server i needed something relatively cheap,(cuz i'm broke not stingy lol), small, that would make too much noise since it's gonna be in my room, and not too much electricity consuming.
But i didn't want to renounce quality, and reliabilty so i came up with a good compromise. 
For my hardware i choose a Raspberry pi 5 with 16 gb of ram, because i thought 8 we're not enough since i needed it to run multiple services and even though maybe 8 gigs could've done the job, i wanted toi have a big margin so that in the long term the hardwware would preserve better and prevent much heat and possible bottlenecks. 
Specs and docs of raspberry pi 5 here: https://pip-assets.raspberrypi.com/categories/892-raspberry-pi-5/documents/RP-008348-DS-6-raspberry-pi-5-product-brief.pdf 

-For the case, i wanted something functional but also ahestetic, so after some research i found the perfect option: The pironman 5 max 

https://docs.sunfounder.com/projects/pironman5/en/latest/pironman5_max/intro_pironman5_max.html

I choose the pironman 5 max for 3 main reasons: 
1) it's very ahesthetic, with it's colours and rgb fans and it's little built in screen
2) It has 2 nvme slots,which makes it very easy to fit a decent amount of storage, and also a lot of i/o ports, like usb-c's, ethernet. usb-a's, hdmi and more
3) It has multiple fans, which can dissipate heat very easily, and if configured correctly, they don't make any kind of noise.

-For the storage i choose 2 nvme ssd's
the first one is 1tb in which resides the os and all the configurations and also all of the docker containers for all the services i needed.
the second one is a crucial 2TB nvme, in which are gonna be stored all of the various data such as: photo's,backups,databases and more. 
most people are gonna say that only having the possibilty to use just nvme ssd's is gonna be a big limit for upgrades also moneywise.
but this server is meant to be personal,educational,for pure curiosity, and not for many users or big services.
So in the future if i will ever need to expand my server or maybe have one more professional, i think the best option is to stick to mechanical hard drives for storage and ssd's for the OS.
(Both of the ssd's had been formatted in exFat since i use both windows and linux based devices in my daily use, so i needed a universally file system for storing data and for samba to share them)
-As a power supply i choose the one reccomended by the pironman producers, which is not gonna be the classic one but it needs a little bit more wattage since it needs to power one more stuff than just the pi

---

# Software Stack
For the operating system there is a ton you can choose, one big requisite is that is doesn't have to be have if you need performance.
for example most of people choose operating system without a GUI because they never gonna use directly they are just gonna ssh into it.
Personally i do also ssh most of the time, but sometimes when i mess up something i open my closet and there is a big ass 24 inch samsung screen to control it directly as it gives me a slightly bigger sense of control in sitauition of panick. 
but one thing i knew for sure was that i wanted a linux based server because it gives you more freedom of costumization and full control over your hardware. 
So my final choice was a classic Raspberry pi OS.
it was my choice also because i didn't have a specific plan for my server when i started building it, but i knew i was gonna be multipurpose.
so OS like OpenMediaVault or trueNas i knew they we're a no-go since this was not gonna be just a nas.  

ok now i have my hardware ready, my os, but how tf should i host all of my services in just one server?
of course the only choice here was to use docker and built a container for each service.
i'm gonna explain more to it's configuration in other sections.
the main services i've built so far are: 
- printer server with CUPS
- docker and portainer to manage all containers
- nextcloud for storage
- homeassistant and homebridge to control my smarthome and migrate it to apple homekit, with the use of an ESP32 also
- samba for file sharing between winddows and linux
- plex for media streaming, although i might switch to jellyfin soon
- duplicati for data backup
- tailscale to access the server even when not at home 

---

# Network Configuration


This document covers the current network setup of my homelab, the decisions I made while building it, and some of the issues I ran into along the way.

The goal was to build a network that is easy to manage, secure enough for remote access, and flexible enough to support future services without constantly reworking the infrastructure.

Current topology:

```text
Internet
    │
    ▼
TIM Router
(192.168.1.1)
    │
    ▼
Network Switch
    │
    ▼
Raspberry Pi Server
(192.168.1.61)
    │
    ├── Docker
    ├── SSH
    ├── SMB
    ├── Plex
    ├── Duplicati
    └── Tailscale
```

## LAN Connectivity

The Raspberry Pi is currently connected through Ethernet.

When I first started building the server it was running over Wi-Fi because it was easier during the initial setup. It worked, but it also caused a few headaches when troubleshooting connectivity issues and figuring out why services were suddenly unreachable.

After rearranging the cabling and moving the server to a wired connection, things became much more predictable. Since then, file transfers, media streaming, backups and remote management have all been noticeably more reliable.

Current path:

```text
Raspberry Pi
    │
Ethernet
    │
Network Switch
    │
TIM Router
```

## IP Addressing and DHCP

The local network uses the standard private subnet:

```text
192.168.1.0/24
```

The router is available at:

```text
192.168.1.1
```

and the Raspberry Pi currently lives at:

```text
192.168.1.61
```

One of the first problems I encountered was caused by DHCP assigning a different address than the one I expected. I spent some time troubleshooting SSH before realizing that the server had simply obtained a new IP address.

To avoid this happening again, I configured a DHCP reservation on the router. The Raspberry Pi still uses DHCP, but it always receives the same address.

This makes service management much easier, especially for SSH, SMB shares, Docker services and any future infrastructure that relies on stable internal addressing.

## Remote Access

Originally I planned to deploy a traditional WireGuard server.

The more I looked into it, the more I realized it would require additional work such as:

- Port forwarding
- Dynamic DNS
- Managing public endpoints
- Manual client configuration

None of these are particularly difficult, but they add complexity that I didn't really need for a small homelab.

I eventually switched to Tailscale, which uses WireGuard under the hood while handling most of the annoying parts automatically.

This ended up being one of the best decisions in the project so far. Remote access became almost effortless and I no longer had to expose management services directly to the Internet.

Tailscale is currently installed on all devices that require access to the homelab, allowing me to securely connect to the server from anywhere while keeping everything behind a private mesh VPN.

## Firewall and Port Exposure

One of the main design goals of the homelab is keeping the attack surface as small as possible.

The router firewall remains enabled and, whenever possible, services are accessed through Tailscale instead of being exposed publicly.

Most administrative interfaces are intentionally kept private, including:

- SSH
- SMB shares
- Docker management interfaces
- Internal service dashboards

This means I can still access everything remotely, but without opening unnecessary ports to the Internet.

The general rule I follow is:

```text
If Tailscale can do it,
don't expose it publicly.
```

For the few services that require external access, port forwarding is configured directly on the router. Any exposed ports are documented together with the service that requires them so they can be audited and managed easily.

## SSH

SSH is the primary method used to administer the server.

Typical connection:

```bash
ssh loyalty@192.168.1.61
```

SSH is available both from the local network and through Tailscale, allowing secure remote administration without exposing port 22 to the public Internet.

One of the first networking issues I encountered involved failed SSH connections. At first I assumed SSH itself was misconfigured, but after some troubleshooting I discovered that the Raspberry Pi had simply received a different IP address through DHCP.

After implementing DHCP reservations, SSH access became consistent and reliable.

## Lessons Learned

Building the network infrastructure for this homelab taught me a few valuable lessons.

The first is that even in a small environment, proper IP management matters. DHCP reservations can save a surprising amount of troubleshooting time.

The second is that Ethernet is absolutely worth the effort. While Wi-Fi is convenient for testing and initial deployment, a wired connection provides the reliability that self-hosted services really need.

The third lesson is that simplicity often beats complexity. While deploying a standalone WireGuard server would have been a great learning experience, Tailscale allowed me to achieve the same goal with significantly less maintenance and configuration overhead.

Finally, I've learned that exposing fewer services publicly makes everything easier. Keeping management interfaces behind a VPN dramatically reduces the security concerns that come with running services from home.



---

# Docker Setup

## Docker Compose

All services are managed using Docker Compose.

Each service has its own dedicated directory containing the compose file, configuration files, and any service-specific documentation.

Current structure:

```text
/srv/docker
├── duplicati
├── homeassistant
├── nextcloud
└── plex
```

Each service can be deployed, updated, or removed independently without affecting the rest of the infrastructure.

Current self-hosted services include:

### Home Assistant

Home Assistant is used as the central platform for home automation and smart device management.

The container provides:

- Device integration
- Automation workflows
- Smart home dashboards
- Remote monitoring

Running Home Assistant inside Docker makes upgrades and backups significantly easier compared to a traditional installation.

### Plex

Plex serves as the media management and streaming platform for the homelab.

It is responsible for:

- Organizing media libraries
- Downloading metadata
- Managing users
- Streaming content across devices

Docker allows Plex to access media storage while keeping the application isolated from the host operating system.

### Nextcloud

Nextcloud acts as the private cloud platform of the homelab.

It is used for:

- File synchronization
- Remote file access
- Personal cloud storage
- Mobile device integration

Running Nextcloud in Docker simplifies deployment and will make future migrations considerably easier.

### Duplicati

Duplicati is responsible for automated backups of critical data.

It provides:

- Scheduled backups
- Encryption support
- Versioned backups
- Remote storage support

Its configuration and database are stored on persistent volumes to survive container recreation.

---

### Common Docker Compose Commands

Start a service:

```bash
docker compose up -d
```

Update a service:

```bash
docker compose pull
docker compose up -d
```

Stop a service:

```bash
docker compose down
```

Restart a service:

```bash
docker compose restart
```

Using Compose instead of manually managing containers provides a consistent deployment workflow and makes infrastructure management significantly easier.

As additional services are added to the homelab, they will follow the same structure and deployment methodology.
---

# Directory Structure

Documents the server filesystem layout and storage organization.

Example:

```text
/mnt/SERVERDATA
/srv/docker
```

---

# Services

Detailed configuration files and explanations for every hosted service.

---
# Tailscale

---
# Storage & NAS

Documentation for:

* SMB shares
* NAS integration
* mount points
* storage organization

---

# Backups

Backup infrastructure using:

* Duplicati
* encrypted backups
* scheduled jobs
* NAS backup targets

---

# Home Assistant

Smart home integration:

* HomeKit
* Matter
* Alexa coexistence
* smart devices
* automations

---

# Plex Media Server

Media server setup and streaming configuration.

---

# Security

Documents:

* firewall rules
* VPN-only access
* exposed ports
* secure practices

---

# Monitoring & Maintenance

Contains:

* logs
* diagnostics
* monitoring commands
* maintenance routines

---

# Troubleshooting

Collection of common issues and fixes encountered during setup and maintenance.

---

# Future Improvements

As the homelab grows, the networking stack will continue to evolve.

Planned improvements include:

- Reverse proxy deployment
- Internal DNS resolution
- HTTPS termination
- Domain-based service routing
- VLAN segmentation
- Network monitoring
- Intrusion detection and prevention
- Self-hosted mail infrastructure

These additions will be documented as they are implemented.

---

# Notes

This repository serves both as:

* documentation
* backup reference
* learning project
* reproducible setup guide

The setup will continue evolving over time.
