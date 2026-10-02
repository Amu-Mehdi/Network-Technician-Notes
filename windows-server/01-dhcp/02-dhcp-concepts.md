# DHCP, Part 2: DHCP Concepts

> Terminology, the DORA process, ports, broadcast and DHCP Relay.

[Previous](./01-networking-fundamentals.md) | [Course index](./README.md) | [Next: Lab setup](./03-lab-setup.md)

## Contents

- [1. What is DHCP?](#1-what-is-dhcp)
- [2. Why do we need DHCP?](#2-why-do-we-need-dhcp)
- [3. What happens without DHCP?](#3-what-happens-without-dhcp)
- [4. What does a DHCP server do?](#4-what-does-a-dhcp-server-do)
- [5. Static IP vs DHCP](#5-static-ip-vs-dhcp)
- [6. What DHCP delivers to a client](#6-what-dhcp-delivers-to-a-client)
- [7. Key terminology](#7-key-terminology)
- [8. The DORA process](#8-the-dora-process)
- [9. Client and server communication](#9-client-and-server-communication)
- [10. DHCP ports](#10-dhcp-ports)
- [11. Broadcast in DHCP](#11-broadcast-in-dhcp)
- [12. DHCP Relay](#12-dhcp-relay)
- [Summary](#summary)

---

## 1. What is DHCP?

**DHCP** stands for **Dynamic Host Configuration Protocol**.

| Word | Meaning |
|------|---------|
| **Dynamic** | Automatic. The address is not fixed and may change over time. |
| **Host** | Any device: PC, phone, laptop, printer. |
| **Configuration** | IP address, subnet mask, gateway, DNS and more. |
| **Protocol** | A set of rules both sides follow to communicate. |

DHCP is a protocol that **automatically delivers network settings** to devices.

> [!NOTE]
> Your home router contains a small DHCP server. When your phone joins the Wi-Fi, that is what gives it an address. You already use DHCP every day.

## 2. Why do we need DHCP?

1. **Saves time.** Configuring 100 computers by hand can take days. With DHCP it takes minutes.
2. **Reduces human error.** Typing addresses by hand can give two computers the same IP (an **IP conflict**) or wrong values. DHCP tracks which address is in use and makes duplicates very unlikely, although a manually configured device can still cause one.
3. **Easier management.** A new employee plugs in a cable and the PC configures itself.

## 3. What happens without DHCP?

**Scenario 1: static IPs.** You must enter IP, mask, gateway and DNS on every device. One typo breaks connectivity, and duplicate addresses cause IP conflicts.

**Scenario 2: do nothing.** Windows enables **APIPA** (Automatic Private IP Addressing) and assigns itself an address in `169.254.x.x`. That address works **only on the local network** and has no internet access. It means *"no DHCP server was found"*.

**Scenario 3: a guest connects.** Without DHCP they cannot connect unless you configure their phone by hand.

> [!TIP]
> If you see an address starting with `169.254.`, the client failed to get an address from a DHCP server (APIPA is active).

## 4. What does a DHCP server do?

1. Leases an **IP address** to the client.
2. Provides the **subnet mask**.
3. Provides the **default gateway**.
4. Provides the **DNS servers**.
5. Sets the **lease time** (how long the client keeps the address).
6. Takes the address back when the lease ends or the client releases it.
7. Reuses it for another client.

## 5. Static IP vs DHCP

| | Static IP | DHCP |
|---|---|---|
| **Who assigns it?** | You | DHCP server |
| **Time required** | Per device | Automatic |
| **Human error** | High | Low |
| **Best for** | Servers, printers, routers | Laptops, PCs, phones |
| **Changing an address** | Manual | Automatic |
| **IP conflicts** | Possible | Very unlikely |

Servers normally use **static** addresses because they must always be reachable at the same place. Clients use **DHCP**.

## 6. What DHCP delivers to a client

| # | Setting | Example |
|---|---------|---------|
| 1 | IP address | `192.168.1.50` |
| 2 | Subnet mask | `255.255.255.0` |
| 3 | Default gateway | `192.168.1.1` |
| 4 | DNS server | `8.8.8.8` |
| 5 | Lease time | 8 hours, 1 day |
| 6 | Domain name (optional) | `company.local` |
| 7 | NTP server (optional) | time synchronization |
| 8 | WINS server (legacy) | old Windows networks |

The first four matter most: **IP, mask, gateway, DNS**.

## 7. Key terminology

**DHCP Server.** The machine that hands out addresses, for example Windows Server with the DHCP role.

**DHCP Client.** Any device that requests an address: laptop, phone, PC.

**Scope.** The range of addresses the server may hand out, for example `192.168.1.100` to `192.168.1.200`. Think of it as a warehouse of addresses the server draws from.

**Address Pool.** The addresses inside a scope that are available for distribution. In practice often used as a synonym for the scope range.

**Lease.** The time an address is *loaned* to a client, for example 8 hours or 7 days. Like a rental contract: while it is valid the address is yours; at the end you renew it or the server takes it back.

**Reservation.** Makes a specific client **always receive the same IP**, identified by its **MAC address**. The device is still a DHCP client; the server holds the mapping.

| Static IP | Reservation |
|-----------|-------------|
| Configured on the device | Configured on the DHCP server |
| Device is not a DHCP client | Device stays a DHCP client |

**Exclusion.** Addresses that are **inside the scope range but must not be handed out**, usually because a device uses them statically. Example: a server at `192.168.1.10` while the scope is `.1` to `.200`; exclude `.1` to `.20` so DHCP never gives that address away.

## 8. The DORA process

**DORA** is the four-step exchange every DHCP client performs to get an address:

```text
D: Discover
O: Offer
R: Request
A: ACK (Acknowledgment)
```

![DORA process](./images/dora-process.svg)

Think of a customer entering a restaurant.

### Step 1: Discover

*The customer enters and asks loudly: "Is anyone here to serve me?"*

The client has no IP yet, so it **broadcasts** to the whole network:

```text
Source:      0.0.0.0 (port 68)
Destination: 255.255.255.255 (port 67)
Message:     "I need a DHCP server."
```

It uses broadcast because it cannot address a server it does not know.

### Step 2: Offer

*The waiter replies: "I have table 50 for you."*

Every DHCP server that heard the Discover answers with an **Offer**:

```text
"I can give you 192.168.1.50"   (from server 192.168.1.1)
```

If several servers exist, each sends an offer. The client usually accepts the **first** one.

### Step 3: Request

*The customer says: "Yes, I'll take table 50."*

The client broadcasts a **Request** naming the server and address it accepted. Because it is a broadcast, other servers see that their offers were not chosen and release them.

### Step 4: ACK

*The waiter hands over the key: "Here you go, you have it for 8 hours."*

The server sends an **ACK** confirming the address and all options:

```text
"192.168.1.50 is yours. Mask 255.255.255.0, gateway 192.168.1.1,
 DNS 8.8.8.8, lease 8 hours."
```

From this moment the client officially has an IP address.

### DORA summary

| Step | Message | Sent by | Destination | Meaning |
|------|---------|---------|-------------|---------|
| 1 | Discover | Client | Broadcast | "Where is a DHCP server?" |
| 2 | Offer | Server | Broadcast/Unicast | "I have an address for you." |
| 3 | Request | Client | Broadcast | "I want this address." |
| 4 | ACK | Server | Broadcast/Unicast | "It is yours." |

> [!NOTE]
> If a server sends an Offer but the client never sends a Request (for example it was switched off), the server keeps the address reserved for a short time and then frees it. Do not confuse this with **DHCPDECLINE**, which a client sends when it detects, after receiving an address, that the address is already in use.

## 9. Client and server communication

| Client | Direction | Server |
|--------|-----------|--------|
| `0.0.0.0` | 1. Discover (broadcast) → | `192.168.1.1` |
| | ← 2. Offer | |
| | 3. Request (broadcast) → | |
| | ← 4. ACK | |
| now has `192.168.1.50` | | |

## 10. DHCP ports

DHCP uses two **UDP** ports:

| Port | Used by | Purpose |
|------|---------|---------|
| **UDP 67** | DHCP server | Server listens here |
| **UDP 68** | DHCP client | Client listens here |

**Why UDP, not TCP?** UDP is lighter and needs no handshake. DHCP happens at boot time when speed matters, and DHCP itself retries if a message is lost. A client with no address also cannot easily open a TCP connection.

**Why two different ports?** Server and client can run on the same machine type without conflict. Servers always use 67, clients always use 68.

## 11. Broadcast in DHCP

A **broadcast** is a message to **all devices** on the local network.

DHCP uses it because at the Discover stage the client has no IP and does not know the server's address, so it must "shout" and let the server answer.

- `255.255.255.255` reaches everything on the local segment.
- `192.168.1.255` is the directed broadcast of `192.168.1.0/24`.

**Problem:** broadcasts create noise in large networks, and **routers do not forward them**. If the DHCP server is on a different network, the client's Discover never arrives. The solution is **DHCP Relay**.

## 12. DHCP Relay

### The problem

A company has three floors, each its own network:

- Floor 1: `192.168.1.0/24` (the DHCP server is here)
- Floor 2: `192.168.2.0/24`
- Floor 3: `192.168.3.0/24`

Clients on floors 2 and 3 cannot reach the server because broadcasts stop at the router.

### The solution

A **DHCP Relay Agent** (usually the router, or a Windows Server running the Relay role) receives the client's broadcast and forwards it as **unicast** to the DHCP server, then returns the server's answer to the client.

![DHCP Relay](./images/dhcp-relay.svg)

1. The client on floor 2 broadcasts a Discover.
2. The relay agent (router) receives it.
3. The relay forwards it to the DHCP server as unicast.
4. The server replies to the relay.
5. The relay delivers the reply to the client.

> [!NOTE]
> Windows Server can act as a relay agent, but on real networks it is normally configured on the router (for example `ip helper-address` on Cisco devices).

---

## Summary

1. **DHCP** automatically delivers network settings.
2. We need it for **speed, accuracy and easy management**.
3. Without it you get static IPs or **APIPA** (`169.254.x.x`).
4. It provides **IP, mask, gateway, DNS and lease time**.
5. Terms: **Scope, Pool, Lease, Reservation, Exclusion**.
6. **DORA:** Discover → Offer → Request → ACK.
7. **Ports:** UDP 67 (server), UDP 68 (client).
8. **Broadcast** is required because the client has no IP yet.
9. **DHCP Relay** carries requests to a server on another network.

---

[Previous](./01-networking-fundamentals.md) | [Course index](./README.md) | [Next: Lab setup](./03-lab-setup.md)
