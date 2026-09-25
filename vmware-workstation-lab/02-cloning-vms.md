# 02 — Cloning VMs

[← Previous: VMware Basics](01-vmware-basics.md) · [Back to README](README.md) · [Next: VM-to-VM Networking →](03-vm-networking.md)

## Taking a Snapshot

Before doing anything risky, it's a good habit to take a snapshot. From the VM menu, go to **Snapshot > Take Snapshot** and give it a clear name, for example:

- `Before-Network-Config`
- `After-Install`

Think of it like saving a game before a tough level — if something breaks, you can roll right back.

## Full Clone from Server-01 to Server-02

Right-click the first VM (Server-01) and go to **Manage > Clone**. Choose **Full Clone**, not Linked Clone. The difference:

- **Full Clone** creates a completely independent copy.
- **Linked Clone** stays dependent on the original VM's files — if the original is deleted, the clone breaks too.

## Storage Path for Server-02

As before, pick an organized path:

```
D:\VMs\Servers\Server-02
```

## After Cloning: Change the Hostname

Since Server-02 is an exact copy of Server-01, the computer name (Hostname) is identical too. Right after booting it up, go to:

```
System Properties > Change Settings > Change...
```

Rename it to `Server-02` and restart.

## After Cloning: Change the IP

If Server-01 already had a static IP assigned, Server-02 has the exact same one (since it's a copy) — this causes an **IP conflict**. Make sure to change the second server's IP (covered in more detail in the next section).

## Snapshot: After-Clone

Once the Hostname and IP are fixed, take a new snapshot called `After-Clone` to lock in this clean state.

---

> **Pro Tip:** Always change the Hostname and IP immediately after cloning. A lot of people skip this step and later get confused by mysterious network conflicts.

> **Exercise:** Create a third clone from Server-01 called Server-03, and change both its name and IP so the process really sticks.

---

[← Previous: VMware Basics](01-vmware-basics.md) · [Back to README](README.md) · [Next: VM-to-VM Networking →](03-vm-networking.md)
