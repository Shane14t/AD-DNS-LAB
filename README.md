# Active Directory & DNS Lab (Windows Server 2025)

**Author:** Shane Townsend
**Platform:** VirtualBox • Windows Server 2025 (evaluation) • Windows client VM
**Goal:** Learn how DNS works by building and breaking it inside an Active Directory environment, and document every step with the reasoning behind it.

## Why this lab
DNS is the backbone of Active Directory. Clients find domain controllers through DNS SRV records, and most "AD is broken" tickets are really DNS problems. Understanding DNS makes me faster at troubleshooting in my server support role, and it maps directly to CompTIA Network+ (port 53, record types, name resolution, `nslookup`/`dig`).

## Lab roadmap
| Phase | Topic | Status |
|------|-------|--------|
| 01 | Build the VM and install Windows Server 2025 (VirtualBox, two NICs) | Complete |
| 02 | Base configuration: hostname, static IP, snapshot | Complete |
| 03 | Install AD DS + DNS, promote the first domain controller | Complete |
| 04 | DNS fundamentals: forward and reverse zones, record types (A, AAAA, CNAME, MX, PTR, SRV, NS, SOA) | In progress |
| 05 | Join a client VM; verify SRV records and name resolution | Not started |
| 06 | Forwarders, conditional forwarders, root hints, recursion | Not started |
| 07 | Break-and-fix troubleshooting (`nslookup`, `Resolve-DnsName`, `dcdiag`, Event Viewer) | Not started |
| 08 | Automate with PowerShell (DnsServer module) | Not started |
| 09 | Final write-up and lessons learned | Not started |

## Environment
| Item | Value |
|------|-------|
| Host hypervisor | VirtualBox |
| Server VM | Windows Server 2025 Standard (Desktop Experience), 2 vCPU / 4 GB RAM / 60 GB disk |
| Internal network | `labnet` (VirtualBox Internal Network), 192.168.50.0/24 |
| Domain | `lab.local` (NetBIOS: `LAB`), AD DS + DNS role installed on DC01 |

## Skills demonstrated
Windows Server administration • Active Directory Domain Services • DNS (zones, records, forwarders) • Virtualization • PowerShell • Offline password recovery • Network troubleshooting • Technical documentation

## Documentation index
- [Phase 01: VM build](01-vm-build.md)
- [Phase 04: DNS fundamentals](04-dns-fundamentals.md)
- [Troubleshooting: Locked out of domain Administrator](troubleshooting-01-locked-out-domain-admin.md)
