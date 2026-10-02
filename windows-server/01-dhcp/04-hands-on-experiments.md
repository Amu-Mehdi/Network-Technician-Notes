# DHCP, Part 4: Hands-On Experiments

> Eight experiments that show how leases, exclusions, reservations and APIPA behave in practice.

[Previous](./03-lab-setup.md) | [Course index](./README.md) | [Next: Troubleshooting](./05-troubleshooting.md)

## Contents

- [Experiment 1: Get an IP and read the full output](#experiment-1-get-an-ip-and-read-the-full-output)
- [Experiment 2: See the lease on the server](#experiment-2-see-the-lease-on-the-server)
- [Experiment 3: Deactivate the scope and watch APIPA](#experiment-3-deactivate-the-scope-and-watch-apipa)
- [Experiment 4: Create an exclusion](#experiment-4-create-an-exclusion)
- [Experiment 5: Create a reservation](#experiment-5-create-a-reservation)
- [Experiment 6: Lease and renewal](#experiment-6-lease-and-renewal)
- [Experiment 7: Essential troubleshooting commands](#experiment-7-essential-troubleshooting-commands)
- [Experiment 8: Force a new lease](#experiment-8-force-a-new-lease)
- [Summary](#summary)

The lab from [Part 3](./03-lab-setup.md) must be working before you start.

---

## Experiment 1: Get an IP and read the full output

**Goal:** see exactly what the client received.

**Why:** the first step of any troubleshooting job is to know the current state.

**On the client**, open CMD:

```cmd
ipconfig /all
```

Annotated output:

```text
Windows IP Configuration

   Host Name . . . . . . . . . . . . : CLIENT-PC
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No

Ethernet adapter Ethernet0:

   Description . . . . . . . . . . . : Intel(R) 82574L Gigabit Network Connection
   Physical Address. . . . . . . . . : 00-0C-29-AB-CD-EF   <- MAC address
   DHCP Enabled. . . . . . . . . . . : Yes                  <- DHCP is on
   Autoconfiguration Enabled . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 192.168.10.100       <- leased address
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Lease Obtained. . . . . . . . . . : <date and time>      <- when it was given
   Lease Expires . . . . . . . . . . : <date and time>      <- when it ends
   Default Gateway . . . . . . . . . : 192.168.10.1
   DHCP Server . . . . . . . . . . . : 192.168.10.1         <- who answered
   DNS Servers . . . . . . . . . . . : 192.168.10.1
```

**Success looks like**

- `DHCP Enabled: Yes`
- `IPv4 Address` between `192.168.10.100` and `192.168.10.200`
- `DHCP Server: 192.168.10.1`
- `Lease Obtained` and `Lease Expires` are populated

**Behind the scenes:** the client broadcast a Discover, the server sent an Offer, the client sent a Request, the server sent an ACK, and the client stored all the options.

---

## Experiment 2: See the lease on the server

**Goal:** view the DHCP service from the server side.

**Why:** a network engineer must be able to see *who has which address*.

**Option 1: DHCP console.** Go to **DHCP → IPv4 → Scope [192.168.10.0] Laboratory Scope → Address Leases**:

```text
IP Address       Client Name      Lease Expires
192.168.10.100   CLIENT-PC        <date and time>
```

**Option 2: PowerShell.**

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
```

```text
IPAddress      ScopeId        ClientId            HostName    AddressState    LeaseExpiryTime
---------      -------        --------            --------    ------------    ---------------
192.168.10.100 192.168.10.0   00-0c-29-ab-cd-ef   CLIENT-PC   Active          <date and time>
```

| Column | Meaning |
|--------|---------|
| `IPAddress` | Address leased |
| `ScopeId` | Which scope |
| `ClientId` | The client's MAC address |
| `HostName` | The client's name |
| `AddressState` | Active, Expired, ... |
| `LeaseExpiryTime` | When the lease ends |

**Success looks like:** a row with your client's name, `AddressState` = `Active` and an expiry time in the future.

---

## Experiment 3: Deactivate the scope and watch APIPA

**Goal:** break DHCP on purpose and see how the client reacts.

**Why:** knowing the symptoms makes real faults quicker to diagnose.

**On the server**

1. In the DHCP console right-click the scope → **Deactivate**.
2. The scope icon turns **red**.

**On the client**

```cmd
ipconfig /release
ipconfig /renew
```

Wait a few seconds (the client retries several times), then:

```cmd
ipconfig
```

Expected:

```text
Ethernet adapter Ethernet0:

   Autoconfiguration IPv4 Address. . : 169.254.123.45
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . :
```

**What happened**

1. The client sent a Discover.
2. The server did not answer (scope inactive).
3. The client retried a few times.
4. Windows enabled **APIPA** and gave itself an address from `169.254.0.0/16`.
5. That address works only on the local link.

**How to recognize it:** the address starts with `169.254.`, and the default gateway and DHCP server fields are empty.

> [!TIP]
> Whenever you see `169.254.x.x`, the client could not obtain an address from a DHCP server.

**Restore**

- Server: right-click the scope → **Activate**.
- Client: `ipconfig /release` then `ipconfig /renew`. You should get `192.168.10.x` again.

---

## Experiment 4: Create an exclusion

**Goal:** remove one address from the pool and confirm clients never receive it.

**Why:** static devices (printers, servers, routers) must not have their addresses given to clients.

**On the server**

1. In the DHCP console open **Address Pool**.
2. Right-click → **New Exclusion Range**.
3. Enter:

```text
Start IP address:   192.168.10.150
End IP address:     192.168.10.150
```

4. Click **Add → Close**.

The Address Pool now shows:

```text
192.168.10.100 - 192.168.10.149
192.168.10.150 - 192.168.10.150   (Exclusion)
192.168.10.151 - 192.168.10.200
```

**On the client**, repeat a few times:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /release
ipconfig /renew
```

**Result:** the client never receives `192.168.10.150`.

**Behind the scenes:** the server keeps an internal list of distributable addresses; `.150` is not on it, so every request is answered from the remaining addresses.

**Success looks like:** the exclusion row appears in the Address Pool and the client never gets `.150`.

---

## Experiment 5: Create a reservation

**Goal:** make a specific client always receive the same address.

**Why:** printers, file servers and cameras need a fixed address, but you may prefer to manage it centrally rather than configure each device.

**Step 1: find the client's MAC address.** On the client:

```cmd
ipconfig /all
```

```text
Physical Address. . . . . . . . . : 00-0C-29-AB-CD-EF
```

**Step 2: create the reservation on the server.**

1. Expand **Scope → Reservations**.
2. Right-click → **New Reservation**.
3. Enter:

```text
Reservation name:   CLIENT-PC
IP address:         192.168.10.120
MAC address:        00-0C-29-AB-CD-EF
Description:        Reserved for CLIENT-PC
Supported types:    Both
```

4. Click **Add → Close**.

**Step 3: test on the client.**

```cmd
ipconfig /release
ipconfig /renew
ipconfig
```

**Result:** the client **always** receives `192.168.10.120`, however often you release and renew.

**Behind the scenes**

1. The client sends a Discover containing its MAC address.
2. The server checks whether that MAC has a reservation.
3. It does, so the server offers the reserved address.

**Success looks like:** a new row under **Reservations**, the client always gets `.120`, and **Address Leases** shows `.120` as a reservation.

To undo: right-click the reservation → **Delete**.

---

## Experiment 6: Lease and renewal

**Goal:** see how a client renews its lease.

**Why:** a lease is like a rental contract. When it runs out, the client must renew or lose the address.

![Lease lifecycle](./images/lease-lifecycle.svg)

**Step 1: view the current lease** on the client:

```cmd
ipconfig /all
```

```text
Lease Obtained. . . . . . . . . . : <time A>
Lease Expires . . . . . . . . . . : <time A + lease duration>
```

**Step 2: renew:**

```cmd
ipconfig /renew
```

**Step 3: check again.** `Lease Obtained` and `Lease Expires` should show **new times**.

**Behind the scenes**

1. The client sends a Request to the server asking to extend the lease.
2. The server answers with an ACK and a fresh lease time.

> [!NOTE]
> A client normally starts renewing on its own when **50%** of the lease has passed (T1). For a 24-hour lease that is after 12 hours. If the original server does not respond, the client tries any server at **87.5%** (T2).

**Success looks like:** new lease times and an **unchanged** IP address.

---

## Experiment 7: Essential troubleshooting commands

| # | Command | Purpose |
|---|---------|---------|
| 1 | `ipconfig` | Basic info: IP, mask, gateway |
| 2 | `ipconfig /all` | Full info: MAC, DHCP server, DNS, lease times |
| 3 | `ipconfig /release` | Give up the current address |
| 4 | `ipconfig /renew` | Request an address from DHCP |
| 5 | `ping 192.168.10.1` | Test reachability of a device |
| 6 | `nslookup google.com` | Test DNS resolution (needs a DNS server) |
| 7 | `arp -a` | Show known IP-to-MAC mappings |
| 8 | `netstat -an` | Show current network connections |

```cmd
ipconfig
ipconfig /all
ipconfig /release
ipconfig /renew
ping 192.168.10.1
nslookup google.com
arp -a
netstat -an
```

These are your everyday tools as a network technician.

---

## Experiment 8: Force a new lease

**Goal:** make the client request an address again, for example to test a scope change.

```cmd
ipconfig /release
ipconfig /renew
```

Repeat in one line if you like:

```cmd
ipconfig /release && ipconfig /renew
```

**Result:** you may get a different address (for example `.100` then `.101`), or the same one.

> [!NOTE]
> Windows clients often get the same address back because the server remembers a client's previous lease for a while. Do not expect the address to change every time.

---

## Summary

| # | Experiment | What you learned |
|---|------------|------------------|
| 1 | Get an IP | Reading complete client information |
| 2 | See the lease | Managing from the server side |
| 3 | Deactivate scope | APIPA (`169.254.x.x`) |
| 4 | Exclusion | Removing addresses from the pool |
| 5 | Reservation | Fixed address for a specific client |
| 6 | Lease and renew | The lease lifecycle |
| 7 | Commands | Daily troubleshooting tools |
| 8 | Force new lease | Requesting a fresh address |

Next: [troubleshooting 12 real-world scenarios](./05-troubleshooting.md).

---

[Previous](./03-lab-setup.md) | [Course index](./README.md) | [Next: Troubleshooting](./05-troubleshooting.md)
