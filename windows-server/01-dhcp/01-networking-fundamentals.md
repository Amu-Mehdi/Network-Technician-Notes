# DHCP, Part 1: Networking Fundamentals

> IP address, subnet mask, default gateway and DNS: the four settings DHCP hands out.

[Course index](./README.md) | [Next: DHCP concepts](./02-dhcp-concepts.md)

## Contents

- [What is DHCP, in one sentence?](#what-is-dhcp-in-one-sentence)
- [Why DHCP exists](#why-dhcp-exists)
- [Course roadmap](#course-roadmap)
- [Default Gateway](#default-gateway)
- [DNS](#dns)
- [Gateway vs DNS](#gateway-vs-dns)
- [Summary](#summary)

---

## What is DHCP, in one sentence?

Think of a hotel with 50 rooms. Every guest needs a room number, a key, the Wi-Fi password and directions to breakfast. Doing that by hand for each guest would be exhausting.

**DHCP** (Dynamic Host Configuration Protocol) is the hotel's **front desk** for a network. Every client that joins gets, automatically:

| Hotel | Network |
|-------|---------|
| Room number | **IP address** |
| House rules | **Subnet mask** |
| Exit to the city | **Default gateway** |
| Phone book | **DNS server** |

## Why DHCP exists

In the 1980s and early 1990s an administrator typed an IP address into **every** computer by hand. In a company with 500 PCs, replacing a laptop or letting guests onto the Wi-Fi meant more manual work each time.

DHCP changed that: the administrator defines a range once, and the server hands out addresses on its own. Modern networks are practically unmanageable without it.

## Course roadmap

| Part | File | Topic |
|------|------|-------|
| 1 | this file | IP, subnet mask, gateway, DNS |
| 2 | [DHCP concepts](./02-dhcp-concepts.md) | Scope, lease, reservation, exclusion, DORA, ports, relay |
| 3 | [Lab setup](./03-lab-setup.md) | Windows Server + client in VMware, DHCP role, first scope |
| 4 | [Experiments](./04-hands-on-experiments.md) | Lease, exclusion, reservation, renew, release |
| 5 | [Troubleshooting](./05-troubleshooting.md) | 12 real-world scenarios |
| 6 | [Final project](./06-final-project.md) | DHCP design for a small company |

The lab we will build:

![Lab topology](./images/lab-topology.svg)

This mirrors a small company network: one server providing DHCP (and later DNS) and clients that only need to connect and receive their settings.

---

## Default Gateway

### Everyday analogy

You live in unit 5 of an apartment building. You can talk to every neighbor inside the building without leaving. To reach a shop **outside**, you must go through the **building's exit door**.

The **Default Gateway** is that exit door for a network.

### Technical definition

The **default gateway** is the IP address of a device (usually a router or firewall) that provides the **path out of the local network**. When your computer wants to reach an address outside its own subnet, it sends the packets to the gateway, and the gateway forwards them.

![Default Gateway](./images/default-gateway.svg)

### What happens without a gateway?

The computer can still talk to devices **on its own network** (a printer, an internal server), but it has **no access to the internet** because it does not know where the exit is.

> [!WARNING]
> "I have an IP address but no internet" is one of the most common network problems, and the cause is very often the **gateway** or **DNS** setting.

### Convention

The gateway is usually the **first** or **last** usable address of the subnet (`192.168.1.1` or `192.168.1.254`). There is no rule; it is just a habit that makes the router easy to remember.

---

## DNS

### Everyday analogy

You want to call a friend but do not remember the number. You type their name into your phone and the number appears. **DNS** (Domain Name System) is the **phone book of the internet**.

### Why it is needed

Computers communicate with **IP addresses**, not names. You type `google.com`; the computer needs something like `142.250.185.78`. DNS translates names into IP addresses.

![DNS resolution](./images/dns-resolution.svg)

### What happens without DNS?

- Connecting by IP still works (`ping 8.8.8.8` succeeds).
- Names do not work (`ping google.com` fails).

> [!TIP]
> **Troubleshooting rule:** if `ping 8.8.8.8` works but `ping google.com` does not, the problem is **DNS**, not the internet connection.

### Well-known DNS servers

| Provider | Address | Notes |
|----------|---------|-------|
| Google Public DNS | `8.8.8.8`, `8.8.4.4` | Free, widely used |
| Cloudflare | `1.1.1.1` | Fast, privacy-focused |
| OpenDNS | `208.67.222.222` | Security filtering |
| Internal company DNS | e.g. `192.168.1.5` | Used in corporate networks |

---

## Gateway vs DNS

| | Default Gateway | DNS Server |
|---|---|---|
| **Purpose** | Path out of the network | Translates names to IPs |
| **Example** | `192.168.1.1` | `8.8.8.8` |
| **If missing** | Local network only | Works only with IP addresses |
| **Analogy** | Building exit door | Phone book |
| **How many** | One | Usually one or two (can be more) |

## Summary

A typical client configuration:

```text
IP:       192.168.1.50
Subnet:   255.255.255.0
Gateway:  192.168.1.1
DNS:      8.8.8.8
```

This means:

- My address is `192.168.1.50`.
- My network is `192.168.1.0/24` (usable hosts `.1` to `.254`).
- To leave the network, I go through `192.168.1.1`.
- To translate a name, I ask `8.8.8.8`.

**All four of these values are delivered to the client by DHCP.**

---

[Course index](./README.md) | [Next: DHCP concepts](./02-dhcp-concepts.md)
