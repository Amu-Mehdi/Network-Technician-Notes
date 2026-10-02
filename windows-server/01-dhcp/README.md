# DHCP: From Concepts to a Hands-On Project

Class notes on **DHCP** with **Windows Server**, using a hands-on lab in **VMware Workstation**.
The path goes from the basics, through building and testing the lab, to troubleshooting and a complete final project.

## Learning path

| # | File | Topic | Type |
|---|------|-------|------|
| 1 | [Networking fundamentals](./01-networking-fundamentals.md) | IP, subnet mask, gateway, DNS | Theory |
| 2 | [DHCP concepts](./02-dhcp-concepts.md) | Scope, lease, reservation, exclusion, DORA, ports, relay | Theory |
| 3 | [Lab setup](./03-lab-setup.md) | 18 steps: VMware network, static IP, DHCP role, scope, first lease | Hands-on |
| 4 | [Hands-on experiments](./04-hands-on-experiments.md) | 8 experiments: lease, exclusion, reservation, renew, APIPA | Hands-on |
| 5 | [Troubleshooting](./05-troubleshooting.md) | 12 real-world scenarios and a checklist | Hands-on |
| 6 | [Final project](./06-final-project.md) | DHCP for a company with three departments | Project |

## Lab topology

![Lab topology](./images/lab-topology.svg)

## Prerequisites

| Item | Requirement |
|------|-------------|
| VMware Workstation | Version 15 or later (Player also works) |
| Windows Server | 2016, 2019 or 2022 |
| Windows Client | Windows 10 or 11 |
| Hardware | At least 8 GB RAM (16 GB recommended) and 100 GB free disk |

## The DORA process at a glance

![DORA process](./images/dora-process.svg)

| Port | Role |
|------|------|
| UDP 67 | DHCP server |
| UDP 68 | DHCP client |

## Command cheat sheet

**On the client (CMD):**

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

**On the server (PowerShell):**

```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
Get-DhcpServerv4Reservation -ScopeId 192.168.10.0
Get-DhcpServerv4ScopeStatistics
Export-DhcpServer -File "C:\Backup\DHCP-Config.xml" -Leases -Force
```

## Notes

- Read the files in numeric order.
- Each file has a table of contents and previous/next links.
- Part 6 has an important note about the limits of running several scopes on one lab network. Read it before building.
- Diagrams are SVG files in [`images/`](./images).
