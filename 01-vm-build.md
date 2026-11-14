# Phase 01: Build the VM and install Windows Server 2025

## Objective
Create an isolated lab environment in VirtualBox and install Windows Server 2025, the future domain controller and DNS server.

## Why it matters
Isolating the lab keeps a DC that hands out DNS from interfering with my home network. It also lets me snapshot and roll back, which makes breaking things safe.

## Design decisions
| Decision | Choice | Reasoning |
|----------|--------|-----------|
| Edition | Server 2025 Standard, Desktop Experience | GUI tools (DNS Manager, ADUC) are easier for learning; I'll repeat tasks in PowerShell later |
| Licensing | 180-day Evaluation ISO from the Microsoft Evaluation Center | Free, fully functional for a lab |
| Resources | 2 vCPU, 4 GB RAM, 60 GB dynamic disk | Enough for AD DS + DNS and leaves room for a client VM |
| NIC 1 | NAT | Gives the server internet access for updates |
| NIC 2 | Internal Network `labnet` | Isolated segment where the lab domain and clients live |
| Domain name | `lab.internal` | `.internal` is reserved for private use. Avoid `.local`, which conflicts with mDNS |

## Steps

### 1. Download the ISO
1. Go to the Microsoft Evaluation Center and download **Windows Server 2025** (ISO, 64-bit, English).
2. Save it somewhere easy to find, e.g. `C:\ISOs\`.

### 2. Create the VM in VirtualBox
1. **New** → Name: `DC01`, Type: Microsoft Windows, Version: Windows 2022 (64-bit) if 2025 isn't listed.
2. Select the ISO, and **check "Skip Unattended Installation"** so I see and learn every install screen.
3. Hardware: 4096 MB RAM, 2 CPUs.
4. Virtual hard disk: 60 GB, dynamically allocated.

### 3. Configure networking (before first boot)
1. VM **Settings → Network → Adapter 1**: Attached to **NAT**.
2. **Adapter 2**: Enable, Attached to **Internal Network**, Name: `labnet`.

> **Why two adapters?** NAT reaches the internet but gives no fixed, controllable addressing. The internal network is where I assign static IPs and where the DC's DNS will serve clients.

### 4. Install Windows Server 2025
1. Start the VM and boot from the ISO.
2. Choose language, **Install now**, and select **Standard (Desktop Experience)**.
3. Select **Custom: Install Windows only** and use the unallocated disk.
4. Set the local Administrator password (record it in a password manager, not in these notes).
5. Sign in and install **VirtualBox Guest Additions** (Devices → Insert Guest Additions CD). Reboot.

### 5. Take a snapshot
VirtualBox → Snapshots → Take: `01-Fresh-Install`.

> **Why:** A clean restore point. If a later phase goes wrong, I roll back instead of reinstalling.

## Verification
- [ ] VM boots to the desktop
- [ ] Adapter 1 shows internet access
- [ ] Adapter 2 shows as a second network adapter (no IP yet)
- [ ] Snapshot `01-Fresh-Install` exists

## Evidence (screenshots)
Save in `screenshots/` and link them here:
- `01-vm-settings.png`: VM hardware and network settings
- `02-install-edition.png`: edition selection screen
- `03-desktop.png`: first sign-in
- `04-snapshot.png`: snapshot list

## What I learned
_(Fill in after completing the phase: 2-3 sentences in your own words.)_

## Next
[Phase 02: Base configuration](02-base-config.md): set the hostname and static IP.
