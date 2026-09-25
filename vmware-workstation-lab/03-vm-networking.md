# 03 — VM-to-VM Networking

[← Previous: Cloning VMs](02-cloning-vms.md) · [Back to README](README.md) · [Next: Connecting to a Real Network →](04-real-network.md)

## Bridged vs. NAT vs. Host-Only vs. Custom

These four network modes in VMware are the most important thing to understand:

| Mode | Description |
|---|---|
| **Bridged** | The VM sits on the real network (router/modem) like an independent physical device and gets its own IP. |
| **NAT** | The VM reaches the internet through the Host but isn't visible from outside. Better for security. |
| **Host-Only** | The VM can only talk to the Host and other VMs on the same virtual network — no real internet access. Great for isolated network practice. |
| **Custom** | You manually choose which virtual network adapter the VM connects to — used for more complex setups. |

## Setting Both VMs to Host-Only

In each VM's settings, go to **Network Adapter** and select **Host-only**. Do this for both Server-01 and Server-02.

## Assigning a Static IP to Server-01

Inside Server-01's Windows, go to the network adapter settings (via Network and Sharing Center or PowerShell) and set a static IP:

```
IP Address: 192.168.20.1
Subnet Mask: 255.255.255.0
```

## Assigning a Static IP to Server-02

Do the same for Server-02, with a different IP:

```
IP Address: 192.168.20.2
Subnet Mask: 255.255.255.0
```

## Why the Subnet Mask Matters

The **Subnet Mask** determines which range of IPs are considered "neighbors" that can talk to each other directly. With `255.255.255.0`, the first three parts of the IP have to match (192.168.20.x) — only the last part can differ.

## Disabling the Windows Server Firewall

To let pings and network tests go through without issues (for practice purposes only — not for production environments), go to:

```
Control Panel > Windows Defender Firewall > Turn Windows Defender Firewall on or off
```

Turn off all three profiles (Domain, Private, Public).

## Pinging Between the Two VMs

From the Command Prompt on one of the servers, run:

```
ping 192.168.20.2
```

If you get replies ("Reply from..."), the network between the two VMs is working correctly.

## Switching Server-02's Adapter to Bridged

Now, to let Server-02 reach the real internet, change its network adapter in VMware settings from Host-only to **Bridged**.

## Getting an Automatic IP from the Modem

After switching to Bridged, go to Windows network settings and set IP mode back to **Obtain an IP address automatically** (i.e., DHCP), so it gets a real IP from your home modem.

## Pinging 8.8.8.8 (Internet Test)

To verify the internet connection is actually working:

```
ping 8.8.8.8
```

`8.8.8.8` is Google's well-known public DNS server, commonly used to test internet connectivity.

## Pinging Between the Host and VMs

From the Host machine itself, you can ping both servers too:

```
ping 192.168.20.1
ping 192.168.20.2
```

---

> **Pro Tip:** If pings aren't going through even though everything looks correctly configured, check the firewall first, then the network adapter settings — most networking issues come from one of these two.

> **Exercise:** Create a third VM (e.g., a Windows client), set it to Host-only with IP `192.168.20.3`, and ping between all three VMs.

---

[← Previous: Cloning VMs](02-cloning-vms.md) · [Back to README](README.md) · [Next: Connecting to a Real Network →](04-real-network.md)
