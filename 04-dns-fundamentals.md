# Phase 04: DNS Fundamentals — Zones, Records, and Reverse Lookup

## Objective
Understand and build out the core DNS components on DC01: forward vs. reverse zones, and the record types that make AD and ordinary name resolution work.

## Environment confirmed
Verified via PowerShell after Phase 03:
- Domain: `lab.local` (NetBIOS: `LAB`)
- DNS Server role: installed and running
- `_msdcs.lab.local`: AD-integrated zone holding SRV records (proves AD registered itself with DNS)
- `lab.local`: primary forward lookup zone, AD-integrated
- `0.in-addr.arpa`, `127.in-addr.arpa`, `255.in-addr.arpa`: Windows default reverse-zone placeholders (not a real reverse zone for our subnet)

> **Note on domain naming:** `lab.local` uses the `.local` TLD, which is reserved for multicast DNS (mDNS/Bonjour, used by Apple devices and some IoT gear) and has been officially discouraged by Microsoft since 2012 for exactly that reason — it can cause name resolution conflicts on networks with mDNS traffic. In an isolated lab network this causes no issues, but in production, `.internal` or a subdomain of a domain you actually own is the better choice. Documenting this tradeoff here rather than rebuilding the domain.

## Why it matters
DNS record types each solve a different resolution problem: a hostname to an IP, an IP back to a hostname, where to find a domain controller, where to deliver mail. Active Directory specifically depends on SRV records so clients can locate DCs, global catalog servers, and the PDC emulator by *service*, not by a hardcoded IP.

## Part A: Confirm what AD already built

```powershell
Get-ADDomain | Select-Object DNSRoot, NetBIOSName, DomainMode
hostname
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.lab.local
```

The SRV lookup should return a record pointing at `dc01.lab.local`. This is the mechanism every domain-joined client uses to find a domain controller — it's not a hardcoded IP anywhere, it's a DNS query.

## Part B: Create a reverse lookup zone

Reverse DNS answers "whose IP is this?" instead of "what's this name's IP?" — used for logging, some authentication protocols, and troubleshooting.

```powershell
Add-DnsServerPrimaryZone -NetworkID "192.168.50.0/24" -ReplicationScope "Domain"
```

> **Why `-ReplicationScope Domain`:** keeps the zone AD-integrated and replicated to any additional DCs I add later, consistent with how `lab.local` itself is configured, rather than storing it as a flat file only DC01 can see.

Verify:
```powershell
Get-DnsServerZone | Where-Object {$_.IsReverseLookupZone -eq $true}
```

## Part C: Record types — create one of each

| Record | Purpose | Command |
|---|---|---|
| A | Hostname → IPv4 | `Add-DnsServerResourceRecordA -ZoneName "lab.local" -Name "test-a" -IPv4Address "192.168.50.50"` |
| PTR | IPv4 → hostname (reverse) | `Add-DnsServerResourceRecordPtr -ZoneName "50.168.192.in-addr.arpa" -Name "50" -PtrDomainName "test-a.lab.local."` |
| CNAME | Alias to another name | `Add-DnsServerResourceRecordCName -ZoneName "lab.local" -Name "www-test" -HostNameAlias "test-a.lab.local."` |
| MX | Mail server for the domain | `Add-DnsServerResourceRecordMX -ZoneName "lab.local" -Name "." -MailExchange "test-a.lab.local." -Preference 10` |

> **On SOA and NS:** these already exist automatically on every zone — SOA (Start of Authority) holds the zone's serial number, refresh/retry/expire timers, and the authoritative server; NS lists the name servers authoritative for the zone. Worth opening DNS Manager and looking at them rather than creating new ones.

Verify each record resolves:
```powershell
Resolve-DnsName test-a.lab.local
Resolve-DnsName -Type PTR 192.168.50.50
Resolve-DnsName www-test.lab.local
Resolve-DnsName -Type MX lab.local
```

## Verification
- [ ] SRV lookup for `_ldap._tcp.dc._msdcs.lab.local` resolves to DC01
- [ ] Reverse lookup zone for 192.168.50.0/24 exists
- [ ] A, PTR, CNAME, and MX test records created and resolve correctly

## Evidence (screenshots)
- `01-get-dnsserverzone.png`: zone list (already captured)
- `02-reverse-zone-created.png`
- `03-records-created.png`
- `04-resolve-dnsname-results.png`

## What I learned
_(Fill in after completing the phase.)_

## Next
[Phase 05: Join a client VM](05-client-join.md)
