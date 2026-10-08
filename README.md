# IT Support Portfolio

I'm Peter, a Calgary-based chef working towards my first IT support role.

I've spent years working in busy kitchens, where a problem rarely arrives at a convenient time. You have to figure out what's happening, decide what matters first, communicate with the people around you, and get things working again. That's the part of the job I've always enjoyed, and it's a big part of what drew me towards IT.

I completed my **CompTIA A+** in April 2026. Outside of work, I've been building a homelab and working through Windows, Linux, networking, and troubleshooting projects. This portfolio is where I keep track of what I've actually done, including the things that didn't work on the first try.

These are **personal projects and lab exercises**, not paid IT administration experience.

## Projects

### [Home Server and Windows Domain Lab](./Home-Server-Lab)

I started with an older Lenovo desktop that wasn't displaying anything on the monitor. After sorting out the hardware issue, I upgraded its memory, installed Proxmox, and began using it as a home server.

It's now running Debian containers, AdGuard Home, a Windows Server 2022 domain controller, and a Windows 11 Pro client. I built the Active Directory domain `ad.seasonandsavour.com`, joined the client, and verified its secure channel.

One of the more useful problems came during the client setup: it could reach the domain controller, but normal DNS lookups were going to the ISP's IPv6 resolvers instead of my AD DNS server. Working through that made the difference between *network connectivity* and *service discovery* much clearer to me.

**What I've worked with:** Proxmox VE, Windows Server 2022, Windows 11 Pro, Active Directory Domain Services, DNS, PowerShell, VirtIO drivers, Debian Linux, and basic network troubleshooting.

**Still to do:** domain-user sign-in, OUs and groups, Group Policy, shared-folder permissions, backup and restore testing, and more support scenarios.

[Read the Home Server Lab](./Home-Server-Lab) · [Windows Server deployment](./Home-Server-Lab/docs/windows-server-ad-ds-deployment.md) · [Windows client domain join](./Home-Server-Lab/docs/windows-client-domain-join.md)

### [Active Directory Lab](./Active-Directory-Lab)

My earlier Active Directory practice focused on getting comfortable with accounts, groups, permissions, and common Windows support tasks. I'm now continuing that work in the Proxmox-based domain above, where I can test changes on a real client VM and document the results.

### [Secure Hosting with Docker and Cloudflare](./Secure-Hosting-Docker)

I wanted a way to host an application for friends without exposing services directly through my home router. I used Docker on Ubuntu with persistent storage and Cloudflare Tunnel for remote access, then tested connectivity and recovery.

### [Linux Hardening Lab](./Linux-Hardening-Lab)

A practice environment for working through SSH settings, user accounts, firewall rules, and reducing unnecessary services. I documented what I changed and how I checked the results.

### [IT Support Lab](./IT-Support-Lab)

Practice with everyday support problems: account access, passwords, shared folders, permissions, printers, and Windows troubleshooting. The aim is to get comfortable diagnosing an issue rather than just memorising a fix.

### [Troubleshooting Playbook](./Troubleshooting-Playbook)

My notes on how to approach common problems involving connectivity, DNS, permissions, Windows services, and system performance. I use them to keep my troubleshooting consistent and to remind myself what to check next.

## Tools I've used in labs and personal projects

- **Windows:** Windows 10/11, Windows Server 2022, Active Directory, PowerShell, Command Prompt
- **Linux and virtualization:** Ubuntu, Debian, Proxmox VE, QEMU/KVM, Docker
- **Networking:** TCP/IP, DHCP, DNS, ICMP, Cloudflare Tunnel, UFW
- **Support tasks:** hardware and driver troubleshooting, account administration, permissions, service checks, and technical documentation

I'm continuing to learn these tools. Listing one here means I've worked with it in a lab or personal project, not that I've administered it professionally.

## Certifications and current learning

- **CompTIA A+** — completed April 2026
- **CompTIA Network+** — studying; not yet certified

## What I'm looking for

I'm looking for my first **Help Desk, IT Support, Service Desk, or Desktop Support** role in Calgary, with remote opportunities also of interest. I'd like to work somewhere I can help people solve problems, learn from experienced technicians, and keep building the practical skills I've started developing here.
