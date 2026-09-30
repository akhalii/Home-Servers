# Proxmox VE Node Documentation

This document outlines the services, infrastructure roles, and configuration details for the primary **Proxmox VE** hypervisor node.

## Node Overview

* **Host Role:** Primary Hypervisor (VMs & LXC Containers)

* **Network Role:** Tailscale Exit Node

* **DNS Role:** Upstream Recursive Resolver (Unbound DNS) & Secondary Ad-Blocking DNS (AdGuard Home 2)

## Networking & DNS Routing Flow

```
                      [ Tailscale Mesh ]
                              │
                              ▼
                   ┌─────────────────────┐
                   │ Proxmox VE Host     │
                   │ (Tailscale Exit Node)│
                   └──────────┬──────────┘
                              │
       ┌──────────────────────┴──────────────────────┐
       ▼                                             ▼
┌───────────────────────────┐             ┌───────────────────────────┐
│ AdGuard Home 2            │             │ Hosted Services & LXCs    │
│ (Secondary DNS Filter)    │             │ • Homepage                │
└──────────────┬────────────┘             │ • Actual Budget           │
               │ (Upstream)               │ • AMP Game Server         │
               ▼                          │ • FreshRSS                │
┌───────────────────────────┐             │ • Wiki.js                 │
│ Unbound DNS               │             │ • Beszel                  │
│ (Recursive DNS Resolver)  │             │ • LubeLogger              │
└──────────────┬────────────┘             └─────────────────────────┘
               │
               ▼
       [ Root DNS Servers ]

```

1. **Tailscale Remote Access:** The host acts as an exit node for remote traffic routing.

2. **AdGuard Home 2:** Filters DNS requests and forwards queries upstream to **Unbound DNS**.

3. **Unbound DNS:** Performs full recursive DNS lookups directly against root DNS servers for max privacy and security.

## Hosted Services & Applications

| Service | Category | Container / Deployment Type | Description | 
 | ----- | ----- | ----- | ----- | 
| **Homepage** | Infrastructure | LXC / Docker | Unified dashboard organizing all homelab services and live system metrics. | 
| **AdGuard Home 2** | Network & Security | LXC | Secondary DNS sinkhole for network-wide ad-blocking and filtering. | 
| **Unbound DNS** | Network & Security | LXC | Local recursive DNS resolver handling upstream queries for AdGuard instances. | 
| **Beszel** | Monitoring | LXC / Docker | Lightweight system monitor tracking performance metrics and uptime. | 
| **Wiki.js** | Docs & Media | LXC / Docker | Modern wiki engine for homelab documentation and notes. | 
| **FreshRSS** | Docs & Media | LXC / Docker | Self-hosted RSS aggregator and reader. | 
| **Actual Budget** | Miscellaneous | LXC / Docker | Local-first, privacy-focused personal budgeting application. | 
| **AMP Dashboard** | Miscellaneous | LXC / VM | Game server management portal powering game server instances. | 
| **LubeLogger** | Miscellaneous | LXC / Docker | Self-hosted vehicle maintenance and garage record tracker. | 

## Management & Operations

* **LXC Management:** Managed via `pct` CLI commands or Proxmox VE Web UI.

* **Backups:** Daily/weekly automated backups executed via Proxmox Backup Server / VZDump to secondary storage.