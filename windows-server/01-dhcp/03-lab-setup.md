# DHCP, Part 3: Building the Lab in VMware

> From an empty VMware network to a client receiving its first address from your own DHCP server.

[Previous](./02-dhcp-concepts.md) | [Course index](./README.md) | [Next: Experiments](./04-hands-on-experiments.md)

## Contents

- [Lab map](#lab-map)
- [Prerequisites](#prerequisites)
- [Step 1: Check VM network settings](#step-1-check-vm-network-settings)
- [Step 2: Choose the lab network](#step-2-choose-the-lab-network)
- [Step 3: Choose the server IP](#step-3-choose-the-server-ip)
- [Step 4: Set a static IP on the server](#step-4-set-a-static-ip-on-the-server)
- [Step 5: Install the DHCP Server role](#step-5-install-the-dhcp-server-role)
- [Step 6: Create the DHCP scope](#step-6-create-the-dhcp-scope)
- [Step 7: Define the IP range](#step-7-define-the-ip-range)
- [Step 8: Exclusions](#step-8-exclusions)
- [Step 9: Lease duration](#step-9-lease-duration)
- [Step 10: Default gateway option](#step-10-default-gateway-option)
- [Step 11: DNS server option](#step-11-dns-server-option)
- [Step 12: Activate the scope](#step-12-activate-the-scope)
- [Step 13: Verify the DHCP server](#step-13-verify-the-dhcp-server)
- [Step 14: Set the client to obtain an IP automatically](#step-14-set-the-client-to-obtain-an-ip-automatically)
- [Step 15: Get an IP from DHCP](#step-15-get-an-ip-from-dhcp)
- [Step 16: Verify the received settings](#step-16-verify-the-received-settings)
- [Step 17: Test connectivity](#step-17-test-connectivity)
- [Step 18: Check the lease on the server](#step-18-check-the-lease-on-the-server)
- [Result](#result)

---

## Lab map

![Lab topology](./images/lab-topology.svg)

**Why this layout?**

- **Host-Only network:** a private network shared only between the VMs and your PC. It is not connected to the internet or to your real network, so the lab is **isolated and safe**.
- **Server with a static IP:** the server must always be at a known address.
- **Client with automatic IP:** the client exists to test DHCP.

## Prerequisites

| Item | Requirement |
|------|-------------|
| VMware Workstation | Version 15 or later (16/17 recommended). Workstation Player also works. |
| Windows Server VM | 2016, 2019 or 2022, installed and bootable |
| Windows Client VM | Windows 10 or 11, installed and bootable |
| RAM | At least 8 GB (16 GB recommended) |
| Disk | At least 100 GB free |

> [!WARNING]
> If your virtual machines are not ready, create them and install Windows first, then come back here.

---

## Step 1: Check VM network settings

**Goal:** both VMs must be on the same shared, isolated network.

**Why:** if they are connected to your real network, your real DHCP server (for example your home router) can answer the clients and ruin the test.

**Steps**

1. Open VMware Workstation.
2. Right-click the **Server VM** → **Settings**.
3. Select **Network Adapter**.
4. Under **Network connection**, choose the **Host-only** option (*Host-only: A private network shared with the host*).
5. Tick **Connect at power on** if it is not ticked.
6. Click **OK**.
7. Repeat for the **Client VM**.

> [!IMPORTANT]
> **Turn off VMware's built-in DHCP service before continuing.** By default VMware runs its own DHCP service on VMnet1, which may answer clients before your server does. Go to **Edit → Virtual Network Editor** (click **Change Settings** if prompted for administrator rights), select **VMnet1**, and **clear** the checkbox *Use local DHCP service to distribute IP addresses to VMs*. Click **Apply → OK**.

**Verify:** both VMs are on the same Host-Only network (normally VMnet1), and the network icon in the VMware status bar shows connected.

**Common problems**

| Problem | Cause | Fix |
|---------|-------|-----|
| No Host-only option | Old VMware version | Update, or use **Custom: Specific virtual network** |
| VMs on different VMnets | Wrong selection | Put both on the same VMnet (for example VMnet1) |
| Network will not connect | VMware services stopped | Check the VMware DHCP/NAT/Authorization services |

---

## Step 2: Choose the lab network

**Goal:** decide the network design before touching anything.

**Why:** without a plan you do not know which addresses to give out or which range to configure.

| Item | Value |
|------|-------|
| Network | `192.168.10.0/24` |
| Subnet mask | `255.255.255.0` |
| Server (DHCP) | `192.168.10.1` (static) |
| DHCP range | `192.168.10.100` to `192.168.10.200` |
| Gateway | `192.168.10.1` (the server itself, because the lab has no router) |
| DNS | `192.168.10.1` (the server) |

![Lab IP plan](./images/lab-ip-plan.svg)

**Why these choices?**

- `192.168.10.0/24` is a **private range** (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), common in real networks, with 254 usable addresses.
- The server uses `.1` by convention (easy to remember).
- The range starts at `.100` so `.1` to `.99` stay free for static devices. This is standard practice.

> [!TIP]
> In this lab the gateway and DNS are the server because there is no separate router. On a real network they are normally separate devices.

---

## Step 3: Choose the server IP

**Why:** a DHCP server cannot get its own address from DHCP. It needs a **static** address so clients always know where it is.

```text
IP:       192.168.10.1
Subnet:   255.255.255.0
Gateway:  (empty - there is no router in the lab)
DNS:      127.0.0.1 (the server itself)
```

- **Why no gateway?** The lab has no internet and no router.
- **Why DNS `127.0.0.1`?** It means "this machine". It is correct if you later install the DNS role. Otherwise you could enter `8.8.8.8`, although it will not work in an isolated lab.

---

## Step 4: Set a static IP on the server

**Steps**

1. Power on the Server VM and sign in.
2. Press **Win + R**, type `ncpa.cpl`, press Enter.
3. Right-click the **Ethernet** adapter → **Properties**.
4. Select **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**.
5. Enter:

```text
(o) Use the following IP address:
    IP address:       192.168.10.1
    Subnet mask:      255.255.255.0
    Default gateway:  (leave empty)

(o) Use the following DNS server addresses:
    Preferred DNS:    127.0.0.1
    Alternate DNS:    (leave empty)
```

6. Click **OK → OK**.

**Verify** in CMD:

```cmd
ipconfig /all
```

Expected:

```text
IPv4 Address. . . . . . . . . . . : 192.168.10.1
Subnet Mask . . . . . . . . . . . : 255.255.255.0
Default Gateway . . . . . . . . . :
```

**Common problems**

| Problem | Cause | Fix |
|---------|-------|-----|
| Cannot set the IP | Adapter disabled | Enable the Ethernet adapter in `ncpa.cpl` |
| Duplicate-address warning | Address in use | Try another address, for example `192.168.10.2` |
| Setting lost after reboot | Windows glitch | Configure it again |

---

## Step 5: Install the DHCP Server role

**Why:** Windows Server does not include the DHCP role by default.

**Steps**

1. Open **Server Manager**.
2. Click **Manage** (top right) → **Add Roles and Features**.
3. Click **Next** until you reach **Server Roles**.
4. Tick **DHCP Server**.
5. In the pop-up about required features, click **Add Features**.
6. Click **Next** until **Confirmation**.
7. Tick **Restart the destination server automatically if required**.
8. Click **Install** and wait 1-2 minutes.
9. When it finishes, click **Complete DHCP configuration**.
10. In the wizard, click **Next → Commit → Close**.

> [!NOTE]
> In a domain environment this step also authorizes the DHCP server in Active Directory. In a standalone lab server the authorization step does not apply.

**Verify** (any one of these):

- **Tools → DHCP** opens the DHCP console.
- **DHCP** appears in the left menu of Server Manager.
- In PowerShell:

```powershell
Get-WindowsFeature DHCP
```

The install state should be `Installed`.

**Common problems**

| Problem | Cause | Fix |
|---------|-------|-----|
| Installation fails | Server IP is not static | Recheck Step 4 |
| DHCP will not open | Service stopped | `services.msc` → DHCP Server → Start |
| Access denied | Not an administrator | Sign in as Administrator |

---

## Step 6: Create the DHCP scope

**Goal:** define the range of IP addresses the server will distribute.

**Steps**

1. **Server Manager → Tools → DHCP**.
2. Expand the tree:

```text
<server-name>
  └── IPv4
```

3. Right-click **IPv4** → **New Scope**. The wizard pages are:

| Page | Setting |
|------|---------|
| Welcome | **Next** |
| Scope Name | Name: `Laboratory Scope`, Description: `DHCP scope for lab network` |
| IP Address Range | Start `192.168.10.100`, End `192.168.10.200`, Length `24`, Mask `255.255.255.0` |
| Add Exclusions and Delay | Leave empty (we add one later in the experiments) |
| Lease Duration | 1 day (or 8 hours if you want it shorter) |
| Configure DHCP Options | **Yes, I want to configure these options now** |
| Router (Default Gateway) | Add `192.168.10.1` |
| Domain Name and DNS Servers | Parent domain empty, add `192.168.10.1` |
| WINS Servers | Leave empty, **Next** |
| Activate Scope | **Yes, I want to activate this scope now** |
| Finish | **Finish** |

**Verify** in the DHCP console:

```text
IPv4
  └── Scope [192.168.10.0] Laboratory Scope
        ├── Address Pool
        │     └── 192.168.10.100 - 192.168.10.200
        ├── Address Leases
        ├── Reservations
        ├── Scope Options
        │     ├── 003 Router          → 192.168.10.1
        │     └── 006 DNS Servers     → 192.168.10.1
        └── ...
```

The scope icon should be **green** (active).

**Common problems**

| Problem | Cause | Fix |
|---------|-------|-----|
| Scope cannot be created | Server has no static IP | Recheck Step 4 |
| Range overlaps the server | Range includes `.1` | Start the range at `.100` |
| Scope inactive | Activate not selected | Right-click → **Activate** |

---

## Step 7: Define the IP range

You defined `192.168.10.100` - `192.168.10.200` in the wizard.

- **From `.100`:** `.1` to `.99` stay reserved for static devices (servers, printers, routers).
- **To `.200`:** `.201` to `.254` are kept for growth and special devices.
- **101 addresses** are plenty for a lab.

---

## Step 8: Exclusions

**Goal:** mark addresses that DHCP must **never** hand out.

**Why:** if a printer uses static `192.168.10.150` and that address is inside the DHCP range, DHCP may give it to a client and cause a **conflict**.

The lab has no static devices yet, so no exclusion is needed now. We add one in [Experiment 4](./04-hands-on-experiments.md).

To add one anyway:

1. In the DHCP console right-click **Address Pool** → **New Exclusion Range**.
2. Enter start and end, for example both `192.168.10.150`.
3. Click **Add → Close**.

---

## Step 9: Lease duration

**Goal:** control how long a client keeps an address.

- **Short lease** (for example 8 hours): addresses return quickly. Good for crowded, changing networks such as cafés and public Wi-Fi.
- **Long lease** (for example 7 days): good for stable networks such as offices.

For the lab, 1 day or 8 hours is fine.

To change it: right-click the scope → **Properties** → **General** tab → **Lease duration for DHCP clients**.

---

## Step 10: Default gateway option

**Goal:** tell clients where the exit from the network is. Without a gateway, clients can only reach their own subnet.

In the lab the gateway is `192.168.10.1` (the server). You already set it on wizard page "Router".

**Check:** open **Scope Options**; you should see:

```text
003 Router     → 192.168.10.1
```

---

## Step 11: DNS server option

**Goal:** tell clients where the name server is. Without DNS, clients can use IP addresses but not names.

In the lab DNS is `192.168.10.1` (set on the wizard page "Domain Name and DNS Servers").

**Check:** in **Scope Options**:

```text
006 DNS Servers     → 192.168.10.1
```

---

## Step 12: Activate the scope

An inactive scope hands out nothing.

1. Right-click the scope.
2. If the menu shows **Activate**, the scope is inactive; click it.
3. If the menu shows **Deactivate**, it is already active; do nothing.

A **green** icon means active, **red** means inactive.

---

## Step 13: Verify the DHCP server

**From the DHCP console:** expand **IPv4** and confirm the scope is green.

**From PowerShell:**

```powershell
Get-Service DHCPServer
```

```text
Status   Name          DisplayName
------   ----          -----------
Running  DHCPServer    DHCP Server
```

**From CMD:**

```cmd
net start | findstr DHCP
```

You should see `DHCP Server` in the running services.

---

## Step 14: Set the client to obtain an IP automatically

**Why:** a client with a static address will not request one from DHCP.

1. Power on the Client VM and sign in.
2. Press **Win + R**, type `ncpa.cpl`, press Enter.
3. Right-click **Ethernet** → **Properties**.
4. Select **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**.
5. Select:

```text
(o) Obtain an IP address automatically
(o) Obtain DNS server address automatically
```

6. Click **OK → OK**.

**Verify** with `ipconfig /all`:

```text
DHCP Enabled. . . . . . . . . . . : Yes
Autoconfiguration Enabled . . . . : Yes
```

---

## Step 15: Get an IP from DHCP

**Why:** the client may still hold an old address. Force it to ask again.

Open **CMD as Administrator** on the client:

```cmd
ipconfig /release
ipconfig /renew
```

| Command | What it does |
|---------|--------------|
| `ipconfig /release` | Gives the current address back |
| `ipconfig /renew` | Requests an address from DHCP |

**What happened behind the scenes**

1. `release`: the client sent a **DHCP Release** message and the server returned the address to the pool.
2. `renew`: the client sent a **Discover** (DORA step 1).
3. The server sent an **Offer**.
4. The client sent a **Request**.
5. The server sent an **ACK**.
6. The client received its new address.

---

## Step 16: Verify the received settings

```cmd
ipconfig /all
```

Expected:

```text
IPv4 Address. . . . . . . . . . . : 192.168.10.100  (any address from .100 to .200)
Subnet Mask . . . . . . . . . . . : 255.255.255.0
Default Gateway . . . . . . . . . : 192.168.10.1
DHCP Server . . . . . . . . . . . : 192.168.10.1
DNS Servers . . . . . . . . . . . : 192.168.10.1
Lease Obtained. . . . . . . . . . : <date and time>
Lease Expires . . . . . . . . . . : <date and time>
```

**Success looks like:**

- The IP is between `192.168.10.100` and `192.168.10.200`.
- `DHCP Server` is `192.168.10.1`.
- `DHCP Enabled` is `Yes`.
- `Lease Obtained` and `Lease Expires` are filled in.

If the address starts with `169.254.`, the client found no DHCP server. See [Troubleshooting](./05-troubleshooting.md).

---

## Step 17: Test connectivity

From the **client**:

```cmd
ping 192.168.10.1
```

Expected:

```text
Reply from 192.168.10.1: bytes=32 time<1ms TTL=128
Reply from 192.168.10.1: bytes=32 time<1ms TTL=128
Reply from 192.168.10.1: bytes=32 time<1ms TTL=128
Reply from 192.168.10.1: bytes=32 time<1ms TTL=128
```

From the **server**, ping the client's address (take it from the client's `ipconfig`):

```cmd
ping 192.168.10.100
```

`Reply from ...` means communication works.

| Result | Typical cause |
|--------|---------------|
| `Request timed out` | Windows Firewall blocking ICMP |
| `Destination host unreachable` | Network not connected |
| `General failure` | Network adapter problem |

---

## Step 18: Check the lease on the server

Look at the server's side: which client received which address.

**DHCP console:** **DHCP → IPv4 → Scope → Address Leases**

```text
IP Address       Client Name      Lease Expires
192.168.10.100   CLIENT-PC        <date>
```

**PowerShell:**

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
```

You should see one row with the client's IP, its name and the expiry time.

---

## Result

If you got this far, you have:

- Configured VMware networking for an isolated lab.
- Built a server with a static IP.
- Installed the DHCP role and created a scope.
- Had a client obtain its address automatically.
- Confirmed connectivity and seen the lease on the server.

Next you will break and change things on purpose in the [experiments](./04-hands-on-experiments.md).

---

[Previous](./02-dhcp-concepts.md) | [Course index](./README.md) | [Next: Experiments](./04-hands-on-experiments.md)
