# Partitioned macOS to run Windows 10

![macOS](https://img.shields.io/badge/macOS-Apple-grey) ![Windows 10](https://img.shields.io/badge/Windows-10-blue)

## Table of Contents
- [Project Summary](#project-summary)
- [Motivation](#motivation)
- [Environment](#environment)
- [What I actually did](#what-i-actually-did)
- [Problems encountered and fixes](#problems-encountered-and-fixes)
- [Outcomes and lessons learned](#outcomes-and-lessons-learned)
- [Next steps](#next-steps)


**Project Summary:** Dual‑boot and VM lab documenting how I partitioned a 2019 MacBook Pro to run Windows 10, what went wrong, and what I learned.

### Motivation
I wanted hands‑on Windows admin experience while keeping my macOS workstation intact. My goals were to compare VM versus native performance, practice safe partitioning, and produce a client that could later join an AD domain.

### Environment
- - **Host:** MacBook Pro 2019 i7 — macOS: **TBD**
- **Target:** Windows 10 ISO
- **VM:** VirtualBox for initial testing  
- **Tools I used:** Disk Utility; Boot Camp Assistant (native installer present on this model); VirtualBox; Time Machine; Homebrew (installed but later not required for the native path); PowerShell for Windows checks

### What I actually did
1. **Backed up first.** I created a Time Machine backup to an external drive and verified a few files to ensure the backup was usable.  
2. **Tested in a VM.** I installed Windows 10 in VirtualBox using the ISO to confirm the installer and drivers worked. I installed Guest Additions and took a snapshot named `baseline`. This gave me a safe rollback point.  
3. **Realized no USB was needed.** Because my MacBook Pro 2019 i7 already had the necessary native installer support, I did not need to create an external USB installer or rely on Homebrew for the install path. I had installed Homebrew earlier but it turned out unnecessary for the native Boot Camp workflow.  
4. **Partitioned the disk.** I used Disk Utility to resize my macOS volume and create a Windows partition. I allocated around 120 GB for Windows and left the partition for the Windows installer to format. During this step I hit a limitation: the partition slider in Disk Utility would only move a small amount and would not resize to the full space I expected. I retried, freed additional space, and proceeded after confirming backups.  
5. **Installed Windows natively.** I used Boot Camp Assistant to proceed with the native install and selected the partition during Windows setup, formatting it as NTFS when prompted. After the first boot into Windows I installed Boot Camp drivers and Windows updates.  
6. **Prepared for AD testing.** I installed RSAT and a few admin tools on the Windows client so it would be ready to join a domain in a later phase.

### Problems encountered and fixes
- **Partition slider limited movement.** Disk Utility initially would not resize the macOS volume as far as I wanted. I freed additional disk space, closed background apps, and retried. When Disk Utility still behaved inconsistently I restored from the Time Machine image to confirm backups were valid, then retried the resize.  
- **Driver gaps after install.** A few devices lacked drivers after the first Windows boot. Reinstalling the Boot Camp driver package and running Windows Update resolved most missing drivers.  
- **Network DHCP issue in bridged mode.** My first bridged configuration did not receive an IP from the router; switching to a different physical NIC and re‑enabling DHCP fixed the issue.


### Outcomes and lessons learned
- Testing in a VM first saved time and prevented a bad native install.  
- Always verify backups before repartitioning; I restored once from the image when Disk Utility behaved unexpectedly.  
- Boot Camp drivers are essential for native installs; keep a copy on USB.  
- Homebrew was useful for other tasks but not required for the native Boot Camp workflow on my 2019 MacBook Pro.


### Next steps
- Build a Windows Server VM for Active Directory and join this client to the domain.  
- Expand this repo with a sanitized step‑by‑step appendix that reproduces the exact commands and screenshots once I recover them from logs or snapshots.

---
