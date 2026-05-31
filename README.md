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
* [Security](#security)
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

---

# Hardware

To build my home server, I needed something relatively cheap (because I'm broke, not stingy 😄), compact, quiet enough to stay in my bedroom, and reasonably power efficient.

At the same time, I didn't want to sacrifice reliability or overall quality, so I tried to find a good compromise between performance, cost, and usability.

For the main hardware, I chose a Raspberry Pi 5 with 16 GB of RAM. While 8 GB would probably have been enough for my current workload, I wanted some extra headroom for future services and long-term scalability.

Having more available memory allows me to run multiple services simultaneously without worrying too much about bottlenecks, and it gives me more flexibility as the project grows.

Raspberry Pi 5 specifications:

https://pip-assets.raspberrypi.com/categories/892-raspberry-pi-5/documents/RP-008348-DS-6-raspberry-pi-5-product-brief.pdf

---

## Case

For the case, I wanted something that was both functional and aesthetically pleasing.

After some research, I found what I considered the perfect option: the **Pironman 5 Max**.

Documentation:

https://docs.sunfounder.com/projects/pironman5/en/latest/pironman5_max/intro_pironman5_max.html

I chose the Pironman 5 Max for three main reasons:

1. **Design and aesthetics**  
   It looks fantastic, featuring RGB fans, a built-in display, and a design that makes the Raspberry Pi feel like a miniature desktop server.

2. **Storage and expandability**  
   It includes two NVMe slots, making it easy to install a decent amount of storage while keeping everything compact. It also offers plenty of I/O, including USB-C, USB-A, HDMI, and Gigabit Ethernet.

3. **Cooling performance**  
   The multiple integrated fans provide excellent cooling. When properly configured, they keep temperatures under control while remaining virtually silent.

---

## Storage

For storage, I chose two NVMe SSDs.

### Primary Drive

The first SSD is a 1 TB NVMe drive and contains:

- Raspberry Pi OS
- Docker containers
- Service configurations
- Application data

### Secondary Drive

The second SSD is a 2 TB Crucial NVMe drive and is dedicated to storing:

- Photos
- Backups
- Databases
- Media files
- General user data

Many people would argue that being limited to NVMe storage is not ideal for future upgrades, especially from a cost-per-terabyte perspective.

However, this server was built as a personal learning project rather than a production environment. It is intended for self-hosting, experimentation, and education rather than serving many users or running large-scale services.

If I ever outgrow this setup and move to a more professional server, I would most likely use:

- SSDs for the operating system and containers
- Mechanical hard drives for mass storage

Both SSDs were originally formatted using exFAT because I regularly work across both Windows and Linux systems and wanted a filesystem that would be easy to access from either platform.

---

## Power Supply

For power delivery, I chose the power supply recommended by the Pironman manufacturer.

The Pironman includes additional hardware compared to a standard Raspberry Pi setup, so it requires slightly more power than the official Raspberry Pi power supply alone would normally provide.

Using the recommended PSU ensures stable operation, especially when using NVMe drives, cooling fans, and USB peripherals simultaneously.

---

# Software Stack

There are many operating systems available for home servers, but one important requirement was that it should remain lightweight and efficient.

Many people choose operating systems without a graphical interface because they rarely interact with the machine directly and instead manage everything through SSH.

I also use SSH for most administrative tasks, but occasionally I break something and need direct access to the machine. In those situations, I simply open my closet, where I have a 24-inch Samsung monitor connected to the Raspberry Pi.

Being able to interact directly with the system gives me a little more confidence when troubleshooting problems or recovering from mistakes.

One thing I knew from the start was that I wanted a Linux-based operating system. Linux provides a high degree of customization, flexibility, and control over the hardware.

For those reasons, I ultimately chose **Raspberry Pi OS**.

Another reason for this choice was that I didn't have a precise plan for the server when I started building it. I only knew that it would be multipurpose.

Because of that, more specialized operating systems such as OpenMediaVault or TrueNAS were not ideal for my use case. While both are excellent solutions, this project was never intended to be just a NAS.

---

## Containerization

Once I had the hardware and operating system ready, the next question was:

**How do I host multiple services on a single machine while keeping everything organized?**

The obvious answer was Docker.

Docker allows each service to run inside its own isolated container, making deployment, updates, troubleshooting, and maintenance significantly easier.

I will cover the Docker configuration in more detail in later sections.

---

## Current Services

The services currently running on the server include:

- **CUPS** — Printer server
- **Docker & Portainer** — Container management
- **Nextcloud** — Personal cloud storage
- **Home Assistant** — Smart home automation
- **Homebridge** — Apple HomeKit integration
- **ESP32 devices** — Custom smart home integrations
- **Samba (SMB)** — File sharing between Windows and Linux systems
- **Plex** — Media streaming platform
- **Duplicati** — Backup management
- **Tailscale** — Secure remote access VPN

I may eventually migrate from Plex to Jellyfin, but for now Plex remains my primary media server solution.

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

To keep the system organized and make backups easier, all persistent data is stored under a single root directory:

```text
/mnt/SERVERDATA
```

Rather than letting each application store data in random locations across the operating system, every service uses dedicated directories inside this storage hierarchy.

Current structure:

```text
/mnt/SERVERDATA
├── appdata
│   ├── duplicati
│   ├── homeassistant
│   ├── nextcloud
│   └── plex
├── backups
├── media
│   ├── Movies
│   ├── TV Shows
│   └── Anime
└── shares
```

This layout keeps application configuration, media files, backups and shared data separated and easy to manage.

---

## AppData Directory

The `appdata` directory contains all persistent container data.

Each service has its own dedicated folder where configuration files, databases, metadata and application-specific data are stored.

Example:

```text
/mnt/SERVERDATA/appdata/plex
/mnt/SERVERDATA/appdata/nextcloud
/mnt/SERVERDATA/appdata/homeassistant
/mnt/SERVERDATA/appdata/duplicati
```

Keeping application data outside Docker containers ensures that services can be recreated, updated or migrated without losing configuration.

---

## Media Storage

Media files are stored separately from application data.

Example:

```text
/mnt/SERVERDATA/media
├── Movies
├── TV Shows
└── Anime
```

This directory is mounted into Plex containers and can easily be expanded as the media library grows.

Separating media from application data simplifies backup strategies and future migrations.

---

## Backup Storage

Backups generated by Duplicati are stored in:

```text
/mnt/SERVERDATA/backups
```

Keeping backups isolated from application data makes recovery procedures easier and reduces the risk of accidental data loss.

---

## SMB Shares

The storage hierarchy is exposed through Samba shares, allowing files to be accessed from Windows, macOS, Linux and mobile devices.

This provides a centralized storage location for:

- Documents
- Media files
- Backups
- Configuration exports
- Project files

The SMB shares are accessible both from the local network and remotely through Tailscale.

---

## Permissions Management

Ownership of the storage directory is assigned to the primary server user:

```bash
sudo chown -R loyalty:loyalty /mnt/SERVERDATA
```

Docker containers use matching PUID and PGID values to ensure proper access permissions:

```yaml
environment:
  - PUID=1000
  - PGID=1000
```

This prevents permission conflicts between the host operating system and containers.

---

## Why This Structure?

The storage layout was designed with a few goals in mind:

- Easy backups
- Easy migration to new hardware
- Clear separation of services
- Simplified troubleshooting
- Consistent Docker bind mounts

By centralizing all persistent data under a single directory tree, the entire homelab can be backed up, restored or migrated with minimal effort.

---

# Services

For config files examples see docker compose files 

---


# Security

Security has been one of the main considerations while building this homelab.

The server itself is a Raspberry Pi housed in a Pironman case and located directly in my bedroom. While this is far from an enterprise environment, I still wanted to follow good security practices from the beginning and build a setup that is both secure and easy to manage.

The overall philosophy is simple:

```text
Keep it simple.
Expose as little as possible.
Access everything through a VPN.
```

The goal isn't to build military-grade security, but rather to protect the services I use daily while learning about networking, self-hosting and system administration.

---

# Security Goals

The primary security objectives are:

- Secure remote access
- Minimize public exposure
- Protect administrative interfaces
- Reduce the attack surface
- Keep management simple and maintainable

Rather than exposing multiple services directly to the Internet, the homelab is designed around private access whenever possible.

---

# Tailscale VPN

One of the most important security decisions was adopting Tailscale as the primary remote access solution.

Initially, I considered deploying a standalone WireGuard server, but this would have required:

- Port forwarding
- Dynamic DNS
- Public endpoint management
- Additional firewall configuration

Tailscale provides WireGuard-based encryption while eliminating most of that complexity.

Benefits include:

- End-to-end encrypted connections
- No exposed VPN ports
- Automatic NAT traversal
- Device-based authentication
- Secure remote access from anywhere

All administrative tasks are performed through Tailscale whenever possible.

---

# Firewall Strategy

The router firewall remains enabled at all times.

The goal is to allow only the traffic that is actually required while blocking unnecessary inbound connections.

General principles:

- Default deny through NAT
- Minimize open ports
- Restrict management interfaces
- Prefer VPN access over public exposure

This significantly reduces the risk of automated scans and brute-force attacks.

---

# Port Exposure

Most services are intentionally kept private and are only accessible through:

- Local Network (LAN)
- Tailscale VPN

Examples include:

- SSH
- SMB Shares
- Docker management interfaces
- Internal dashboards
- Administrative panels

Whenever a service requires public accessibility, only the specific required port is exposed.

Every forwarded port should have a documented purpose and should be reviewed periodically.

The general rule I follow is:

```text
If Tailscale can do it,
don't expose it publicly.
```

---

# SSH Security

SSH is the primary management interface of the server.

Typical connection:

```bash
ssh loyalty@192.168.1.61
```

SSH is not exposed directly to the Internet.

Instead, access is performed through:

```text
Local Network
or
Tailscale VPN
```

This eliminates the majority of common SSH attacks.

Future improvements may include:

- Public key authentication
- Disabling password authentication
- Fail2Ban integration
- SSH hardening

---

# Docker Isolation

All services run inside Docker containers.

Containerization provides a basic layer of isolation between applications and the host operating system.

Benefits include:

- Service separation
- Easier recovery
- Reduced dependency conflicts
- Simplified updates
- Cleaner management

Persistent application data is stored outside containers using bind mounts.

This allows containers to be recreated without losing data.

---

# Data Protection

Application data is stored under:

```text
/mnt/SERVERDATA
```

This central storage structure simplifies:

- Backups
- Permission management
- Recovery procedures
- Future migrations

Service configurations, databases and user data remain persistent even if containers are recreated.

---

# Backup Strategy

Backups are managed through Duplicati.

The backup system provides:

- Automated backup jobs
- Encryption support
- Version history
- Recovery capabilities

Backup data is stored separately from application data whenever possible.

This separation helps protect against accidental deletion and simplifies restoration procedures.

---

# Permission Management

Ownership of persistent storage is assigned to the primary server user.

Example:

```bash
sudo chown -R loyalty:loyalty /mnt/SERVERDATA
```

Docker containers use matching user and group identifiers:

```yaml
environment:
  - PUID=1000
  - PGID=1000
```

This prevents common permission issues between the host system and containers.

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
