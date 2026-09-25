# 04 — Connecting a VM to a Real Network

[← Previous: VM-to-VM Networking](03-vm-networking.md) · [Back to README](README.md) · [Next: Advanced Features →](05-advanced-features.md)

## Setting Bridged and Connecting to the Real Network

To make a VM appear on your home or office network exactly like a real physical device, set its network adapter to **Bridged**. This way the VM gets an IP from the same modem/router your actual computer is connected to.

## Getting an Automatic IP from the Wi-Fi Modem

In the VM's Windows network settings, keep **DHCP** enabled (Obtain an IP address automatically) so the Wi-Fi modem assigns an IP automatically.

## Pinging 8.8.8.8

Test it again:

```
ping 8.8.8.8
```

If you get a response, the VM is now connected to the internet exactly like a real device would be.

## Port Forwarding (General Overview)

**Port Forwarding** means telling your router or modem to send any incoming request from the outside internet on a specific port to a particular device on your internal network — for example, this VM. Say you're running a website on the server: you could forward port 80 to the server's internal IP so it's reachable from outside too.

Right now, since I only have a simple Wi-Fi modem and no separate, dedicated router, I haven't actually tested this in practice yet — but that's the concept. I'll set it up properly once I get a proper router.

---

> **Pro Tip:** When you use Bridged mode, the VM is exposed on the network exactly like a real device — meaning any security vulnerability inside that VM could affect your whole home network. For practice, it's safer to stick with NAT or Host-only unless you genuinely need Bridged.

> **Exercise:** Compare the IP the VM gets in Bridged mode with your Host computer's own IP (using `ipconfig` on both). Check whether they're in the same range or not.

---

[← Previous: VM-to-VM Networking](03-vm-networking.md) · [Back to README](README.md) · [Next: Advanced Features →](05-advanced-features.md)
