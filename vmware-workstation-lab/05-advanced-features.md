# 05 — Advanced Features (So Far)

[← Previous: Connecting to a Real Network](04-real-network.md) · [Back to README](README.md) · [Next: Key Takeaways →](NOTES.md)

## Snapshot Manager

From the VM menu, go to **Snapshot > Snapshot Manager**. This gives you a tree view of every snapshot you've taken, showing which snapshot branched off of which.

## Switching Between Snapshots

Inside Snapshot Manager, click any snapshot and select **Go to** — the VM will jump right back to the exact state it was in when that snapshot was taken. It's like rewinding time.

## Shared Folders Between Host and Guest

A **Shared Folder** lets you see a folder from the Host inside the VM too, without needing a USB drive or a separate network share.

## Creating a C:\Share Folder on the Host

On your main computer's C: drive, create a folder:

```
C:\Share
```

## Enabling Shared Folders in VM Settings

Go to the VM's settings (VM Settings) > **Options** tab > **Shared Folders** section. Select **Always enabled**, then click Add and point it to `C:\Share`.

## Accessing It from Inside the VM

Inside the VM's File Explorer, open this path:

```
\\vmware-host\Shared Folders\Share
```

This is exactly the folder you created on the Host.

## Two-Way Test

Drop a test file into `C:\Share` from the Host and check that it shows up inside the VM. Then do the reverse — put a file inside the VM in that shared path and confirm it appears on the Host.

---

> **Pro Tip:** Use Shared Folders for light files (scripts, config files, text documents) rather than large ones — the speed usually doesn't match a real network transfer.

> **Exercise:** Create a text file, write something in it from the VM, open it on the Host and add a line, then reopen it from the VM to see the change reflected.

---

[← Previous: Connecting to a Real Network](04-real-network.md) · [Back to README](README.md) · [Next: Key Takeaways →](NOTES.md)
