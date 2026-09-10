# Lenovo Home Server Lab

I was given an older Lenovo desktop and wanted to determine whether it was worth repurposing as a home server and virtualization lab rather than leaving it unused.

The machine initially powered on but produced no display. After troubleshooting that issue, I inventoried and validated the hardware, upgraded the memory, tested the existing hard drive, enabled hardware virtualization, and installed Proxmox VE as a bare-metal hypervisor.

The system now runs headless on my home network and is remotely administered through Proxmox. I have deployed an initial Debian LXC container and am using the server to build practical experience with virtualization, Linux, networking, storage, DNS, and infrastructure troubleshooting.

This repository documents the build, including the problems encountered, troubleshooting process, configuration decisions, and validation performed along the way.

## Starting Hardware

| Component | Configuration |
| --- | --- |
| System | Lenovo 10A8S00200 |
| CPU | Intel Core i5-4570 @ 3.20 GHz |
| CPU configuration | 4 cores / 4 logical processors |
| Memory | 8 GB DDR3-1600 |
| Memory configuration | 2 x 4 GB Hynix |
| Storage | 500 GB Seagate HDD |
| Drive model | ST500DM002-1SB10A |
| GPU | NVIDIA T1000 |
| Firmware | UEFI |
| Operating system | Windows 10 Pro, Build 19045 |

## Initial Troubleshooting

### No Display Output

When I first powered on the machine, the fans started, the power LED remained on, and the keyboard received power, but the monitor reported `No Signal`.

The monitor was connected by DisplayPort to the motherboard.

Since the machine appeared to be powering up normally, I inspected the system for another possible display output before assuming there was a larger hardware problem.

Opening the chassis revealed an NVIDIA T1000 discrete graphics card. The card uses Mini DisplayPort outputs, so I connected the monitor directly to the T1000 using a Mini DisplayPort-to-DisplayPort cable.

Video output returned immediately and the system booted normally.

**Cause:** The display was connected to the motherboard video output rather than the installed discrete GPU.

**Resolution:** Connected the monitor directly to the NVIDIA T1000.

![NVIDIA T1000 and motherboard](images/lenovo-motherboard.jpg)

This was a useful reminder not to jump from "no display" directly to failed RAM, motherboard, CPU, or power supply. The other signs of life suggested the machine might already be completing POST, so checking the available display hardware was a low-risk place to start.

## System Inventory

Once I had access to Windows, I wanted to establish exactly what hardware I was working with before buying anything.

I started with Windows System Information (`msinfo32`).

![Windows System Information](images/system-information.png)

This confirmed:

- Intel Core i5-4570
- 8 GB installed RAM
- UEFI firmware
- Windows 10 Pro
- Lenovo 10A8 platform

The CPU reports the required virtualization extensions, but System Information initially showed:

```text
Virtualization Enabled in Firmware: No
```

This identified hardware virtualization as a configuration issue that needed to be resolved before deploying the system as a virtualization host. I addressed this during the hardware-validation stage documented below.

## Memory Investigation

I wanted the exact specifications of the existing memory before deciding what to purchase.

I queried the installed DIMMs with PowerShell:

```powershell
Get-CimInstance Win32_PhysicalMemory |
    Select-Object DeviceLocator, Manufacturer, PartNumber, Capacity, Speed
```

The system returned two Hynix modules:

```text
Manufacturer : Hynix/Hyundai
PartNumber   : HMT451U6AFR8C-PB
Capacity     : 4294967296
Speed        : 1600
```

Each module is 4 GB, giving the system its starting 8 GB total.

![PowerShell memory and disk inventory](images/powershell-hardware-inventory.png)

Physical inspection showed four DIMM slots, with two populated.

![Internal system overview](images/lenovo-internal-overview.jpg)

Based on this assessment, I decided to add another 2 x 4 GB DDR3-1600 kit rather than replacing the existing memory.

The target configuration was:

```text
Starting:  8 GB
Target:   16 GB
```

Sixteen gigabytes would provide considerably more room for lightweight virtual machines and containers while keeping the amount invested in this older platform low.

## Storage Investigation

Disk Management showed one physical disk with approximately 465 GB of usable capacity.

![Windows Disk Management](images/disk-management.png)

The disk contained the existing Windows installation with:

- 100 MB EFI System Partition
- approximately 465 GB NTFS Windows partition
- 548 MB Recovery partition

I then queried the physical disk from PowerShell:

```powershell
Get-CimInstance Win32_DiskDrive |
    Select-Object Model, Size, Status
```

The installed drive was identified as:

```text
Model  : ST500DM002-1SB10A
Size   : 500105249280
Status : OK
```

This is a 500 GB Seagate mechanical hard drive.

The drive reported an OK status through Windows, but because it is an older HDD I did not want to rely on that result alone before using it as the storage device for the Proxmox installation.

## Physical Inspection

After shutting the machine down and disconnecting power, I opened the chassis to see what expansion options were available.

![Internal system overview](images/lenovo-internal-overview.jpg)

I was specifically looking for:

- available memory slots
- storage mounting locations
- SATA connections
- PCIe expansion
- existing cabling
- general condition of the machine

The system has four DIMM slots, with two originally occupied, which made the planned memory upgrade straightforward.

The NVIDIA T1000 occupies the primary PCIe slot, with additional expansion available below it.

### Drive Cage

The existing Seagate HDD is mounted in Lenovo's drive assembly.

![Lenovo drive cage](images/lenovo-drive-cage.jpg)

There is enough flexibility in the chassis to continue investigating an SSD and additional storage, but I decided not to buy storage immediately.

I looked at several SATA SSD options during the assessment, but the prices I found did not make sense relative to the age and value of the machine. I decided to upgrade the inexpensive DDR3 memory, continue validating the existing hardware, and revisit SSD storage later.

## Upgrade Decisions

My approach with this machine was to avoid spending money simply because something *could* be upgraded.

Based on the initial assessment, I chose:

```text
Memory:   8 GB -> 16 GB DDR3-1600
Storage:  Retain existing 500 GB HDD
SSD:      Deferred
Backups:  Future dedicated storage
```

### Storage Upgrade Decision

I initially planned to replace the existing 500 GB HDD with a SATA SSD before deploying the server.

While researching 500 GB and 1 TB SATA drives, I found that current SSD pricing was considerably higher than I expected. Rather than spending heavily on storage for an older platform, I decided to keep the existing HDD for the initial build.

For the first stage of the project, the HDD is sufficient for installing Proxmox, learning the platform, and running lightweight services. VM and container storage performance will be slower than it would be on an SSD, but this does not prevent me from building and testing the environment.

The SSD upgrade has therefore been deferred rather than cancelled. I will revisit it when I find a reasonably priced SATA SSD.

## Hardware Upgrade and Validation

Before replacing Windows with Proxmox, I completed the planned memory upgrade and tested the existing hardware to make sure the system was stable enough to use as a virtualization host.

### Memory Upgrade

I installed an additional 2 x 4 GB Gigastone DDR3-1600 kit alongside the existing Hynix memory.

The resulting configuration is:

```text
Existing:  2 x 4 GB Hynix DDR3-1600
Added:     2 x 4 GB Gigastone DDR3-1600
Total:     16 GB DDR3-1600
```

After installation, I checked the system firmware and Windows to confirm that all 16 GB was detected and operating at 1600 MHz.

![16 GB memory detected in UEFI](images/Uefi-16gb-memory.jpg)

Because the final configuration uses DIMMs from two manufacturers, detection alone was not enough to consider the upgrade successful.

### Memory Testing

I ran Windows Memory Diagnostic using two passes to check the upgraded memory configuration for errors.

After the test completed, I checked Event Viewer for the diagnostic result.

Event ID 1201 reported:

```text
The Windows Memory Diagnostic tested the computer's memory and detected no errors.
```

![Windows Memory Diagnostic passed](images/Windows-memory-diagnostic-pass.png)

With all 16 GB detected and the memory diagnostic completing without errors, I decided to keep the mixed Hynix and Gigastone configuration.

### Virtualization Readiness

The initial Windows assessment showed that the processor supported virtualization, but virtualization was disabled in firmware.

I entered UEFI and enabled:

- Intel Virtualization Technology (VT-x)
- Intel VT-d

After booting back into Windows, I returned to System Information to verify the change.

The system reported:

```text
Hyper-V - VM Monitor Mode Extensions: Yes
Hyper-V - Second Level Address Translation Extensions: Yes
Hyper-V - Virtualization Enabled in Firmware: Yes
Hyper-V - Data Execution Prevention: Yes
```

![Virtualization enabled in firmware](images/Virtualization-enabled.png)

This confirmed that the system was ready to run a bare-metal hypervisor.

### HDD Health Validation

Because I decided to retain the existing 500 GB mechanical hard drive, I wanted more information about its condition than the basic `Status : OK` result returned by Windows.

I installed `smartmontools` and inspected the SMART data for the drive.

Important attributes included:

```text
Reallocated sectors:       0
Reported uncorrectable:    0
Current pending sectors:   0
Offline uncorrectable:     0
UDMA CRC errors:           0
Power-on hours:            ~10,272
Temperature:               34 C
```

The SMART error log did not contain recorded disk errors.

I then ran an extended SMART self-test against the drive. The completed test reported:

```text
Completed without error
```

No first-error LBA was reported.

![HDD extended SMART test](images/Hhd-smart-extended-test.png)

These results were good enough for me to proceed with the HDD for the initial lab deployment.

The drive is still older mechanical storage, so I do not intend to treat it as the only copy of important data. A future SSD upgrade and proper backup strategy remain part of the project.

## Proxmox VE Deployment

With the memory, virtualization support, and HDD validated, I replaced the Windows installation with Proxmox VE.

Proxmox was installed directly on the Lenovo as a bare-metal hypervisor using the existing 500 GB Seagate HDD.

The initial management configuration is:

```text
Hostname:         pve
Management IP:    192.168.1.10/24
Default gateway:  192.168.1.254
Web interface:    https://192.168.1.10:8006
```

The management address is configured statically so the location of the hypervisor does not depend on a changing DHCP lease.

The Proxmox web interface uses a self-signed certificate by default, so browsers currently display a certificate warning when I access the management interface over the LAN. This is expected for the current lab configuration.

![Initial Proxmox VE deployment](images/Proxmox-Summary-Initial.png)

## Repository Configuration and Updates

After installation, I attempted to update the Proxmox host.

The update initially returned:

```text
401 Unauthorized
```

The failing source was the Proxmox enterprise repository.

![Proxmox enterprise repository 401 error](images/Proxmox-enterprise-repo-401.png)

The enterprise repository requires a paid subscription, which this lab does not use. I enabled the `pve-no-subscription` repository instead.

A subsequent update still produced an authorization error because the enterprise Ceph repository remained enabled.

I disabled the enterprise Ceph repository as well and ran the update again.

The host was then able to retrieve updates successfully from the Debian, Debian security, and Proxmox no-subscription repositories.

The update task completed successfully with:

```text
TASK OK
```

![Proxmox repositories corrected](images/Proxmox-repositories-fixed.png)

This was my first configuration issue after installing Proxmox and provided a useful example of separating a repository authentication problem from a general network or package-manager failure.

### Kernel Update Verification

After completing the Proxmox updates, the system reported that a new kernel had been installed and recommended rebooting the node.

I rebooted the host and checked the active kernel:

```bash
uname -r
```

The system returned:

```text
7.0.14-16-pve
```

This confirmed that the host had successfully booted using the updated Proxmox kernel.

## Proxmox Networking

I inspected the host networking with:

```bash
ip addr
```

and:

```bash
ip route
```

The relevant configuration was:

```text
Management bridge:  vmbr0
Host address:       192.168.1.10/24
Default gateway:    192.168.1.254
LAN subnet:         192.168.1.0/24
```

The physical Ethernet interface is attached to the Linux bridge `vmbr0`.

The host's routing table showed:

```text
default via 192.168.1.254 dev vmbr0
192.168.1.0/24 dev vmbr0 proto kernel scope link src 192.168.1.10
```

`vmbr0` allows virtual machines and containers to connect through the physical network interface while appearing as separate systems on the LAN.

This also gave me a clearer practical understanding of the difference between the physical network interface and the virtual bridge used by the hypervisor.

![Proxmox IP address and routing configuration](images/Iproute-ipaddr.png)

## Proxmox Storage Layout

After installation, I initially noticed that:

```bash
df -h
```

showed approximately 94 GB for the root filesystem.

Since the machine contains a 500 GB disk, I wanted to determine where the remaining capacity had been allocated.

I inspected the block devices using:

```bash
lsblk
```

The Proxmox installer had created an LVM-based storage layout consisting approximately of:

```text
500 GB Seagate HDD
|
+-- EFI partition
|
+-- pve-swap       8 GB
|
+-- pve-root      96 GB
|
+-- pve-data     ~337 GB LVM-thin pool
```

The missing capacity was therefore not actually missing.

`df -h` reports mounted filesystems, while the large `pve-data` LVM-thin pool is used for VM and LXC disks and is not displayed as a conventional mounted filesystem.

![Proxmox storage layout](images/Proxmox-storage-lsblk.png)

This was a useful example of why storage troubleshooting sometimes requires looking beyond filesystem usage and examining the underlying block-device and volume layout.

Within Proxmox, this storage is presented primarily as:

- `local` for directory-based host storage such as templates and backups
- `local-lvm` for VM and LXC virtual disks

## First LXC Container

With the Proxmox host updated and networking verified, I created my first Linux container.

I wanted the first container to remain a general-purpose Debian environment rather than immediately turning it into a household service. This gives me a lightweight system that I can experiment with, troubleshoot, and rebuild without affecting other devices.

I downloaded the Debian 13 standard LXC template:

```text
debian-13-standard_13.6-1_amd64.tar.zst
```

I then created:

| Setting | Configuration |
| --- | --- |
| CT ID | 100 |
| Hostname | `debian-lab` |
| Operating system | Debian 13 |
| Container type | Unprivileged LXC |
| CPU | 1 core |
| Memory | 512 MiB |
| Swap | 512 MiB |
| Root disk | 8 GiB |
| Storage | `local-lvm` |
| Network bridge | `vmbr0` |
| IPv4 | DHCP |

The resource allocation was intentionally small because the container is being used as a basic Linux lab rather than a resource-intensive workload.

## LXC Network Validation

After starting the container, I logged in through the Proxmox console and inspected its network configuration.

```bash
ip addr
```

The container received an IPv4 address from DHCP:

```text
192.168.1.77/24
```

I then checked its routing table:

```bash
ip route
```

The result included:

```text
default via 192.168.1.254 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.77
```

This confirmed that the container had received both a valid LAN address and the correct default gateway.

Rather than treating a single successful ping as proof that networking was working, I tested connectivity in stages.

### Local Gateway

First, I tested the router:

```bash
ping -c 4 192.168.1.254
```

A successful response confirmed local Layer 3 connectivity between the container and the router.

### Internet Routing

Next, I tested an external IP address:

```bash
ping -c 4 1.1.1.1
```

This also succeeded.

Because the test used an IP address rather than a hostname, it demonstrated that the container could route traffic beyond the local network without relying on DNS.

### DNS Resolution

Finally, I tested a hostname:

```bash
ping -c 4 google.com
```

This succeeded as well.

The troubleshooting sequence therefore validated:

```text
Container
    |
    v
Local network
    |
    v
Default gateway
    |
    v
Internet routing
    |
    v
DNS resolution
```

If the gateway and `1.1.1.1` had responded while the hostname failed, DNS would have become the primary troubleshooting target.

## LXC Updates and Resource Usage

Once basic networking was confirmed, I updated the Debian container:

```bash
apt update
apt upgrade
```

The upgrade completed successfully.

I then checked its memory and disk usage:

```bash
free -h
df -h
```

At idle, the container was using approximately:

```text
Memory:  34 MiB / 512 MiB
Disk:    713 MiB / 8 GiB
```

The low resource usage demonstrates one of the reasons I wanted to experiment with LXC containers. A basic Linux environment can provide an isolated workload while consuming substantially fewer resources than a full virtual machine.

This is especially useful on an older system with 16 GB of total memory.

## Headless Deployment

After validating the Proxmox host and first LXC container, I shut down the system and moved it to its intended location near the home network equipment.

The Lenovo now runs without a dedicated monitor, keyboard, or mouse and connects to the home network through wired Ethernet.

![Lenovo Proxmox server deployed headless](images/Proxmox-headless-deployment.jpg)

After powering the system back on, I returned to another workstation on the LAN and connected to:

```text
https://192.168.1.10:8006
```

The Proxmox node was online and accessible without requiring any locally attached input or display devices.

![Remote Proxmox administration](images/Proxmox-remote-management.jpg)

This completed the initial objective of converting the unused Lenovo desktop into a remotely managed virtualization host.

The management path is now:

```text
Remote workstation
        |
        | LAN
        v
Home router
        |
        | Ethernet
        v
Lenovo Proxmox host
        |
        +---- vmbr0
                 |
                 +---- LXC / VM workloads
```

## Current Environment

The server has progressed from the original Windows 10 desktop configuration to the following environment:

| Component | Current Configuration |
| --- | --- |
| Host | Lenovo 10A8S00200 |
| CPU | Intel Core i5-4570, 4 cores / 4 threads |
| Memory | 16 GB DDR3-1600 |
| Storage | 500 GB Seagate HDD |
| Hypervisor | Proxmox VE 9.2 |
| Kernel | `7.0.14-16-pve` |
| Management IP | `192.168.1.10/24` |
| Default gateway | `192.168.1.254` |
| Virtual bridge | `vmbr0` |
| First container | Debian 13 LXC |
| Container resources | 1 vCPU / 512 MiB RAM / 8 GiB disk |
| Container networking | DHCP via `vmbr0` |
| Administration | Headless via Proxmox web interface |

## Intended Use

With Proxmox VE now deployed, I intend to expand the system gradually as both a home server and an IT lab.

The next workloads I am considering include:

- AdGuard Home for local DNS filtering
- additional Linux containers and VMs
- a Windows Server lab
- SMB file sharing
- PC/server backups
- basic monitoring

I want to keep experimental lab systems separate from services that other devices in the house depend on.

For example, a Windows Server DNS or Active Directory lab should be something I can break and rebuild without taking down normal household DNS.

The exact design will continue to be documented as I deploy and test each service rather than designing the entire environment in advance.

## Next Steps

- Deploy a dedicated AdGuard Home LXC container
- Assign or reserve a stable LAN address for DNS services
- Configure and validate DNS forwarding and filtering
- Develop a basic Proxmox backup strategy
- Test LXC backup and restore
- Create a Windows Server VM for Active Directory experimentation
- Keep Active Directory/DNS testing isolated from household DNS services
- Add host and service monitoring
- Evaluate a future SATA SSD upgrade
- Document recovery procedures for important services

## Project Log

### September 7, 2026 — Initial Assessment

Received the Lenovo system and performed the initial hardware assessment.

The machine initially appeared to have a no-display problem. I determined that the monitor was connected to the motherboard DisplayPort output while an NVIDIA T1000 was installed. Connecting the display to the T1000 restored video.

Once in Windows, I:

- checked the system configuration with `msinfo32`
- inspected the existing disk with Disk Management
- queried installed memory using `Get-CimInstance Win32_PhysicalMemory`
- queried storage using `Get-CimInstance Win32_DiskDrive`
- identified two 4 GB Hynix DDR3-1600 DIMMs
- identified the existing 500 GB Seagate HDD
- confirmed virtualization support was present but disabled in firmware
- opened the chassis and inspected the memory, storage, PCIe, and drive layout
- decided to begin with a low-cost RAM upgrade rather than immediately investing heavily in storage

### September 8, 2026 — Hardware Validation

- upgraded memory from 8 GB to 16 GB DDR3-1600
- confirmed all memory was detected at 1600 MHz
- completed Windows Memory Diagnostic with no errors
- enabled Intel VT-x and VT-d in UEFI
- verified virtualization support from Windows
- inspected SMART attributes for the existing Seagate HDD
- completed an extended SMART self-test without error

### September 9, 2026 — Proxmox Deployment

- installed Proxmox VE bare metal on the existing HDD
- configured the management interface as `192.168.1.10/24` with `192.168.1.254` as the default gateway
- corrected enterprise repository configuration for a non-subscription installation
- updated the Proxmox host and activated the new kernel
- verified Linux bridge and default routing configuration
- inspected the Proxmox LVM-thin storage layout
- downloaded a Debian 13 LXC template
- created an unprivileged Debian test container
- validated DHCP, gateway connectivity, Internet routing, and DNS resolution
- updated the Debian container
- checked container memory and disk usage
- relocated the server to its permanent location
- verified unattended boot and remote Proxmox administration

---

This project is a work in progress. I will continue updating this README with the actual configurations, commands, problems, troubleshooting steps, and resolutions as additional services are deployed.
