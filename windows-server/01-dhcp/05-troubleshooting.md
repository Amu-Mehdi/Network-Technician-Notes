# DHCP, Part 5: Troubleshooting

> Twelve real-world scenarios. For each: symptoms, likely causes, what to check, and the fix.

[Previous](./04-hands-on-experiments.md) | [Course index](./README.md) | [Next: Final project](./06-final-project.md)

## Contents

- [Start here: the troubleshooting order](#start-here-the-troubleshooting-order)
- [1. Client gets no IP](#1-client-gets-no-ip)
- [2. Client gets the wrong IP](#2-client-gets-the-wrong-ip)
- [3. Client gets 169.254.x.x](#3-client-gets-169254xx)
- [4. Client has an IP but no internet](#4-client-has-an-ip-but-no-internet)
- [5. Client cannot reach the server](#5-client-cannot-reach-the-server)
- [6. DHCP server is not running](#6-dhcp-server-is-not-running)
- [7. Scope is not active](#7-scope-is-not-active)
- [8. Address pool exhausted](#8-address-pool-exhausted)
- [9. Wrong DNS setting](#9-wrong-dns-setting)
- [10. Wrong gateway](#10-wrong-gateway)
- [11. VMware did not attach the network](#11-vmware-did-not-attach-the-network)
- [12. Multiple DHCP servers on the network](#12-multiple-dhcp-servers-on-the-network)
- [Golden checklist](#golden-checklist)
- [Summary](#summary)

---

## Start here: the troubleshooting order

Whenever a client does not get an address, work through these checks in order.

![Troubleshooting order](./images/troubleshooting-flow.svg)

---

## 1. Client gets no IP

**Symptoms**

- `ipconfig /renew` returns no address.
- The client shows `169.254.x.x`.
- `ipconfig` shows nothing useful.

**Likely causes**

1. DHCP server is off.
2. DHCP service is not running.
3. Scope is inactive.
4. Network cable/adapter is disconnected.
5. A firewall blocks the DHCP ports.
6. The client is on the wrong network.

**What to check**

1. Client: `ipconfig /all`. Is `DHCP Enabled` = `Yes`?
2. Client: `ping 192.168.10.1`. No reply means no connectivity.
3. Server: `Get-Service DHCPServer` should show `Running`.
4. DHCP console: scope is green and the pool has free addresses.

**Fix**

| If | Then |
|----|------|
| Service is stopped | `Start-Service DHCPServer` |
| Scope is inactive | Right-click → **Activate** |
| Cable connected but no network | Check the VMnet settings |
| Firewall blocking | Allow UDP ports 67 and 68 |
| Client on the wrong network | Attach the adapter to the correct VMnet |

---

## 2. Client gets the wrong IP

**Symptoms**

- The client gets an address, but from an **unexpected range**: for example `192.168.1.x` instead of `192.168.10.x`.

**Likely causes**

1. Another DHCP server is on the network.
2. VMware's own DHCP service is active.
3. The client is on a different network.

**What to check**

1. Client: `ipconfig /all`. Which address is listed as `DHCP Server`?
2. Server: is the client's address under **Address Leases**?
3. VMware: **Edit → Virtual Network Editor**. Is the local DHCP service enabled on VMnet1?

**Fix**

| If | Then |
|----|------|
| VMware DHCP is on | Clear **Use local DHCP service** in Virtual Network Editor |
| Another DHCP server exists | Turn it off or isolate it |
| Wrong network | Attach the adapter to the correct VMnet |

> [!TIP]
> This is one of the **most common lab problems**. VMware runs a small DHCP service on VMnet1 by default; switch it off.

---

## 3. Client gets 169.254.x.x

**Symptoms**

- Address starts with `169.254.`.
- Gateway and DHCP server fields are empty.

**Likely causes**

1. No DHCP server could be found.
2. The client could not communicate with the server.
3. Scope is inactive or exhausted.
4. Cable or adapter problem.

**What to check**

1. Client: `ipconfig /all`. An `Autoconfiguration IPv4 Address` line means APIPA is active.
2. Client: `ping 192.168.10.1`.
3. Server: service `Running`, scope active, free addresses available.

**Fix**

| If | Then |
|----|------|
| Server does not answer ping | Check the network |
| Server answers ping but DHCP does not respond | Check the service and scope |
| Scope exhausted | Extend the range |
| Still failing | `ipconfig /release` and `ipconfig /renew` |

> [!TIP]
> APIPA (`169.254.x.x`) means *"I found no DHCP server, so I made up an address."* It only works for local communication.

---

## 4. Client has an IP but no internet

**Symptoms**

- The client has `192.168.10.x`.
- It cannot reach the internet and `ping 8.8.8.8` fails.

**Likely causes**

1. **Wrong gateway**.
2. **Wrong DNS**.
3. In the isolated lab there is **no gateway/internet** at all (this is normal).
4. The router has a problem.

**What to check**

1. `ipconfig /all`: are the gateway and DNS correct?
2. `ping 192.168.10.1`: if it answers, the internal network is fine.
3. `ping 8.8.8.8`: if it fails, the problem is the gateway or internet.
4. `ping google.com`: if `8.8.8.8` works but this fails, the problem is **DNS**.

**Fix**

| If | Then |
|----|------|
| Gateway empty or wrong | Fix `003 Router` in Scope Options |
| DNS wrong | Fix `006 DNS Servers` in Scope Options |
| No internet in the lab | Expected: the lab is isolated |
| `8.8.8.8` works, `google.com` does not | DNS problem |

---

## 5. Client cannot reach the server

**Symptoms**

- The client has an address but `ping 192.168.10.1` fails.

**Likely causes**

1. Windows Firewall is blocking ICMP.
2. The machines are on different networks.
3. A network adapter problem.
4. Network services are stopped.

**What to check**

1. Client: `arp -a`. If the server is not listed, the link is not working.
2. Client: `ping 192.168.10.1`.
3. Server: `ping 192.168.10.100`.
4. On both machines:

```powershell
Get-NetFirewallProfile | Select Name, Enabled
```

If `Enabled` is `True`, the firewall may be the cause.

**Fix**

| If | Then |
|----|------|
| Firewall blocks it | Temporarily disable it, or allow ICMP |
| Different networks | Attach both VMs to the same VMnet |
| Adapter problem | Disable and re-enable the adapter |

Temporarily disable the firewall (lab use only):

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

> [!WARNING]
> Turn the firewall back on after testing. A cleaner fix is to allow ICMP echo requests with a firewall rule.

---

## 6. DHCP server is not running

**Symptoms**

- The service shows red or stopped in the DHCP console.
- Clients get no addresses.

**Likely causes**

1. The service was stopped.
2. The service crashed.
3. The server rebooted and the service did not start.

**What to check and do**

```powershell
Get-Service DHCPServer
```

If `Stopped`:

```powershell
Start-Service DHCPServer
```

If `Running` but not working:

```powershell
Restart-Service DHCPServer
```

**Fix**

| If | Then |
|----|------|
| Service stopped | `Start-Service DHCPServer` |
| Service misbehaving | `Restart-Service DHCPServer` |
| Still failing | Reboot the server |
| Broken installation | Remove and reinstall the role |

Make the service start automatically:

```powershell
Set-Service DHCPServer -StartupType Automatic
```

---

## 7. Scope is not active

**Symptoms**

- The scope icon is **red**.
- Clients get no addresses.

**Likely causes**

1. The scope was deactivated manually.
2. It was created but never activated.

**Fix**

- **GUI:** right-click the scope → **Activate**.
- **PowerShell:**

```powershell
Get-DhcpServerv4Scope | Select ScopeId, Name, State
```

If `State` is `Inactive`:

```powershell
Set-DhcpServerv4Scope -ScopeId 192.168.10.0 -State Active
```

---

## 8. Address pool exhausted

**Symptoms**

- Clients cannot get addresses.
- **Address Leases** is full of active leases.
- The pool has no free addresses.

**Likely causes**

1. More clients than addresses.
2. Leases for clients that **left** have not expired.
3. Lease duration is too long.

**What to check**

```powershell
Get-DhcpServerv4ScopeStatistics -ScopeId 192.168.10.0
```

```text
ScopeId         : 192.168.10.0
Free            : 0          <- free addresses
InUse           : 101        <- addresses in use
PercentageInUse : 100
```

**Fix**

| If | Then |
|----|------|
| `Free: 0` | Extend the range (for example `.200` to `.250`) |
| Lease duration too long | Shorten it (for example 1 day to 8 hours) |
| Stale leases | Right-click → **Delete** |
| Too many clients | Create another scope |

Extend the range:

```powershell
Set-DhcpServerv4Scope -ScopeId 192.168.10.0 -EndRange 192.168.10.250
```

---

## 9. Wrong DNS setting

**Symptoms**

- `ping 8.8.8.8` works but `ping google.com` does not.
- Sites open by IP but not by name.

**Likely causes**

1. Wrong DNS server in Scope Options.
2. DNS server unreachable.
3. DNS server not working.

**What to check** (on the client)

```cmd
ipconfig /all
nslookup google.com
nslookup google.com 8.8.8.8
```

If it works when you ask `8.8.8.8` directly, the DNS server given by DHCP is the problem.

**Fix**

- DHCP console → **Scope Options** → right-click → **Configure Options** → `006 DNS Servers` → enter the correct address.
- For a quick client-side test only:

```cmd
netsh interface ip set dns "Ethernet0" static 8.8.8.8
```

---

## 10. Wrong gateway

**Symptoms**

- The client has an address but cannot leave the local network.
- `ping 8.8.8.8` fails but `ping 192.168.10.1` works.

**Likely causes**

1. `003 Router` in Scope Options is wrong.
2. The gateway does not exist or is off.
3. The gateway is on a different network.

**What to check** (on the client)

```cmd
ipconfig /all
ping 192.168.10.1
route print
```

**Fix**

- DHCP console → **Scope Options** → set `003 Router` to the correct gateway address.
- In the isolated lab there is no real gateway, so this symptom is expected.

---

## 11. VMware did not attach the network

**Symptoms**

- The VMs boot but have no network.
- The network icon shows disconnected.
- `ipconfig` shows `Media disconnected`.

**Likely causes**

1. The adapter is not attached to the right VMnet.
2. **Connected** is not ticked.
3. VMware services are stopped.

**What to check**

1. VM → **Settings → Network Adapter**: **Connected** and **Connect at power on** ticked.
2. **Edit → Virtual Network Editor**: VMnet1 should be **Host-only**.
3. In `services.msc` these should be `Running`:
   - `VMware DHCP Service`
   - `VMware NAT Service`
   - `VMware Authorization Service`
   - `VMware Workstation Server`

**Fix**

| If | Then |
|----|------|
| **Connected** not ticked | Tick it |
| Service stopped | Start it |
| Virtual Network Editor misbehaves | **Restore Defaults** |
| Still failing | Restart VMware |

---

## 12. Multiple DHCP servers on the network

**Symptoms**

- Clients receive **different** addresses from **different** ranges.
- Some come from one server, some from another.
- You do not know which server answered.

**Likely causes**

1. VMware's own DHCP service.
2. Another DHCP server on the network.
3. A home or office router with DHCP enabled.

**What to check**

- Client: `ipconfig /all`, look at `DHCP Server`.
- VMware: **Edit → Virtual Network Editor**, is **Use local DHCP service** ticked?
- Real network: `arp -a` and look for unknown devices.

**Fix**

| If | Then |
|----|------|
| VMware DHCP is on | Clear the checkbox |
| Another DHCP server exists | Turn it off or isolate it |
| You do not know who answers | Check `DHCP Server` in the client's `ipconfig /all` |

To disable VMware's DHCP: Virtual Network Editor → select VMnet1 → clear **Use local DHCP service to distribute IP addresses to VMs** → **Apply → OK**.

> [!TIP]
> This is **very common** in VMware labs. Always check it.

---

## Golden checklist

When a client gets no address, check these in order:

```text
[ ] 1.  Is the cable/adapter connected?
[ ] 2.  Is the client set to obtain an address automatically?
[ ] 3.  Is the client on the correct VMnet?
[ ] 4.  Is the server on and using a static IP?
[ ] 5.  Can the server and client ping each other?
[ ] 6.  Is the DHCPServer service Running?
[ ] 7.  Is the scope active (green)?
[ ] 8.  Are there free addresses in the pool?
[ ] 9.  Is VMware's DHCP service disabled?
[ ] 10. Is a firewall blocking the traffic?
[ ] 11. Is the address 169.254.x.x? (APIPA)
[ ] 12. Test again with ipconfig /release and /renew
```

## Summary

| # | Scenario | Key fix |
|---|----------|---------|
| 1 | No IP | Check the service and scope |
| 2 | Wrong IP | Turn off VMware DHCP |
| 3 | `169.254.x.x` | No DHCP server was found |
| 4 | No internet | Check gateway and DNS |
| 5 | Cannot reach the server | Check firewall and network |
| 6 | Server not running | Start the service |
| 7 | Scope inactive | Activate it |
| 8 | Pool exhausted | Extend the range |
| 9 | Wrong DNS | Fix Scope Options (006) |
| 10 | Wrong gateway | Fix `003 Router` |
| 11 | VMware network missing | Check adapter and services |
| 12 | Multiple servers | Turn off VMware DHCP |

---

[Previous](./04-hands-on-experiments.md) | [Course index](./README.md) | [Next: Final project](./06-final-project.md)
