# 01 — VMware Basics

[← Back to README](README.md) · [Next: Cloning VMs →](02-cloning-vms.md)

## What is VMware Workstation, and why use it?

Say you want a Windows Server to practice on, but you don't want to mess up your main PC or buy separate hardware. That's where a **Virtual Machine (VM)** comes in. VMware Workstation is software that lets you run several "fake" computers inside your real one. Each VM behaves like an independent machine — as if you had several separate laptops, all packed inside one box.

For anyone learning networking, this is huge: you can build an entire lab environment without buying any physical hardware, and you can break it and rebuild it as many times as you want.

## Virtual Machine vs. Physical Machine

- **Physical Machine**: your actual computer, with real hardware (real RAM, real disk, real CPU).
- **Virtual Machine**: a simulated computer that "borrows" resources (RAM, disk, CPU) from the physical machine, but behaves as if it's a completely separate system.

Fun fact: a VM has no idea it's virtual — as far as it's concerned, it's a real computer.

## Core Concepts

- **Host** — your main, real computer that VMware is installed on.
- **Guest** — the virtual machine running inside the Host.
- **Hypervisor** — the software layer responsible for creating and managing VMs. VMware Workstation itself is a type of Hypervisor.
- **Snapshot** — a point-in-time save of a VM's exact state. If something breaks later, you can roll back to that moment.
- **Clone** — a full copy of an existing VM, so you don't have to reinstall everything from scratch.

## Installing VMware Workstation and Creating Your First VM

After downloading and installing VMware Workstation (a standard Next → Next → Finish install), to create your first VM:

1. Go to **File > New Virtual Machine**.
2. Choose **Typical (recommended)** — it's enough to get started.
3. Point it to your Windows Server ISO file.

## Selecting the Windows Server ISO

An ISO file is basically the Windows installer packed into a single bootable file. On the install screen, choose **Installer disc image file (iso)** and browse to your file.

## Easy Install Settings

VMware has a feature called **Easy Install** that automates most of the Windows setup. It will ask for:

- Username
- Password
- Product key (optional — you can skip it)

## Naming the VM

Pick a clear, meaningful name, for example:

```
Server-01
```

This name shows up in the VMware library and becomes the default folder name too.

## Choosing a Storage Location

It's better to use a dedicated path rather than dumping everything on your C: drive:

```
D:\VMs\Servers\Server-01
```

## Suggested Folder Structure for VMs

To avoid losing track of things later, organize your VMs folder like this:

```
D:\VMs
├── Servers
├── Clients
├── ISOs
└── Backups
```

## Choosing UEFI Instead of BIOS

In the VM's hardware settings, find the Firmware Type option and select **UEFI** instead of the older BIOS. UEFI is the modern standard and is recommended for newer Windows Server versions.

## Processor Configuration

For practice and learning, this is usually enough:

- Number of processors: **2**
- Number of cores per processor: **1**

## Memory Configuration

Set the VM's dedicated RAM to **4096 MB (4 GB)**. This is a reasonable minimum for Windows Server.

## Choosing NAT for Networking

On the Network Type step, select **NAT**. This lets the VM reach the internet through the Host, without getting its own separate IP on the outside network.

## Choosing LSI Logic SAS for the Controller

For I/O Controller Type, choose **LSI Logic SAS** — it has good compatibility with modern Windows Server versions.

## Choosing NVMe for Disk Type

For Disk Type, choose **NVMe**. It's faster than the older IDE or SCSI options and closer to what real modern disks look like.

## Choosing "Create a New Virtual Disk"

From the available options, select **Create a new virtual disk** (as opposed to reusing an existing one).

## Setting Maximum Disk Size

Set the maximum disk size to **60 GB**. This is just a ceiling — the disk won't actually take up that much space right away.

## Choosing "Allocate Disk Space Later"

This means the disk space is allocated dynamically — the actual file on the Host's hard drive only grows as data is actually used, not immediately taking up the full 60 GB.

## Choosing "Store Virtual Disk as a Single File"

This stores the entire virtual disk as one single file rather than splitting it into multiple pieces — easier to manage, especially if you need to move it later.

## Installing Windows Server Inside the VM

After clicking Finish, VMware boots from the ISO and Windows Server setup begins. With Easy Install, the entire installation and initial configuration is usually handled automatically.

---

> **Pro Tip:** Before you start, make sure Virtualization is enabled in your Host's BIOS (usually labeled Intel VT-x or AMD-V) — otherwise VMware won't even boot a VM.

> **Exercise:** Create a new VM with the same settings, but this time set Memory to 2048 MB and compare the boot speed difference.

---

[← Back to README](README.md) · [Next: Cloning VMs →](02-cloning-vms.md)
