# TrueNAS Node Documentation

This document outlines the services, storage infrastructure, ZFS pool topology, backup target configurations, and Docker containers hosted on the **TrueNAS** node.

## Node Overview

* **Host Role:** Primary Network Attached Storage (NAS) & Container Host

* **Network Role:** Tailscale Exit Node

* **DNS Role:** Primary Ad-Blocking DNS Server (AdGuard Home 1)

* **Container Management:** Dockge (Docker Compose GUI)

## Storage Architecture & ZFS Pools

The storage layer is configured using ZFS Mirroring (RAID 1 equivalency) across two dedicated pools for redundancy and performance isolation.

```
┌────────────────────────────────────────────────────────────────────────┐
│                          TrueNAS Storage Pools                         │
├───────────────────────────────────┬────────────────────────────────────┤
│           2TB ZFS Pool            │            8TB ZFS Pool            │
│          (Mirror / RAID 1)        │          (Mirror / RAID 1)         │
├───────────────────────────────────┼────────────────────────────────────┤
│ • Core Application Data           │ • Immich Photo/Video Library       │
│ • Docker Stack Volumes            │ • Nextcloud Primary Storage        │
│ • AdGuard Home 1 Configs          │ • Veeam PC Backup Target (SMB/NFS) │
└───────────────────────────────────┴────────────────────────────────────┘

```

### Storage Datasets

1. **2TB Pool (Mirror -** $2 \times 2\text{TB}$ **HDDs/SSDs):**

   * High-IOPS container persistent data (Dockge, RustDesk, AdGuard Home 1).

   * System datasets and configuration back-ups.

2. **8TB Pool (Mirror -** $2 \times 8\text{TB}$ **HDDs):**

   * High-capacity media and cloud storage (Immich & Nextcloud data stores).

   * Dedicated SMB/NFS share serving as the target repository for **Veeam Agent** backups.

## Client Backup Operations (Veeam Agent)

* **Source:** Workstation / PC

* **Backup Client:** Veeam Agent for Microsoft Windows / Linux

* **Destination:** TrueNAS 8TB Pool (SMB Share)

* **Schedule:** Weekly Incremental Backups (with scheduled synthetic full backups)

* **Retention Policy:** Automated block-level retention managed on the SMB repository target.

## Network & Container Architecture

```
                      [ Tailscale Mesh ]
                              │
                              ▼
                   ┌─────────────────────┐
                   │ TrueNAS Host        │
                   │ (Tailscale Exit Node)│
                   └──────────┬──────────┘
                              │
       ┌──────────────────────┼──────────────────────┐
       ▼                      ▼                      ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────────┐
│ AdGuard Home 1│     │ ZFS Pools     │     │ Dockge            │
│ (Primary DNS) │     │ • 2TB Mirror  │     │ (Docker Compose)  │
└───────┬───────┘     │ • 8TB Mirror  │     └─────────┬─────────┘
        │ (Upstream)  └───────┬───────┘               │
        ▼                     │                       ▼
 [ Unbound DNS ]              ▼              ┌───────────────────┐
 (on Proxmox)        [ Veeam Backup Share ]  │ RustDesk          │
                                             └───────────────────┘

```

1. **AdGuard Home 1:** Serves as the primary DNS server for Tailnet and local devices.

2. **Dockge Integration:** Manages Docker Compose stacks cleanly with an interactive UI.

3. **RustDesk:** Runs as a Docker Compose stack managed via Dockge to enable self-hosted remote desktop control.

## Hosted Services & Storage Applications

| Service | Category | Host / Stack Type | Target Storage Pool | Description | 
 | ----- | ----- | ----- | ----- | ----- | 
| **AdGuard Home 1** | Network & Security | Native / App | 2TB Pool | Primary network-wide DNS sinkhole and ad-blocking filter. | 
| **Dockge** | Infrastructure | Docker Container | 2TB Pool | Reactive interface for editing, starting, and building Docker Compose stacks. | 
| **RustDesk** | Remote Access | Compose Stack (Dockge) | 2TB Pool | Self-hosted remote desktop signaling and relay server. | 
| **Immich** | Docs & Media | App / Container | 8TB Pool | Self-hosted photo and video backup solution with AI indexing. | 
| **Nextcloud** | Docs & Media | App / Container | 8TB Pool | Personal cloud server providing storage, file synchronization, and sharing. | 
| **Veeam Endpoint** | Backup Target | SMB Share | 8TB Pool | Dedicated target dataset for weekly workstation disk image backups. | 

## Maintenance & Storage Operations

* **ZFS Health:** Scrub tasks scheduled bi-weekly across both 2TB and 8TB pools.

* **ZFS Snapshots:** Periodic snapshots enabled on the 8TB pool to protect against ransomware or accidental deletion of Veeam backups.

* **Docker Management:** Container updates and environment configurations handled directly inside Dockge (`docker-compose.yml`).