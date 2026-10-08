# Windows 11 Client Deployment and Active Directory Domain Join

**Environment:** Independent Proxmox homelab — not professional production experience  
**Status:** Domain joined and secure channel validated; domain-user sign-in and AD computer-object screenshot pending.

## Objective
Deploy a Windows 11 Pro workstation on Proxmox, configure it to locate a Windows Server 2022 domain controller through AD DNS, join `ad.seasonandsavour.com`, and verify the workstation's domain trust.

## Environment
| Component | Configuration |
|---|---|
| Hypervisor | Proxmox VE 9.2 |
| VM | 103, `win11-client` |
| Windows hostname | `WIN11-CLIENT` |
| Guest OS | Windows 11 Pro |
| Assigned memory | 4 GiB |
| Virtual network | Red Hat VirtIO Ethernet adapter, bridged to LAN |
| IPv4 address observed | `192.168.1.64/24` via DHCP |
| Gateway / DHCP server | `192.168.1.254` |
| Domain controller | `DC-01`, `192.168.1.12` |
| AD domain | `ad.seasonandsavour.com` |
| Initial local account | `LabAdmin` |

## Deployment and baseline
Installed Windows 11 Pro in VM 103, completed local-account setup, and verified the VirtIO network interface. The initial `ipconfig /all` output showed DHCP addressing, the router as the gateway, and DNS servers from the household network. Connectivity checks to the gateway and DC returned replies.

## DNS investigation and resolution
The client could reach `192.168.1.12`, and explicit DNS queries against that address resolved the AD domain's A record and the LDAP domain-controller SRV record. However, default `nslookup` calls selected ISP-provided IPv6 DNS servers and returned non-existent-domain responses for the private AD zone. This showed that network reachability and correct default DNS selection were separate issues.

Configured the Windows client's IPv4 DNS to use `192.168.1.12`. Because IPv6 router-advertised resolvers remained in use, adjusted the client's IPv6 DNS configuration through the network adapter interface. After this change, `Get-DnsClientServerAddress` showed the AD DNS server for IPv4 and no IPv6 DNS server addresses, and unqualified `nslookup` queries returned the AD domain and LDAP SRV record from `192.168.1.12`. IPv6 itself was not disabled. The broader household IPv6 DNS design remains a follow-up consideration.

## Domain join and verification
Used System Properties (`sysdm.cpl`) to join `WIN11-CLIENT` to `ad.seasonandsavour.com` with domain administrator credentials. Windows accepted the join and requested a restart. After restarting:

```powershell
Test-ComputerSecureChannel
# True

(Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters').Domain
# ad.seasonandsavour.com
```

The secure-channel result confirms the computer trust relationship was working when tested. The registry result confirms the primary DNS domain suffix; it is supporting evidence rather than a standalone domain-membership test.

## Separate unresolved observation
A `Get-CimInstance Win32_ComputerSystem` query returned `Invalid class (0x80041010)` after earlier mistyped attempts. This did not prevent the secure-channel check from succeeding. CIM/WMI class availability has **not** been repaired or independently diagnosed and should not be described as resolved.

## Evidence
| Evidence | File |
|---|---|
| Domain join accepted; restart required | [client-08-domain-join-restart-required.png](../images/client-08-domain-join-restart-required.png) |
| Post-restart secure channel `True` | [client-10-secure-channel-verified.png](../images/client-10-secure-channel-verified.png) |
| CIM error retained for later investigation | [client-09-cim-verification-error.png](../images/client-09-cim-verification-error.png) |

Additional baseline and DNS screenshots are in [`../images/`](../images/); see the evidence manifest for source filenames and hashes.

## Skills demonstrated
Windows 11 VM deployment; VirtIO networking; DHCP and TCP/IP validation; DNS A/SRV record troubleshooting; IPv4/IPv6 DNS resolver investigation; Windows domain joining; PowerShell trust verification; documenting an unresolved diagnostic separately from a successful service validation.

## Next steps
1. On DC-01, capture `WIN11-CLIENT` in Active Directory Users and Computers (Computers container or assigned OU).
2. Create a limited-privilege test domain user and verify interactive sign-in on `WIN11-CLIENT`.
3. Create a small OU/group structure and one test Group Policy; verify application with `gpresult`.
4. Add SMB/NTFS permissions exercises and deliberately broken support scenarios.
5. Investigate the CIM/WMI invalid-class error separately if it persists.
