# Lenovo Home Server Lab

I was given an older Lenovo desktop and wanted to determine whether it was worth repurposing as a home server and virtualization lab rather than leaving it unused.

The machine initially powered on but produced no display. After troubleshooting that issue, I used Windows tools and PowerShell to inventory the hardware, opened the system to inspect its expansion options, and developed a low-cost upgrade plan.

My current goal is to use the machine as a Proxmox host for learning virtualization and running a few useful home network services.

This repository documents the project as I work through it.

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

### No display output

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

The CPU reports the required virtualization extensions, but System Information showed:

```text
Virtualization Enabled in Firmware: No
```

I therefore need to enable hardware virtualization in UEFI before using the system as a virtualization host.

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

Each module is 4 GB, giving the system its current 8 GB total.

![PowerShell memory and disk inventory](images/powershell-hardware-inventory.png)

Physical inspection showed four DIMM slots, with two currently populated.

![Internal system overview](images/lenovo-internal-overview.jpg)

For the first upgrade, I decided to add another 2 x 4 GB DDR3-1600 kit rather than replacing the existing memory.

That will bring the machine to:

```text
Current:  8 GB
Planned: 16 GB
```

Sixteen gigabytes should give me considerably more room for lightweight virtual machines and containers while keeping the amount invested in this older platform low.

## Storage Investigation

Disk Management showed one physical disk with approximately 465 GB of usable capacity.

![Windows Disk Management](images/disk-management.png)

The disk contains the existing Windows installation with:

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

The drive currently reports an OK status, but because it is an older HDD I don't intend to rely on it as the only location for important household data.

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

The system has four DIMM slots with two currently occupied, which makes the planned memory upgrade straightforward.

The NVIDIA T1000 occupies the primary PCIe slot, with additional expansion available below it.

### Drive cage

The existing Seagate HDD is mounted in Lenovo's drive assembly.

![Lenovo drive cage](images/lenovo-drive-cage.png)

There is enough flexibility in the chassis to continue investigating an SSD and additional storage, but I decided not to buy storage immediately.

I looked at several SATA SSD options during the assessment, but the prices I found did not make sense relative to the age and value of the machine. For now, I would rather upgrade the inexpensive DDR3 memory, continue testing the computer, and add an SSD when I find a reasonable option.

## Current Upgrade Plan

My approach with this machine is to avoid spending money simply because something *can* be upgraded.

The first upgrade will be:

**8 GB → 16 GB DDR3-1600**

After installing the additional RAM, I will confirm that all 16 GB is detected and test the memory before moving on.

An SSD is still planned, but I am not treating it as urgent enough to justify overpaying for one.

Longer term, I expect the storage layout to separate the hypervisor/VM storage from important household data rather than trying to make the existing 500 GB HDD handle everything.

## Intended Use

Assuming the hardware continues to test well, I plan to install Proxmox VE and use this machine as both a home server and an IT lab.

The first workloads I am considering are:

- AdGuard Home or Pi-hole for local DNS filtering
- Linux containers and VMs
- a Windows Server lab
- SMB file sharing
- PC/server backups
- basic monitoring

I want to keep experimental lab systems separate from services that other devices in the house depend on. For example, a Windows Server DNS or Active Directory lab should be something I can break and rebuild without taking down normal household DNS.

The exact design will be documented once I actually deploy it rather than designing the entire environment in advance.

## Next Steps

- [ ] Install additional 8 GB DDR3 memory
- [ ] Confirm 16 GB is detected
- [ ] Run memory testing
- [ ] Enable Intel virtualization in UEFI
- [ ] Perform additional health testing on the existing HDD
- [ ] Source a reasonably priced SATA SSD
- [ ] Install Proxmox VE
- [ ] Configure management networking
- [ ] Deploy first VM or LXC container
- [ ] Configure backups

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
- confirmed virtualization support is present but disabled in firmware
- opened the chassis and inspected the memory, storage, PCIe, and drive layout
- decided to begin with a low-cost RAM upgrade rather than immediately investing heavily in storage

The next session will be the memory upgrade and virtualization-readiness testing.

---

This project is a work in progress. I will update this README with the actual configuration, commands, problems, and resolutions as the server is built.
