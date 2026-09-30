# Home Server Infrastructure & Homelab

Welcome to my personal home-server repository. This repository documents the architecture, self-hosted services, storage pools, network routing, and deployment configurations across my homelab infrastructure.

## Infrastructure Overview

The homelab consists of two primary bare-metal nodes running in a hyper-converged setup linked via a mesh VPN network:

* [**Proxmox VE**](./proxmox_ve_documentation.md)**:** Hypervisor hosting LXCs/VMs for core utilities, management dashboards, media readers, and recursive DNS.

* [**TrueNAS**](./truenas_documentation.md)**:** NAS and Docker host providing ZFS mirror storage pools, cloud sync, photo management, remote desktop servers, and client PC backup repositories.

```
                         ┌─────────────────────────────────┐
                         │     Tailscale Mesh Network      │
                         └────────────────┬────────────────┘
                                          │
                  ┌───────────────────────┴───────────────────────┐
                  ▼                                               ▼
┌───────────────────────────────────┐           ┌───────────────────────────────────┐
│       Proxmox VE Node             │           │          TrueNAS Node             │
│    (See proxmox.md for details)   │           │    (See truenas.md for details)   │
├───────────────────────────────────┤           ├───────────────────────────────────┤
│ • AdGuard Home 2 (Secondary DNS)  │           │ • AdGuard Home 1 (Primary DNS)    │
│ • Unbound DNS (Recursive Resolver)│           │ • Dockge (Docker Stack Manager)   │
│ • Homepage (Unified Dashboard)    │           │   └── RustDesk Server             │
│ • Actual Budget                   │           │ • Immich (Photo & Video)          │
│ • AMP Game Server                 │           │ • Nextcloud (Cloud Data/Files)    │
│ • FreshRSS                        │           │ • ZFS Pool 1: 2TB Mirror (Apps)   │
│ • Wiki.js                         │           │ • ZFS Pool 2: 8TB Mirror (Data)   │
│ • Beszel Stats                    │           │ • Veeam Backup Target (SMB)       │
│ • LubeLogger                      │           │                                   │
└───────────────────────────────────┘           └───────────────────────────────────┘

```

## Sub-System Documentation

Detailed specs, network topologies, and service tables are split into host-specific guides:

* [**Proxmox VE Documentation**](./proxmox_ve_documentation.md)

  *Covers hypervisor setup, LXC containers, application details, and local Unbound recursive DNS config.*

* [**TrueNAS Documentation**](./truenas_documentation.md)

  *Covers ZFS storage pools (2TB & 8TB mirrors), Dockge/Docker Compose stacks, Nextcloud/Immich storage layout, and weekly Veeam PC backup configuration.*

## Network & Security Architecture

### 1. Remote Access & Mesh VPN

* **Tailscale** is deployed across all machines, containers, and mobile devices.

* Both **Proxmox VE** and **TrueNAS** act as **Exit Nodes** on the Tailnet for secure remote routing.

### 2. Private, Network-Wide DNS Flow

* **Tailscale DNS:** Points directly to **AdGuard Home 1** (TrueNAS) and **AdGuard Home 2** (Proxmox) for ad-blocking and local name resolution across all devices.

* **Upstream Recursion:** Both AdGuard Home instances forward non-blocked queries to **Unbound DNS** running on Proxmox, which performs direct recursive lookups against global root servers for privacy.

## Storage & Backup Strategy

| Target | Mechanism | Frequency | Storage Pool | 
 | ----- | ----- | ----- | ----- | 
| **PC / Workstation** | Veeam Agent (Image Backup) | Weekly Incremental | TrueNAS 8TB Pool (Mirror) | 
| **TrueNAS Data** | ZFS Datasets & Snapshots | Automated Schedule | TrueNAS Pools | 
| **Proxmox Workloads** | Proxmox Backup / VZDump | Scheduled | Secondary Storage | 
