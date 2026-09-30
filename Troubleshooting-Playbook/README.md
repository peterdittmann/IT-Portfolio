# Windows Troubleshooting Playbook

## Project Overview

Created a series of controlled Windows troubleshooting scenarios to practise diagnosing common connectivity, DNS, performance, permissions, service, and storage problems.

The focus of this project is the troubleshooting process: identifying the reported symptom, gathering relevant information, narrowing the problem to a likely component, applying a targeted change, and verifying that the original issue is resolved.

The scenarios were intentionally created in a Windows virtual machine so that I could repeatedly practise a structured approach to technical problems.

---

## Environment & Tools

| Component | Configuration |
| --- | --- |
| Host OS | Ubuntu Linux |
| Virtualization | QEMU/KVM with virt-manager |
| Guest OS | Windows 10 |
| Command Line | Command Prompt |
| Windows Tools | Task Manager, Services, Network Settings, File Explorer |

Commands documented in the scenarios include:

```text
ipconfig
ipconfig /renew
ping
```

---

# Troubleshooting Method

I used the same general process throughout the scenarios:

1. **Identify the problem**  
   Establish the original symptom and what is not working.

2. **Gather information**  
   Inspect the relevant network configuration, service, permissions, storage, or system state.

3. **Isolate the problem**  
   Use the available evidence to narrow the issue to a particular component or configuration.

4. **Apply a targeted fix**  
   Change the configuration responsible for the problem rather than making unrelated changes.

5. **Verify the resolution**  
   Retest the original problem and confirm that normal functionality has been restored.

A major goal of the project was to practise using test results to determine the next troubleshooting step instead of immediately applying a possible fix.

---

# Troubleshooting Cases

## Case 1 – No Internet Connectivity

### Initial Report

Internet connectivity is unavailable and websites cannot be accessed.

### Diagnostic Process

Checked the network adapter state and reviewed the Windows IP configuration using:

```text
ipconfig
```

The system did not have a valid IP configuration because the network adapter was disabled.

### Finding

The disabled network adapter prevented the system from obtaining the network configuration required for connectivity.

### Resolution

Re-enabled the network adapter and renewed the IP configuration using:

```text
ipconfig /renew
```

### Verification

Tested connectivity by pinging an external domain and confirmed that network connectivity had been restored.

### Evidence – Problem Identified

![No Internet Issue](screenshots/project3-no-internet.png)

> Network adapter disabled, resulting in no connectivity.

### Evidence – IP Configuration

![IP Config Failure](screenshots/project3-ipconfig-failure.png)

> System showing no valid IP configuration while the adapter is disabled.

### Evidence – Resolution

![IP Renew](screenshots/project3-ipconfig-renew.png)

> IP configuration renewed after re-enabling the network adapter.

### Evidence – Verification

![Ping Success](screenshots/project3-ping-success.png)

> Successful ping used to verify restored connectivity.

---

## Case 2 – DNS Resolution Failure

### Initial Report

Websites cannot be reached even though the system still has network connectivity.

### Diagnostic Process

Tested connectivity in two different ways:

```text
ping google.com
ping 8.8.8.8
```

The IP-address test succeeded while the hostname test failed.

This distinction helped narrow the problem. The system could communicate over the network using an IP address, but it could not successfully resolve a hostname.

### Finding

The DNS server configuration was incorrect, preventing hostname resolution.

### Resolution

Updated the network adapter configuration to use a valid DNS server.

### Verification

Retested connectivity using a domain name and confirmed that hostname resolution was working.

### Evidence – DNS Failure

![DNS Failure](screenshots/project3-dns-failure.png)

> Domain-name resolution failing while IP connectivity remains available.

### Evidence – Connectivity Testing

![Ping Tests](screenshots/project3-ping-tests.png)

> Testing IP and hostname connectivity separately to narrow the issue toward DNS resolution.

### Evidence – DNS Configuration

![DNS Settings](screenshots/project3-dns-settings.png)

> DNS server configuration corrected in the network adapter settings.

### Evidence – Verification

![DNS Success](screenshots/project3-dns-success.png)

> Domain name successfully resolves after correcting the DNS configuration.

---

## Case 3 – Slow System Performance

### Initial Report

The Windows system experiences slow startup and reduced performance.

### Diagnostic Process

Reviewed the applications configured to run automatically during startup using Windows Task Manager.

Multiple unnecessary applications were configured to launch with the system.

### Finding

Excessive startup applications were contributing to system resource usage during startup.

### Resolution

Disabled unnecessary startup applications.

### Verification

Retested the system after the changes and observed improved startup performance and reduced system load.

### Evidence – Before

![Startup Before](screenshots/project3-startup-before.png)

> Multiple applications configured to run automatically during startup.

### Evidence – Resolution

![Startup After](screenshots/project3-startup-after.png)

> Non-essential startup applications disabled.

### Evidence – Verification

![Performance Improved](screenshots/project3-taskmanager-performance.png)

> System performance reviewed after startup configuration was changed.

---

## Case 4 – Permission Denied

### Initial Report

A user receives an **Access Denied** error when attempting to access a folder.

### Diagnostic Process

Reviewed the folder's Windows security settings and the permissions assigned to the affected user.

The user's permissions did not provide the required folder access.

### Finding

The user lacked the permissions required to access the folder.

### Resolution

Modified the folder permissions to provide the necessary access.

### Verification

Retested access using the affected user and confirmed that the folder could be opened successfully.

### Evidence – Problem Identified

![Access Denied](screenshots/project3-access-denied.png)

> User unable to access the restricted folder.

### Evidence – Permission Review

![Permissions Before](screenshots/project3-permissions-before.png)

> Folder security settings reviewed to identify the missing access.

### Evidence – Resolution

![Permissions After](screenshots/project3-permissions-after.png)

> Required permissions applied to the folder.

### Evidence – Verification

![Folder Access](screenshots/project3-folder-access.png)

> Successful folder access after the permissions were updated.

---

## Case 5 – Print Spooler Failure

### Initial Report

Print jobs remain stuck in the queue and do not complete.

### Diagnostic Process

Checked the printer queue and reviewed the Windows services responsible for printing.

The Print Spooler service was stopped.

### Finding

The stopped Print Spooler service prevented print jobs from being processed.

### Resolution

Restarted the Print Spooler service.

### Verification

Confirmed that the Print Spooler was running and that print functionality had been restored.

### Evidence – Problem Identified

![Spooler Stopped](screenshots/project3-spooler-stopped.png)

> Windows Print Spooler service not running.

### Evidence – Resolution

![Spooler Restart](screenshots/project3-spooler-restart.png)

> Print Spooler service restarted.

### Evidence – Verification

![Spooler Running](screenshots/project3-spooler-running.png)

> Print Spooler running after the change.

---

## Case 6 – Low Disk Space

### Initial Report

The system reports warnings that available disk space is critically low.

### Diagnostic Process

Reviewed storage usage in Windows system settings to determine whether available storage was causing the warning.

The disk was filled with unnecessary files.

### Finding

Insufficient free disk space was triggering the storage warning.

### Resolution

Removed unnecessary files to recover storage capacity.

### Verification

Reviewed storage again and confirmed that available disk space had increased and the warning was resolved.

### Evidence – Problem Identified

![Disk Warning](screenshots/project3-disk-warning.png)

> Windows reporting critically low available storage.

### Evidence – Storage Review

![Disk Full](screenshots/project3-disk-full.png)

> Storage usage reviewed while investigating the low-space warning.

### Evidence – Cleanup

![Cleanup](screenshots/project3-cleanup.png)

> Unnecessary files removed to recover storage capacity.

### Evidence – Verification

![Disk Recovered](screenshots/project3-disk-recovered.png)

> Available storage increased after cleanup.

---

# Diagnostic Reasoning

These scenarios helped reinforce that similar user symptoms can require different troubleshooting paths.

For example, the two network scenarios initially involve an inability to reach network resources, but the tests identify different problems.

### Connectivity Failure

```text
No Internet Connectivity
        ↓
Check Network Adapter
        ↓
Adapter Disabled
        ↓
Enable Adapter
        ↓
Renew IP Configuration
        ↓
Test Connectivity
        ↓
Connectivity Restored
```

### DNS Failure

```text
Website / Hostname Fails
        ↓
Test Hostname
        ↓
Hostname Fails
        ↓
Test Known IP Address
        ↓
IP Connectivity Works
        ↓
Investigate DNS
        ↓
Correct DNS Configuration
        ↓
Retest Hostname
        ↓
Name Resolution Restored
```

The DNS scenario was particularly useful because successful IP connectivity provided evidence that the system still had network connectivity. The failed hostname test narrowed the investigation toward name resolution rather than treating the problem as a complete network failure.

---

# Skills Practised

## Windows Troubleshooting

- Windows 10
- Command Prompt
- Windows Network Settings
- Windows Services
- Task Manager
- File and folder security settings
- Storage management

## Networking

- IP configuration
- Connectivity testing
- DNS troubleshooting
- Differentiating IP connectivity from hostname resolution

## User & System Support

- Folder permission troubleshooting
- Print Spooler troubleshooting
- Startup performance troubleshooting
- Low disk-space investigation
- Configuration validation

## Troubleshooting Process

- Identifying symptoms
- Gathering relevant system information
- Narrowing the affected component
- Applying targeted changes
- Retesting the original problem
- Verifying successful resolution
- Documenting the troubleshooting process

---

# Support Relevance

This project provides hands-on practice with several types of problems commonly encountered in Windows support environments:

- Loss of network connectivity
- DNS resolution failures
- Slow system startup
- User access and permission problems
- Windows service failures
- Low disk space

The environment is a controlled lab rather than professional production experience. The value of the project is in practising a repeatable diagnostic process and documenting how test results were used to narrow each problem before applying a resolution.

---

# What I Learned

The main lesson from this project was that troubleshooting is more effective when each test is used to narrow the problem.

The DNS scenario demonstrated this clearly. Testing an IP address separately from a hostname helped distinguish a name-resolution problem from a complete connectivity failure.

The other scenarios reinforced the same general process: inspect the part of the system related to the reported symptom, identify the configuration or service responsible, make a targeted change, and then retest the original issue.

The project also reinforced the importance of verification. A configuration change is not enough on its own; the original problem should be tested again to confirm that normal functionality has actually been restored.

---

# Future Development

As my home server and Windows lab environments expand, I plan to add troubleshooting case studies based on problems encountered while building and maintaining those systems.

Future cases will be documented only after the work is performed, with an emphasis on:

- Initial symptoms
- Information gathered
- Diagnostic tests performed
- Possible causes considered
- Evidence used to narrow the issue
- Resolution
- Verification
- Lessons that can be reused when troubleshooting similar problems

This will allow the playbook to develop from controlled troubleshooting exercises into a broader record of problems encountered and resolved while working with Windows, networking, virtualization, and other lab infrastructure.
