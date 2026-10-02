# DHCP, Part 6: Final Project

> Design and build DHCP for a small company with three departments: subnetting, multiple scopes, exclusions, reservations, testing, documentation and backup.

[Previous](./05-troubleshooting.md) | [Course index](./README.md)

## Contents

- [Project scenario](#project-scenario)
- [Step 1: Design the network](#step-1-design-the-network)
- [Step 2: Lab infrastructure](#step-2-lab-infrastructure)
- [Step 3: Design the scopes](#step-3-design-the-scopes)
- [Step 4: Implementation](#step-4-implementation)
- [Step 5: Final testing](#step-5-final-testing)
- [Step 6: Documentation](#step-6-documentation)
- [Step 7: Troubleshooting scenarios](#step-7-troubleshooting-scenarios)
- [Step 8: Final review and backup](#step-8-final-review-and-backup)
- [Course summary](#course-summary)

---

## Project scenario

**Fanavaran Pars** is a small software company that has just grown. The CEO has asked you to set up the whole company network. There are three departments:

| Department | Staff | Needs |
|------------|-------|-------|
| **Management** | 10 | Addresses for laptops + a printer |
| **Sales** | 15 | Addresses for laptops + a printer + a scanner |
| **Technical** | 20 | Addresses for desktops + a file server + a printer |

**Total:** 45 staff + 3 printers + 1 scanner + 1 file server = **50 devices**.

**Requirements**

1. All staff devices get addresses automatically.
2. Printers and the file server must **always** have the same address.
3. Each department has its **own** address range.
4. Security: nobody should be able to join another department's network with a manual address.
5. Growth: the company expects to double in two years.

> [!WARNING]
> **About requirement 4:** DHCP alone cannot stop someone from typing a manual address. That needs switch features such as Port Security, DHCP Snooping, Dynamic ARP Inspection or 802.1X. In this project DHCP delivers the part that is in its power: **a separate address range per department**.

---

## Step 1: Design the network

**Goal:** decide which addresses are used for what.

**Why:** without a plan nobody knows who owns which address. Good design makes management easy.

One `/24` is enough for the whole company, but to keep departments **separate** we use subnetting and split it into `/26` subnets:

![Subnet plan](./images/company-subnet-plan.svg)

**Why `/26`?**

| Prefix | Usable addresses |
|--------|------------------|
| `/24` | 254 |
| `/25` | 126 |
| `/26` | 62 (our choice) |
| `/27` | 30 (may be too few) |

`/26` is enough for each department (10-20 staff plus peripherals), leaves room to grow, and the fourth block is kept for the future.

**`/26` calculation**

```text
Subnet mask:        255.255.255.192
Total addresses:    64
Usable addresses:   62 (the first is the network ID, the last is the broadcast)
```

| Department | Network | Mask | Usable range |
|------------|---------|------|--------------|
| Management | `192.168.1.0/26` | `255.255.255.192` | `.1` - `.62` |
| Sales | `192.168.1.64/26` | `255.255.255.192` | `.65` - `.126` |
| Technical | `192.168.1.128/26` | `255.255.255.192` | `.129` - `.190` |
| Future | `192.168.1.192/26` | `255.255.255.192` | `.193` - `.254` |

---

## Step 2: Lab infrastructure

**Goal:** extend the lab from one client to three (one per department) plus a simulated printer.

![Lab layout](./images/company-lab-topology.svg)

> [!NOTE]
> If you cannot create three clients, one is enough. During testing you move it between departments by changing its settings.

> [!IMPORTANT]
> **Limitation of this layout (please read).** All three scopes live on a single Host-Only network, and the server's NIC is `192.168.1.1/26`. Without a relay agent, Windows DHCP answers only from the scope that matches the subnet of the NIC that received the request, which here is the **Management** scope. Sales and Technical clients therefore will not receive addresses in this exact layout. The gateways `.65` and `.129` are also just addresses; no device answers on them yet.
>
> For a complete implementation:
> 1. Create a separate virtual network per department (VMnet2, VMnet3, VMnet4 or LAN Segments).
> 2. Add a **router VM** with a NIC in each subnet. It acts as the gateway (`.1`, `.65`, `.129`).
> 3. Configure **DHCP Relay** on the router (on Cisco: `ip helper-address <DHCP server>`) pointing to the DHCP server.
>
> If you only want to practice the scope structure, test one client on one subnet at a time.

---

## Step 3: Design the scopes

**Goal:** one separate scope per department.

**Why:** each department gets addresses from its own range, can have its own options (DNS, gateway), and a problem in one department does not affect the others.

### Scope 1: Management

```text
Name:    Management
Range:   192.168.1.1 to 192.168.1.62
Mask:    255.255.255.192
```

| Range | Use |
|-------|-----|
| `.1` - `.10` | Servers and router (**Exclusion**) |
| `.11` - `.49` | Staff (dynamic) |
| `.50` | Printer (**Reservation**) |
| `.51` - `.62` | Future use (**Exclusion**) |

### Scope 2: Sales

```text
Name:    Sales
Range:   192.168.1.64 to 192.168.1.126
Mask:    255.255.255.192
```

| Range | Use |
|-------|-----|
| `.65` - `.74` | Servers and router (**Exclusion**) |
| `.75` - `.109` | Staff (dynamic) |
| `.110`, `.111` | Printer and scanner (**Reservation**) |
| `.112` - `.126` | Future use (**Exclusion**) |

### Scope 3: Technical

```text
Name:    Technical
Range:   192.168.1.129 to 192.168.1.190
Mask:    255.255.255.192
```

| Range | Use |
|-------|-----|
| `.129` - `.139` | Servers and router (**Exclusion**) |
| `.140` - `.179` | Staff (dynamic) |
| `.180`, `.181` | File server and printer (**Reservation**) |
| `.182` - `.190` | Future use (**Exclusion**) |

---

## Step 4: Implementation

### 4.1 Set the server's static IP

```text
IP:       192.168.1.1
Subnet:   255.255.255.192
Gateway:  (empty)
DNS:      127.0.0.1
```

### 4.2 Install the DHCP role

**Server Manager → Add Roles and Features → DHCP Server**.

### 4.3 Create the Management scope

DHCP console → right-click **IPv4** → **New Scope**:

```text
Name:       Management
Start IP:   192.168.1.1
End IP:     192.168.1.62
Mask:       255.255.255.192

Exclusions:
  192.168.1.1  - 192.168.1.10   (servers and router)
  192.168.1.51 - 192.168.1.62   (future use)

Lease:      8 days
Router:     192.168.1.1
DNS:        192.168.1.1
```

### 4.4 Create the Sales scope

```text
Name:       Sales
Start IP:   192.168.1.65
End IP:     192.168.1.126
Mask:       255.255.255.192

Exclusions:
  192.168.1.65  - 192.168.1.74   (servers and router)
  192.168.1.112 - 192.168.1.126  (future use)

Lease:      8 days
Router:     192.168.1.65
DNS:        192.168.1.1
```

### 4.5 Create the Technical scope

```text
Name:       Technical
Start IP:   192.168.1.129
End IP:     192.168.1.190
Mask:       255.255.255.192

Exclusions:
  192.168.1.129 - 192.168.1.139  (servers and router)
  192.168.1.182 - 192.168.1.190  (future use)

Lease:      8 days
Router:     192.168.1.129
DNS:        192.168.1.1
```

### 4.6 Activate all scopes

Right-click each scope → **Activate**.

### 4.7 Create the reservations

For each device: scope → **Reservations** → right-click → **New Reservation**, then enter the device's MAC address and the reserved IP.

Example:

```text
Name:   Printer-Management
IP:     192.168.1.50
MAC:    00-11-22-33-44-55
Type:   Both
```

### 4.8 Optional: the same setup with PowerShell

```powershell
# Management
Add-DhcpServerv4Scope -Name "Management" -StartRange 192.168.1.1 -EndRange 192.168.1.62 `
    -SubnetMask 255.255.255.192 -LeaseDuration 8.00:00:00 -State Active
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.1  -EndRange 192.168.1.10
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.51 -EndRange 192.168.1.62
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -Router 192.168.1.1 -DnsServer 192.168.1.1
Add-DhcpServerv4Reservation -ScopeId 192.168.1.0 -IPAddress 192.168.1.50 `
    -ClientId "00-11-22-33-44-55" -Name "Printer-Management"

# Sales
Add-DhcpServerv4Scope -Name "Sales" -StartRange 192.168.1.65 -EndRange 192.168.1.126 `
    -SubnetMask 255.255.255.192 -LeaseDuration 8.00:00:00 -State Active
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.64 -StartRange 192.168.1.65  -EndRange 192.168.1.74
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.64 -StartRange 192.168.1.112 -EndRange 192.168.1.126
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.64 -Router 192.168.1.65 -DnsServer 192.168.1.1
Add-DhcpServerv4Reservation -ScopeId 192.168.1.64 -IPAddress 192.168.1.110 `
    -ClientId "00-11-22-33-44-66" -Name "Printer-Sales"
Add-DhcpServerv4Reservation -ScopeId 192.168.1.64 -IPAddress 192.168.1.111 `
    -ClientId "00-11-22-33-44-77" -Name "Scanner-Sales"

# Technical
Add-DhcpServerv4Scope -Name "Technical" -StartRange 192.168.1.129 -EndRange 192.168.1.190 `
    -SubnetMask 255.255.255.192 -LeaseDuration 8.00:00:00 -State Active
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.128 -StartRange 192.168.1.129 -EndRange 192.168.1.139
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.128 -StartRange 192.168.1.182 -EndRange 192.168.1.190
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.128 -Router 192.168.1.129 -DnsServer 192.168.1.1
Add-DhcpServerv4Reservation -ScopeId 192.168.1.128 -IPAddress 192.168.1.180 `
    -ClientId "00-11-22-33-44-88" -Name "FileServer"
Add-DhcpServerv4Reservation -ScopeId 192.168.1.128 -IPAddress 192.168.1.181 `
    -ClientId "00-11-22-33-44-99" -Name "Printer-Technical"
```

---

## Step 5: Final testing

For each client run `ipconfig /release`, `ipconfig /renew`, `ipconfig /all`.

### Test 1: Management client

Expected:

```text
IP:       192.168.1.11 (any address from .11 to .49)
Subnet:   255.255.255.192
Gateway:  192.168.1.1
DNS:      192.168.1.1
DHCP Server: 192.168.1.1
```

### Test 2: Sales client

```text
IP:       192.168.1.75 (any address from .75 to .109)
Subnet:   255.255.255.192
Gateway:  192.168.1.65
```

### Test 3: Technical client

```text
IP:       192.168.1.140 (any address from .140 to .179)
Subnet:   255.255.255.192
Gateway:  192.168.1.129
```

### Test 4: Communication between clients

From the Management client:

```cmd
ping 192.168.1.75      (Sales client)
ping 192.168.1.140     (Technical client)
```

Because the subnets are different, clients **cannot reach each other without a router**. This is expected and is the purpose of subnet separation.

> [!TIP]
> For departments to communicate you need a **router** between the subnets. Real networks have one; in the lab you can use a Windows Server with Routing and Remote Access.

### Test 5: Reservations

On the server:

```powershell
Get-DhcpServerv4Reservation -ScopeId 192.168.1.0
```

You should see the printer reservation.

### Test 6: Scope statistics

```powershell
Get-DhcpServerv4ScopeStatistics
```

Example output (your numbers will differ):

```text
ScopeId         : 192.168.1.0
Free            : 30
InUse           : 10
PercentageInUse : 25
```

---

## Step 6: Documentation

**Why:** in real networks documentation is vital. When a new engineer arrives they must know how the network is built.

**1. Scopes**

| Department | Scope | Range | Subnet | Gateway | DNS |
|------------|-------|-------|--------|---------|-----|
| Management | Management | .1 - .62 | /26 | .1 | .1 |
| Sales | Sales | .65 - .126 | /26 | .65 | .1 |
| Technical | Technical | .129 - .190 | /26 | .129 | .1 |

**2. Reservations**

| Device | Department | IP | MAC address |
|--------|------------|----|-------------|
| Management printer | Management | .50 | 00-11-22-33-44-55 |
| Sales printer | Sales | .110 | 00-11-22-33-44-66 |
| Sales scanner | Sales | .111 | 00-11-22-33-44-77 |
| File server | Technical | .180 | 00-11-22-33-44-88 |
| Technical printer | Technical | .181 | 00-11-22-33-44-99 |

**3. Exclusions**

| Department | Exclusion | Reason |
|------------|-----------|--------|
| Management | .1 - .10 | Servers and router |
| Management | .51 - .62 | Future use |
| Sales | .65 - .74 | Servers and router |
| Sales | .112 - .126 | Future use |
| Technical | .129 - .139 | Servers and router |
| Technical | .182 - .190 | Future use |

---

## Step 7: Troubleshooting scenarios

**1. A Management client receives a Sales address**
Cause: the client is on the wrong VMnet, or another DHCP server is answering. Fix: check the adapter, disable VMware's DHCP, restart the client.

**2. A printer gets the wrong address**
Cause: wrong reservation or MAC. Fix: re-check the MAC address, delete and recreate the reservation, restart the printer.

**3. Departments cannot talk to each other**
Cause: separate subnets with no router. This is expected. Fix: add a router (or Routing and Remote Access on Windows Server).

**4. A department runs out of addresses**
Cause: more devices than free addresses. Fix: shrink exclusions, move to a larger subnet (`/26` → `/25`), or add a scope.

---

## Step 8: Final review and backup

### Checklist

```text
[ ] Server has a static IP (192.168.1.1)
[ ] DHCPServer service is running
[ ] Management scope is active
[ ] Sales scope is active
[ ] Technical scope is active
[ ] Exclusions are configured
[ ] Reservations are configured
[ ] Management client got a correct address
[ ] Sales client got a correct address
[ ] Technical client got a correct address
[ ] Printers received their reserved addresses
[ ] Documentation is complete
[ ] A configuration backup exists
```

### Backup and restore

```powershell
Export-DhcpServer -File "C:\Backup\DHCP-Config.xml" -Leases -Force
```

Restore:

```powershell
Import-DhcpServer -File "C:\Backup\DHCP-Config.xml" -BackupPath "C:\Backup\" -Leases -Force
```

---

## Course summary

| Part | What you learned |
|------|------------------|
| 1 | IP, subnet mask, gateway, DNS |
| 2 | Scope, pool, lease, reservation, exclusion, DORA, ports, broadcast, relay |
| 3 | VMware lab, DHCP role, scope, first lease |
| 4 | Lease and renewal, exclusion, reservation, APIPA, commands |
| 5 | Twelve troubleshooting scenarios and the golden checklist |
| 6 | Subnetting, multiple scopes, documentation, backup |

You can now design a small network, install and configure a DHCP server, create scopes, exclusions and reservations, troubleshoot common DHCP faults, document the result and back it up.

**Where to go next:** DNS Server, VLANs and routing, DHCP failover, and switch security features such as DHCP Snooping.

---

[Previous](./05-troubleshooting.md) | [Course index](./README.md)
