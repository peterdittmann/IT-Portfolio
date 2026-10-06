# Windows Domain Lab

## Objective

Expand my Proxmox home server into a small Windows domain environment for practising Windows administration and common IT support tasks.

The completed environment will include a Windows Server domain controller and a Windows client so that I can practise:

- Active Directory Domain Services
- DNS
- User and group administration
- Group Policy
- File and NTFS permissions
- Domain joins
- Windows client administration
- Common authentication, networking, and access troubleshooting

The environment is an independent homelab used for learning and technical practice. It is not a production environment.

---

## Environment

### Physical Host

| Component | Configuration |
|---|---|
| Hypervisor | Proxmox VE 9.2 |
| CPU | Intel Core i5-4570 |
| CPU configuration | 4 cores / 4 threads |
| Memory | 16 GB DDR3-1600 |
| Storage | 500 GB HDD |

### Existing Network

| Component | Configuration |
|---|---|
| LAN | `192.168.1.0/24` |
| Proxmox management | `192.168.1.10/24` |
| Gateway | `192.168.1.254` |
| Virtual bridge | `vmbr0` |
| AdGuard Home | `192.168.1.11/24` |

The existing Proxmox host, Linux containers, virtual bridge, and AdGuard Home service provided a known-good infrastructure baseline before the Windows environment was added.

---

# Phase 1 — Windows Server Deployment

## Objective

Deploy a Windows Server 2022 virtual machine on Proxmox and establish a patched, network-connected, known-good standalone server baseline before installing Active Directory Domain Services.

### Success Criteria

Phase 1 would be considered complete when:

- Windows Server 2022 was installed successfully.
- Required VirtIO devices were recognized by Windows.
- The server had working LAN connectivity.
- The server could reach the default gateway.
- The server could reach the existing DNS service.
- External DNS resolution worked.
- Windows Update completed successfully.
- The server was renamed to `DC-01`.
- A static IPv4 configuration was applied and validated.
- The server remained stable before installation of AD DS.

---

## Starting State

Proxmox was already operational, and existing Linux containers had validated the host's storage, networking, and `vmbr0` bridge.

The Windows Server VM was therefore added to an already functioning virtualization and network environment rather than troubleshooting multiple new infrastructure components simultaneously.

---

## VM Configuration

| Setting | Configuration | Reason |
|---|---|---|
| Proxmox node | `pve` | Existing single-node Proxmox host |
| VM ID | `102` | Unique Proxmox VM identifier |
| VM name | `dc01` | Identifies the VM's intended future role |
| Operating system | Windows Server 2022 Standard Evaluation | Provides the Windows Server platform required for the domain lab |
| Interface | Desktop Experience | Provides GUI administration tools relevant to Windows support practice |
| CPU | 1 socket, 2 cores / 2 vCPUs | Preserves host capacity for existing containers and a future Windows client |
| Memory | 4096 MiB / 4 GB | Provides sufficient lab resources while preserving host memory |
| System disk | 64 GiB, SCSI | Provides capacity for Windows Server, updates, and planned lab services |
| Storage backend | `local-lvm` | Uses the existing Proxmox VM storage pool |
| Machine type | `q35` | Provides a modern virtual hardware platform |
| Firmware | OVMF / UEFI | Provides UEFI firmware |
| EFI disk | `local-lvm` | Persists UEFI configuration |
| TPM | TPM 2.0 | Provides a virtual Trusted Platform Module |
| SCSI controller | VirtIO SCSI single | Provides paravirtualized storage |
| Discard | Enabled | Allows unused blocks to be reclaimed by thin-provisioned storage |
| Network bridge | `vmbr0` | Connects the VM to the existing LAN |
| Network adapter | VirtIO | Provides a paravirtualized network interface |
| VLAN | None | The lab currently uses the existing untagged LAN |
| QEMU Guest Agent | Enabled | Provides guest integration capability with Proxmox |
| High availability | Disabled | The homelab uses a single Proxmox node rather than a cluster |

Resources were intentionally limited because the physical host has four CPU cores and 16 GB of RAM and must also support the existing Linux containers and a future Windows client VM.

The VM was created without immediately starting the operating system installation so that the Windows VirtIO driver ISO could be attached first.

![DC01 virtual hardware configuration](../images/windows-domain/dc01-proxmox-hardware.png)

*DC01 virtual hardware configuration in Proxmox, including UEFI/TPM, VirtIO storage and networking, and separate Windows Server and VirtIO driver installation media.*

---

## Windows Server Installation

Windows Server 2022 Standard Evaluation with Desktop Experience was selected.

Standard provides the server functionality required for the planned Active Directory and DNS lab. Desktop Experience was selected because the graphical administration tools are useful for practising common Windows Server support and administration workflows.

---

## Troubleshooting — Windows Setup Could Not Detect the Virtual Disk

### Symptom

During Windows Server installation, Windows Setup did not initially detect the 64 GiB virtual system disk.

### Evidence

The virtual disk was visible in the Proxmox hardware configuration, confirming that it had been created and attached to the VM.

The VM was configured to use the VirtIO SCSI controller.

### Likely Cause

Windows Setup did not have the required VirtIO SCSI storage driver loaded.

### Resolution

The VirtIO driver ISO had already been attached to the VM.

The Windows Server 2022 x64 VirtIO SCSI driver was loaded from:

`vioscsi/2k22/amd64`

### Validation

After loading the driver, Windows Setup successfully detected the 64 GiB virtual system disk and installation was able to continue.

![Windows Setup with detected VirtIO disk](../images/windows-domain/windows-setup-storage-driver-loaded.png)

*Windows Setup detecting the 64 GiB virtual system disk after the Windows Server 2022 VirtIO SCSI driver was loaded.*

### Lesson

A virtual disk being present in the hypervisor does not guarantee that the guest operating system can use it. The guest also requires a compatible driver for the virtual storage controller.

---

## Base Operating System Installation

Windows Server 2022 Standard Evaluation with Desktop Experience was successfully installed on the virtual system disk.

After installation, the local Administrator account was used to access the standalone server.

Before adding Active Directory Domain Services, I validated the base operating system and network configuration to establish a known-good baseline.

---

## Troubleshooting — VirtIO Network Adapter

### Symptom

After Windows Server installation, the virtual Ethernet controller was not initially available as a functioning Windows network adapter.

Device Manager showed an Ethernet controller without the required driver.

### Investigation

The Proxmox VM configuration showed that the VM was connected to `vmbr0` using a VirtIO network adapter.

Because the virtual NIC existed at the hypervisor level but Windows did not have a functioning network interface, the guest driver became the primary troubleshooting focus.

### Resolution

The appropriate VirtIO network driver was installed from the attached VirtIO driver media.

Windows subsequently identified the interface as:

`Red Hat VirtIO Ethernet Adapter`

The remaining VirtIO guest components were installed and Device Manager was checked again for unresolved devices.

### Validation

The warning for the Ethernet controller was cleared in Device Manager, Windows recognized the Red Hat VirtIO Ethernet Adapter, and the remaining previously unidentified virtual devices no longer showed unresolved driver warnings.

Windows then obtained an IPv4 configuration from the existing DHCP server.

This established that the virtual NIC, Windows driver, and Proxmox bridge were functioning together.

### Lesson

When virtual hardware exists in the hypervisor but is unavailable inside the guest operating system, verify the guest driver before troubleshooting higher layers such as IP addressing, routing, or DNS.

---

## Initial DHCP Network Validation

Before assigning the server a static address, I validated networking using the DHCP configuration supplied by the existing network.

The server initially received:

| Setting | Value |
|---|---|
| IPv4 address | `192.168.1.64/24` |
| Default gateway | `192.168.1.254` |
| IPv4 DNS server | `192.168.1.11` |

The configuration was inspected using:

```cmd
ipconfig /all
```

Connectivity was then tested in stages.

### Gateway Connectivity

```cmd
ping 192.168.1.254
```

The default gateway responded successfully with no packet loss.

### DNS Server Connectivity

```cmd
ping 192.168.1.11
```

The existing AdGuard Home DNS server responded successfully with no packet loss.

### DNS Resolution

```cmd
nslookup google.com
```

The lookup returned a valid result.

These tests confirmed that the following components were functioning before additional server configuration:

`Windows NIC -> VirtIO driver -> vmbr0 -> LAN -> gateway/DNS`

This provided a known-good network baseline before changing from DHCP to static addressing.

---

## Initial Server Configuration

After the base installation and network validation, the server was renamed from its automatically generated Windows hostname to:

`DC-01`

The rename was completed before installing Active Directory Domain Services so that the intended server identity was established before domain-controller promotion.

The system time zone was also changed to Mountain Time to match the physical location of the lab.

Windows Update was then run before adding server roles.

After the available updates and required restarts completed, Windows Update reported that the server was up to date.

This established a patched standalone baseline before introducing Active Directory or DNS roles.

---

## Static IPv4 Configuration

A domain controller should not depend on a changing DHCP lease, so a static IPv4 address was configured before installing Active Directory Domain Services.

The existing DHCP configuration was reviewed before selecting the address.

The router's DHCP pool begins at:

`192.168.1.64`

The address selected for `DC-01` was:

`192.168.1.12`

This placed the server outside the DHCP pool and alongside the existing statically addressed infrastructure.

Before assigning the address, the proposed address was tested for an obvious conflict. It did not respond to ICMP and did not appear in the ARP table after the test. Combined with its location outside the DHCP pool, this provided reasonable evidence that the address was available for the lab server.

### Configuration

| Setting | Configuration |
|---|---|
| Hostname | `DC-01` |
| IPv4 address | `192.168.1.12` |
| Subnet mask | `255.255.255.0` (`/24`) |
| Default gateway | `192.168.1.254` |
| DNS server | `192.168.1.11` — AdGuard Home |
| DHCP | Disabled |

> **DNS note:** `192.168.1.11` represents the standalone server's current pre-AD DNS configuration. DNS will be redesigned as part of the Active Directory deployment because domain members must be able to resolve the DNS records published by Active Directory.

---

## Static Network Validation

After changing the interface from DHCP to static addressing, the network configuration was checked again.

```cmd
ipconfig /all
```

This confirmed:

- Hostname `DC-01`
- IPv4 address `192.168.1.12`
- Subnet mask `255.255.255.0`
- Default gateway `192.168.1.254`
- DNS server `192.168.1.11`
- DHCP disabled

Connectivity was then retested.

```cmd
ping 192.168.1.254
ping 192.168.1.11
nslookup google.com
```

The default gateway remained reachable, the existing DNS service remained reachable, and external DNS resolution continued to work.

This demonstrated that the server retained network functionality after moving from DHCP to its permanent lab address.

---

## Phase 1 Validation

| Test | Result |
|---|---|
| Windows Server 2022 installation | PASS |
| VirtIO SCSI storage driver | PASS |
| 64 GiB system disk detected | PASS |
| VirtIO network driver | PASS |
| Network adapter recognized | PASS |
| Unresolved virtual-device driver warnings cleared | PASS |
| DHCP baseline connectivity | PASS |
| Gateway connectivity | PASS |
| AdGuard connectivity | PASS |
| External DNS resolution | PASS |
| Hostname changed to `DC-01` | PASS |
| Time zone configured | PASS |
| Windows Update completed | PASS |
| Static IPv4 `192.168.1.12/24` | PASS |
| DHCP disabled | PASS |
| Connectivity after static configuration | PASS |

---

## Phase 1 Result

Phase 1 is complete.

A Windows Server 2022 Standard Evaluation VM is operational on the Proxmox host as `DC-01`.

The server has:

- working VirtIO storage and networking
- a patched Windows Server installation
- a static IPv4 address of `192.168.1.12/24`
- connectivity to the local gateway
- connectivity to the existing AdGuard Home DNS service
- working external DNS resolution

The server remains a standalone system. Active Directory Domain Services and Windows DNS have not yet been installed.

This provides a known-good baseline from which the Windows domain environment can now be built.

---

## Lessons Learned

### Validate from the Bottom Up

The deployment reinforced the value of validating infrastructure in layers.

For example, network troubleshooting was performed in the following order:

`virtual NIC -> guest driver -> IP configuration -> gateway -> DNS server -> hostname resolution`

This made it easier to distinguish a missing VirtIO driver from an IP or DNS problem.

### Hypervisor Configuration and Guest Configuration Are Separate

Proxmox can present a device correctly while Windows is still unable to use it.

Both the storage and network configuration demonstrated that guest operating system drivers must be considered separately from the hypervisor configuration.

### Establish a Known-Good Baseline Before Adding Roles

Windows Server was installed, patched, renamed, assigned a static address, and tested before Active Directory was introduced.

This reduces the number of variables involved if problems occur during the next phase.

### Re-Test After Configuration Changes

Network connectivity was tested while the server was using DHCP and then tested again after static addressing was configured.

A configuration change should not be assumed successful simply because the settings were accepted by the operating system.

---

# Phase 2 — Active Directory and DNS

## Objective

Promote the validated standalone Windows Server into the first domain controller
for the lab and establish working Active Directory-integrated DNS.

## Domain Design

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

...
---

## Current Status

**Completed:** Windows Server 2022 VM deployment and standalone baseline.

**Current milestone:** Prepare `DC-01` for Active Directory Domain Services and DNS.

**Next milestone:** Deploy and validate the Active Directory domain.

**Following milestone:** Deploy a Windows client VM and join it to the domain.
