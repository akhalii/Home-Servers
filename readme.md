# 🏠 Personal Homelab Infrastructure

A production-ready home infrastructure setup focused on self-hosting, network-wide ad-blocking, privacy, data storage, and media management.

---

## 🗺 Network & Infrastructure Architecture

```
                      [ Tailscale Mesh Network ]
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
┌─────────────────────────────────┐               ┌─────────────────────────────────┐
│     Proxmox VE (Exit Node)      │               │       TrueNAS (Exit Node)       │
├─────────────────────────────────┤               ├─────────────────────────────────┤
│ • AdGuard Home 2 (Primary DNS)  │               │ • AdGuard Home 1 (Secondary DNS)│
│ • Unbound DNS (Recursive)       │               │ • Dockge (Docker Composer)      │
│ • Homepage                      │               │   └── RustDesk                  │
│ • Actual Budget                 │               │ • Immich                        │
│ • AMP Game Server               │               │ • Nextcloud                     │
│ • FreshRSS                      │               └─────────────────────────────────┘
│ • Wiki.js                       │
│ • Beszel                        │
│ • LubeLogger                    │
└─────────────────────────────────┘
```

### DNS & Remote Access Flow
1. **Tailscale:** Deployed across all nodes, LXCs, and VMs for secure, mesh-encrypted remote access. Proxmox and TrueNAS both serve as network **Exit Nodes**.
2. **Global DNS Routing:** Tailscale Tailnet DNS points directly to **AdGuard Home 1 & 2** for network-wide ad-blocking and local domain resolution.
3. **Upstream Resolution:** Both AdGuard Home instances use **Unbound DNS** on Proxmox for recursive, privacy-focused DNS resolution directly against root servers.

---

## 💻 Server Nodes & Hosted Services

### 🖥️ Proxmox VE
Primary hypervisor managing VMs and LXC containers.

| Service | Category | Description | Status / Dashboard Widget |
| :--- | :--- | :--- | :--- |
| **Homepage** | Infrastructure | Unified dashboard organizing services & live metrics | Central Hub |
| **AdGuard Home 2** | Network & Security | Secondary DNS filter & local ad-blocking | Active |
| **Unbound DNS** | Network & Security | Recursive DNS resolver | Active |
| **Beszel** | Monitoring | Lightweight server stats and resource usage tracker | 2/2 Systems Up |
| **Wiki.js** | Docs & Media | Documentation and knowledge base management | Active |
| **FreshRSS** | Docs & Media | Self-hosted RSS feed aggregator | Active |
| **Actual Budget** | Misc | Local-first personal finance and budgeting | Active |
| **AMP Dashboard** | Misc | Management interface for game server instances | Active |
| **LubeLogger** | Misc | Self-hosted vehicle health, maintenance, and garage tracker | Active |

---

### 💾 TrueNAS
Primary Network Attached Storage (NAS) node providing storage pools and hosting core application stacks.

| Service | Category | Description | Status / Dashboard Widget |
| :--- | :--- | :--- | :--- |
| **AdGuard Home 1** | Network & Security | Primary DNS filter & local ad-blocking | Active |
| **Dockge** | Infrastructure | Reactive GUI for managing Docker Compose stacks | Active |
| **RustDesk** | Remote Access | Self-hosted remote desktop server (managed via Dockge) | Active |
| **Immich** | Docs & Media | High-performance self-hosted photo & video backup solution | Active |
| **Nextcloud** | Docs & Media | Cloud storage, file sharing, and productivity suite | Active |

---

## 🛠 Top-Level Directory Layout

```text
.
├── homepage/              # Homepage config files (settings.yaml, services.yaml, widgets.yaml)
├── docker/                # Compose files for stacks managed by Dockge
│   └── rustdesk/          # RustDesk compose stack configuration
├── adguard/               # AdGuard Home blocklists & DNS rewrite configuration specs
├── unbound/               # Unbound DNS resolver configuration files
└── docs/                  # Network diagrams, hardware specs, and maintenance guides
```

---

## ⚙️ Maintenance & Operations

- **Backups:** Critical data on TrueNAS backed up via ZFS snapshots.
- **Service Management:**
  - Proxmox LXCs managed via `pct` / Proxmox VE API.
  - TrueNAS / Dockge stacks managed via Docker Compose.