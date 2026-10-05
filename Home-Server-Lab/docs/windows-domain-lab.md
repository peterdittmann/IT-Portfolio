# Windows Domain Lab

## Objective

Expand my Proxmox home server into a small Windows domain
environment for practising Windows administration and common
IT support tasks.

The environment will include a Windows Server domain controller
and a Windows client so that I can practise Active Directory,
DNS, user and group administration, Group Policy, permissions,
domain joins, and troubleshooting.

## Environment

### Physical Host
- Hypervisor: Proxmox VE 9.2
- CPU: Intel Core i5-4570
- CPU configuration: 4 cores / 4 threads
- Memory: 16 GB DDR3-1600
- Storage: 500 GB HDD

### Existing Network
- LAN: 192.168.1.0/24
- Proxmox management: 192.168.1.10/24
- Gateway: 192.168.1.254
- Virtual bridge: vmbr0
- AdGuard Home: 192.168.1.11/24

## Phase 1 — Windows Server Deployment

### Objective

Deploy a Windows Server virtual machine on Proxmox and establish
a known-good baseline before installing Active Directory Domain
Services.

### Starting State

Proxmox is operational and existing Linux containers have
validated the host's storage, networking, and vmbr0 bridge.

### VM Configuration

| Setting | Configuration | Reason |
|---|---|---|
| Operating system | TBD | |
| VM ID | TBD | |
| Hostname | TBD | |
| CPU | TBD | |
| Memory | TBD | |
| Disk | TBD | |
| Network bridge | vmbr0 | Connect VM to LAN |
| Network adapter | TBD | |

### Deployment

Document the significant configuration decisions and installation
steps here.

### Validation

Record the tests performed after installation and their results.

### Problems Encountered

Record symptoms, evidence, investigation, root cause and resolution.
Do not remove problems simply because they were resolved.

### Result

To be completed after validation.

### Lessons Learned

To be completed after deployment.
