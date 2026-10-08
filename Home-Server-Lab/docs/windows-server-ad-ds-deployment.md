# Windows Server 2022 Active Directory Domain Services Deployment

**Project:** Home Server Lab  
**Environment:** Independent homelab  
**Platform:** Proxmox VE 9.2  
**Status:** AD DS and DNS deployed and validated

## What I set out to do

Expand the existing Proxmox home server into a Microsoft Windows domain environment for practising entry-level IT support and Windows administration. This phase covered Windows Server 2022 deployment, baseline networking, Active Directory Domain Services, DNS, and validation of the new forest.

This is an independently built homelab, not a production or professional business environment.

## Environment

### Physical Host

- Lenovo workstation
- Intel Core i5-4570
- 16 GB DDR3-1600
- 500 GB HDD
- Proxmox VE 9.2

### Network

| Component | Configuration |
|---|---|
| LAN | `192.168.1.0/24` |
| Gateway | `192.168.1.254` |
| Proxmox | `192.168.1.10` |
| AdGuard Home | `192.168.1.11` |
| Bridge | `vmbr0` |

### Windows Server VM

| Setting | Configuration |
|---|---|
| VM ID | `102` |
| Proxmox VM name | `dc01` |
| Windows hostname | `DC-01` |
| OS | Windows Server 2022 Standard Evaluation (Desktop Experience) |
| CPU | 2 vCPU |
| Memory | 4 GB |
| Disk | 64 GB |
| Machine | q35 |
| Firmware | OVMF / UEFI |
| Storage controller | VirtIO SCSI Single |
| Network | VirtIO via `vmbr0` |
| TPM | 2.0 |
| IPv4 | `192.168.1.12/24` |
| Gateway | `192.168.1.254` |

## Windows Server Deployment

VM 102 was created in Proxmox using q35, UEFI firmware, VirtIO storage and a VirtIO network adapter.

### Storage Driver Troubleshooting

Windows Setup initially could not detect the virtual disk. The VirtIO driver ISO was mounted and the compatible Windows Server 2022 storage driver was loaded. Windows Setup then detected the 64 GB virtual disk and installation continued.

**Lesson:** paravirtualized hardware can require guest-specific drivers before Windows can use the device.

### Network Driver Troubleshooting

After installation, Device Manager initially showed an unrecognised Ethernet controller and the server had no working network connection. The appropriate VirtIO network driver was installed from the driver media.

Device Manager then identified the adapter as:

`Red Hat VirtIO Ethernet Adapter`

Connectivity was restored without changing the virtual NIC model.

## Network Baseline

Before configuring Active Directory, the server was tested for LAN and DNS connectivity.

```powershell
ipconfig /all
ping 192.168.1.254
ping 192.168.1.11
nslookup google.com
```

The server successfully reached the gateway and AdGuard Home and resolved external DNS names.

### Static IPv4

Before promotion to a domain controller, the server was moved from DHCP to a static IPv4 address.

```text
Hostname:        DC-01
IPv4 Address:    192.168.1.12
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.254
```

Connectivity and name resolution were re-tested successfully after the change.

## Pre-AD DS Snapshot

A Proxmox snapshot named `pre-ad-ds-baseline` was created before installing AD DS and DNS.

It represents a known-good standalone Windows Server state and provides a rollback point if the lab needs to be rebuilt.

> The snapshot is a rollback mechanism, not a substitute for a separate backup.

## Active Directory Domain Services

`DC-01` was promoted as the first domain controller in a new forest.

| Setting | Configuration |
|---|---|
| Forest root domain | `ad.seasonandsavour.com` |
| NetBIOS domain | `AD` |
| Domain controller | `DC-01` |
| DNS Server | Enabled |
| Global Catalog | Enabled |
| Forest functional level | Windows Server 2016 |
| Domain functional level | Windows Server 2016 |
| DNS delegation | Not created |

The domain uses a subdomain of a domain under my control.

The prerequisite checks passed before installation. A DNS delegation warning was present because a delegation was not being created in a parent DNS zone for the new forest.

## Post-Promotion Validation

### Authentication and Domain

```powershell
whoami
Get-ADDomain
```

Validation confirmed the administrator session was operating in the `AD` domain and that `ad.seasonandsavour.com` had been created successfully.

In this single-domain-controller lab, `DC-01` holds the domain FSMO roles.

### Forest

```powershell
Get-ADForest
```

Validation confirmed the forest exists, uses Windows Server 2016 forest mode, and contains the expected DNS application partitions. `DC-01` holds the forest FSMO roles.

### DNS

```powershell
Get-Service DNS
Get-DnsClientServerAddress
nslookup ad.seasonandsavour.com
nslookup -type=SRV _ldap._tcp.dc._msdcs.ad.seasonandsavour.com
```

The DNS Server service was running. After promotion, the domain controller was using its local DNS service.

The domain resolved to `DC-01` / `192.168.1.12`, and the LDAP SRV lookup returned `DC-01.ad.seasonandsavour.com`.

This verifies a key part of Active Directory service discovery: clients use DNS SRV records to locate domain controllers and directory services.

Additional domain-controller diagnostics were run and passed.

## Troubleshooting Notes

### Windows Setup Could Not Detect Disk

**Symptom:** No installation disk appeared.

**Cause:** Windows did not yet have the VirtIO SCSI driver.

**Action:** Loaded the Windows Server 2022-compatible VirtIO storage driver.

**Validation:** The 64 GB disk appeared and Windows installation proceeded.

### Ethernet Adapter Not Recognised

**Symptom:** Unknown Ethernet controller and no network connectivity.

**Cause:** Missing VirtIO network driver.

**Action:** Installed the appropriate VirtIO Ethernet driver.

**Validation:** Device Manager displayed `Red Hat VirtIO Ethernet Adapter`, followed by successful LAN and DNS testing.

### Initial SRV Lookup Failed

An initial AD DNS SRV lookup was entered with an incorrect query/domain format and failed. The query was corrected to:

```powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.ad.seasonandsavour.com
```

The corrected query returned the domain controller's LDAP service record.

**Lesson:** verify the diagnostic command itself before treating a failed test as a service failure.

## Evidence

| Screenshot | Evidence |
|---|---|
| `dc01-proxmox-hardware.png` | VM CPU, memory, storage, firmware, VirtIO and TPM configuration |
| `windows-setup-storage-driver-loaded.png` | 64 GB VirtIO-backed disk recognised by Windows Setup |
| `03-virtio-drivers-validated.png` | VirtIO Ethernet adapter recognised |
| `network-verification.png` | Gateway, AdGuard and external DNS connectivity |
| `server-ip-dns-gateway-initial-verification.png` | Static networking before AD promotion |
| `pre-ad-ds-baseline-snapshot.png` | Known-good rollback point |
| `ad-ds-review-options.png` | Forest/domain and DC configuration |
| `Get-ADDomain.png` | Successful domain validation |
| `Get-ADForest.png` | Successful forest and DNS service validation |

## Skills Demonstrated

- Proxmox VM provisioning
- Windows Server 2022 installation
- VirtIO storage and network driver troubleshooting
- Windows Device Manager troubleshooting
- Static IPv4 configuration
- TCP/IP and DNS testing
- Active Directory Domain Services deployment
- Windows DNS Server deployment
- Forest and domain creation
- PowerShell AD validation
- DNS SRV record validation
- Snapshot-based rollback planning
- Technical documentation

## Current State

```text
Proxmox VE Host
│
├── CT 100 — debian-lab
│   └── Debian 13
│
├── CT 101 — adguard
│   ├── Debian 13
│   ├── AdGuard Home
│   └── 192.168.1.11
│
└── VM 102 — dc01
    ├── Windows Server 2022
    ├── Hostname: DC-01
    ├── 192.168.1.12
    ├── Active Directory Domain Services
    ├── DNS Server
    └── ad.seasonandsavour.com
```

The Windows Server portion has progressed from a standalone VM to a functioning single-domain-controller Active Directory lab.

## Next Phase

The next phase shifts from infrastructure deployment toward endpoint and support administration:

1. Verify DNS forwarding and external resolution from the domain controller.
2. Deploy a Windows 11 client VM.
3. Configure the client to use the AD DNS server.
4. Validate domain discovery and join the client to `ad.seasonandsavour.com`.
5. Validate domain authentication and the client computer object.
6. Build a small, realistic OU, user and security-group structure.
7. Configure and test a Group Policy.
8. Add SMB shares and NTFS permissions.
9. Create controlled support incidents involving DNS, accounts, groups, permissions and Group Policy.
10. Document investigation, resolution and validation.

The goal is a small, explainable environment that demonstrates Windows and Active Directory support work relevant to Help Desk, Service Desk and Desktop Support roles.
