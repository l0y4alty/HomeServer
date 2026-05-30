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

Contains:

* router configuration
* DHCP reservations
* port forwarding
* local network setup
* Wi-Fi/Ethernet configuration

---

# Docker Setup

Contains:

* Docker installation
* Docker Compose files
* container management
* networking
* volumes and bind mounts

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

# SSH Configuration

Contains:

* SSH hardening
* key authentication
* remote management
* troubleshooting

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

Planned additions:

* reverse proxy
* HTTPS
* cloud backups
* monitoring dashboards
* UPS integration
* RAID/NAS expansion

---

# Notes

This repository serves both as:

* documentation
* backup reference
* learning project
* reproducible setup guide

The setup will continue evolving over time.
