<div align="center">

# 🐧 The Complete Guide to Installing Ubuntu on VMware Workstation

### From zero to done — step by step, with every option explained

![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04.2%20LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![VMware](https://img.shields.io/badge/VMware-Workstation%2016%20Pro-607078?style=for-the-badge&logo=vmware&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner%20to%20Intermediate-blue?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

</div>

---

> 📌 **About this guide**
> This is a complete, visual, step-by-step walkthrough for installing Ubuntu 24.04 LTS on a virtual machine in VMware Workstation 16 Pro. Every installer option comes with the reasoning behind the choice, so you don't just click through — you actually **understand** what you're doing.

<br>

## 📑 Table of Contents

- [Introduction & Core Concepts](#-introduction--core-concepts)
- [Prerequisites](#-prerequisites)
- [Part 1: Creating the Virtual Machine](#-part-1-creating-the-virtual-machine)
- [Part 2: Installing Ubuntu](#-part-2-installing-ubuntu)
- [Part 3: Post-Install Checks & Initial Setup](#-part-3-post-install-checks--initial-setup)
- [Important Notes](#-important-notes)
- [Summary](#-summary)
- [Useful Resources](#-useful-resources)
- [Appendix: Handy Linux Commands](#-appendix-handy-linux-commands)

<br>

---

## 🧩 Introduction & Core Concepts

<table>
<tr><td width="140"><b>🐧 Linux</b></td><td>
An <b>open-source</b> operating system — the layer between your hardware and your applications, similar to Windows or macOS, except anyone can view, modify, and freely use its source code. On its own, Linux is just a <b>kernel</b>; combined with a set of software and tools, it becomes a <b>distribution (distro)</b>.
</td></tr>
<tr><td><b>🟠 Ubuntu</b></td><td>
A Linux distribution backed by <b>Canonical</b> — free, user-friendly, and a great starting point for beginners.
</td></tr>
<tr><td><b>💻 VMware</b></td><td>
<b>Virtualization software</b> that lets you build a "virtual" computer on top of your real machine, one that borrows resources (CPU, RAM, storage) from your host system.
</td></tr>
<tr><td><b>📦 Virtual Machine</b></td><td>
That virtual computer running inside VMware (a <b>VM</b>). You can install any OS on it and run several VMs side by side.
</td></tr>
</table>

### Why install Ubuntu inside a VM?

| Benefit | Explanation |
|---|---|
| 🛡️ **Safety** | Anything that breaks stays inside the VM — your host machine is untouched |
| 🧪 **Testing & learning** | Experiment freely, tweak settings, wipe it and start over anytime |
| 🔀 **Run side by side** | Keep Windows and Ubuntu open at the same time |
| 💾 **No risky partitioning** | Unlike a native install, there's zero risk to your real disk's data |

> ⚖️ **Native install vs. virtual machine**
> A native install is faster and sees all your real hardware (including the GPU), but requires disk partitioning. A VM install is a bit slower but completely safe — perfect for learning and experimentation.

<br>

## ✅ Prerequisites

### 1. The Ubuntu ISO file
- Official source: **[ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)**
- Recommended version: `Ubuntu 24.04 LTS`
- File size: roughly 4–6 GB
- `LTS` = Long Term Support → 5 years of updates, the best choice for beginners

### 2. VMware Workstation
- Download: **[vmware.com/products/workstation-player](https://www.vmware.com/products/workstation-player.html)**
- Version: VMware Workstation 16 Pro or later
- Installation is straightforward — accept the defaults

### 3. Recommended system specs

| Resource | Minimum | Recommended |
|---|---|---|
| 🧠 RAM | 8 GB | 12 GB or more |
| ⚙️ CPU | 4 cores | — |
| 💽 Free disk space | 30 GB | — |
| 🖥️ Host OS | Windows 10/11, Linux, or macOS | — |

### 4. Suggested folder structure

```text
C:\VMs\
├── Backups\         # Backups
├── Clients\         # Client virtual machines
│   └── Ubuntu-24.04-Learning\
├── ISOs\            # ISO files
│   └── ubuntu-24.04.2-desktop-amd64.iso
├── Servers\         # Server virtual machines
└── Templates\       # Templates
```

<br>

---

## 🛠 Part 1: Creating the Virtual Machine

<details open>
<summary><b>Step 1 — Open VMware and start the wizard</b></summary>

1. Open **VMware Workstation**.
2. From the menu: `File > New Virtual Machine`
3. Or click **Create a New Virtual Machine** on the home screen.

The **New Virtual Machine Wizard** opens.
</details>

<details open>
<summary><b>Step 2 — Configuration type</b></summary>

| Option | Description |
|---|---|
| Typical (recommended) | Simple, default settings |
| **Custom (advanced)** ✅ | Manual setup, full control |

**Why Custom?** Because we want to configure everything ourselves and actually learn the process.
</details>

<details open>
<summary><b>Step 3 — Hardware compatibility</b></summary>

- **Hardware compatibility:** `Workstation 16.2.x` ✅ (kept as default)
- **Compatible with:** `ESX Server` (checkbox couldn't be unchecked — no issue)
- **Limits:** up to 128 GB RAM, 32 CPUs, 10 network adapters, 8 TB disk — far beyond what we need.
</details>

<details open>
<summary><b>Step 4 — Select the ISO file (Guest OS Installation)</b></summary>

Option chosen: **Installer disc image file (ISO)** ✅

```text
C:\VM\ISOs\ubuntu-24.04.2-desktop-amd64.iso
```

> ⚠️ If you pick a folder instead of the actual file, VMware wrongly shows `Windows Server 2019 detected`. Selecting the correct ISO file shows `Ubuntu 64-bit 24.04.2 detected` instead.

**About Easy Install:** When VMware detects an Ubuntu ISO, it enables automatic installation (Easy Install). To learn every installer step yourself, it's better to disable it — in VMware 16 Pro there's no checkbox for this, so you exit it by clicking **Cancel** on the next screen (`Easy Install Information`).
</details>

<details open>
<summary><b>Step 5 — Name and location of the VM</b></summary>

| Field | Value |
|---|---|
| Virtual machine name | `Ubuntu-24.04-Learning` |
| Location | `C:\VMs\Clients\Ubuntu-24.04-Learning` |

💡 Tip: Keep the name and path consistent, and store it on a drive with enough free space.
</details>

<details open>
<summary><b>Step 6 — Processor configuration</b></summary>

For a 4-core host system:

| Field | Value |
|---|---|
| Number of processors | `1` |
| Number of cores per processor | `2` |
| Total processor cores | `2` (automatic) |

**Why?** 2 cores are plenty for Ubuntu Desktop, leaving 2 cores free for your host system.

> 🚫 Never allocate more cores to a VM than your CPU actually has.
</details>

<details open>
<summary><b>Step 7 — Memory (RAM)</b></summary>

Chosen amount: **`4096 MB`** (4 GB)

- Enough for Ubuntu Desktop.
- Leaves 8 GB free for the host system.

> ⚠️ Don't go below 2048 MB (2 GB) — Ubuntu may run slowly or fail to install.
</details>

<details open>
<summary><b>Step 8 — Network type</b></summary>

| Option | Description |
|---|---|
| Bridged | Direct connection to the external network with its own IP |
| **NAT** ✅ | Routed through the host — simple, secure, automatic |
| Host-only | Private network between host and VM, no internet |
| No network connection | — |

**Why NAT?** The simplest option — internet works automatically, better security (external machines can't reach the VM directly), and it works over Wi-Fi too.
</details>

<details open>
<summary><b>Step 9 — I/O controller type</b></summary>

Chosen: **LSI Logic (Recommended)** ✅ — its driver ships with Linux by default, making it the safest, most reliable option.
</details>

<details open>
<summary><b>Step 10 — Disk type</b></summary>

| Type | Notes |
|---|---|
| IDE | Old standard, slow |
| SCSI | Stable, older than SATA |
| SATA | Common, modern standard |
| **NVMe** ✅ | Newest and fastest — final choice |

Ubuntu 24.04 ships with an NVMe driver, giving better boot and file-access speed.
</details>

<details open>
<summary><b>Step 11 — Select a disk</b></summary>

Chosen: **Create a new virtual disk** ✅

> 🚫 **Warning:** Never select **Use a physical disk** — it can wipe data on your actual hard drive.
</details>

<details open>
<summary><b>Step 12 — Disk capacity</b></summary>

| Field | Value |
|---|---|
| Maximum disk size | `20 GB` (use `40 GB` if you have the space) |
| Allocate all disk space now | ❌ off |
| Split virtual disk into multiple files | ✅ on |

**Why these settings?** Not pre-allocating means only actual used space is consumed; splitting the disk makes it easier to move and back up.
</details>

<details open>
<summary><b>Step 13 — Disk file</b></summary>

```text
Ubuntu-24.04-Learning.vmdk
```

`VMDK` = Virtual Machine Disk, the file that holds the virtual hard drive. Since we chose Split, companion files like `s001`, `s002`, etc. are created alongside it.
</details>

<details open>
<summary><b>Step 14 — Ready to create</b></summary>

Final settings summary:

```yaml
Name:              Ubuntu-24.04-Learning
Location:          C:\VMs\Clients\Ubuntu-24.04-Learning
Version:           Workstation 16.2.x
Operating System:  Ubuntu 64-bit
Hard Disk:         20 GB, Split
Memory:            4096 MB
Network Adapter:   NAT
Other Devices:     2 CPU cores, CD/DVD, USB Controller, Printer, Sound
Power on after creation: ✅
```

Click **Finish** — the VM is created and powered on.
</details>

<br>

---

## 💿 Part 2: Installing Ubuntu

<details open>
<summary><b>Step 1 — Boot from the ISO</b></summary>

- If you see `Press any key to boot from CD or DVD`, press a key (e.g. Space) quickly.
- If a `moved or copied` prompt appears, choose **I Copied It**.
- Easy Install may kick off automatically — that's expected.
</details>

<details open>
<summary><b>Step 2 — Choose language (Welcome to Ubuntu)</b></summary>

Chosen: **English** ✅

**Why?** For learning Linux, most tutorials and documentation are in English; you can always change it later from settings.

> 💡 If your mouse or keyboard isn't responding, click once inside the Ubuntu window.
</details>

<details open>
<summary><b>Step 3 — Accessibility</b></summary>

Options like Seeing, Hearing, Typing, Pointing and clicking, Zoom.

Chosen: **none** ✅ — not needed for a standard install; can be enabled later from `System Settings > Accessibility`.
</details>

<details open>
<summary><b>Step 4 — Keyboard layout</b></summary>

| Field | Value |
|---|---|
| Keyboard layout | `English (US)` ✅ |
| Keyboard variant | `English (US)` ✅ |

Type something in the test box (e.g. `hello world`) to confirm the layout is correct.
</details>

<details open>
<summary><b>Step 5 — Internet connection</b></summary>

Chosen: **Use wired connection** ✅

**Why?** Since the VM's network is set to NAT, VMware provides a virtual network adapter that behaves like a wired connection, which Ubuntu detects as such. The Wi-Fi option is greyed out because the VM has no Wi-Fi hardware.
</details>

<details open>
<summary><b>Step 6 — Try or install Ubuntu</b></summary>

| Option | Description |
|---|---|
| **Install Ubuntu** ✅ | Permanent installation to disk |
| Try Ubuntu | Temporary (Live) session, no changes saved |
</details>

<details open>
<summary><b>Step 7 — Type of installation</b></summary>

| Option | Description |
|---|---|
| **Interactive installation** ✅ | Step-by-step, fully user-controlled |
| Automated installation | Driven by an `autoinstall.yaml` file, meant for organizations |
</details>

<details open>
<summary><b>Step 8 — Applications</b></summary>

| Option | Size | Contents |
|---|---|---|
| **Default selection** ✅ | ~5–6 GB | Essential apps (browser, file manager, terminal...) |
| Extended selection | ~8–10 GB | + LibreOffice, Thunderbird, Rhythmbox, games, etc. |

**Why Default?** Disk space is limited (20 GB); anything extra can be installed later with `sudo apt install <name>`.
</details>

<details open>
<summary><b>Step 9 — Optimise your computer (proprietary software)</b></summary>

Checked: **Install third-party software for graphics and Wi-Fi hardware** ✅

The "Additional media formats" option was greyed out because the installer hadn't detected an internet connection; it can be installed later with:

```bash
sudo apt install ubuntu-restricted-extras
```
</details>

<details open>
<summary><b>Step 10 — Disk setup</b></summary>

Chosen: **Erase disk and install Ubuntu** ✅

> ⚠️ The wording **Erase disk** sounds alarming, but since you're inside a **virtual machine**, that "disk" is just a virtual file under `C:\VMs\...` — there's no risk to your host OS or real data.

**Advanced features** (LVM, Encryption, ZFS) was left as `None selected` — not needed for home use.
</details>

<details open>
<summary><b>Step 11 — Automatic partitioning</b></summary>

Ubuntu created these partitions automatically:

- `nvme0n1p1` → Boot/EFI partition
- `nvme0n1p2` → Root partition (`/`) using the `ext4` filesystem

<table>
<tr><td width="140"><b>Disk</b></td><td>The physical drive — like a bookshelf</td></tr>
<tr><td><b>Partition</b></td><td>A division of the disk into smaller sections — like shelves</td></tr>
<tr><td><b>Filesystem</b></td><td>The method used to store data on a partition — like how books are arranged</td></tr>
<tr><td><b>Root (/)</b></td><td>The most important partition; holds the OS, applications, and settings</td></tr>
<tr><td><b>Home (/home)</b></td><td>Where each user's personal files live</td></tr>
<tr><td><b>Swap</b></td><td>Extra disk space used as temporary RAM when memory fills up</td></tr>
</table>

**Disk naming in Linux:**

| Prefix | Type |
|---|---|
| `sda`, `sdb`, ... | SATA / SCSI |
| `nvme0n1`, `nvme1n1`, ... | NVMe |
| `vda`, `vdb`, ... | VirtIO (virtual) |
</details>

<details open>
<summary><b>Step 12 — Create your account</b></summary>

| Field | Rules |
|---|---|
| **Your name** | Display name; letters (any language), spaces allowed |
| **Computer's name** | No spaces; lowercase letters, digits, and hyphens only — e.g. `ubuntu-learning` |
| **Username** | Only `a-z`, `0-9`, `-`, `_` — no spaces |
| **Password** | At least 8 characters, mix of upper/lowercase, digits, symbols |
| **Require password to log in** | ✅ on (extra security) |
| **Use Active Directory** | ❌ off (organizations only) |

> ⚠️ **Write your password down somewhere safe** — you'll need it for installing software and every admin command in the terminal, and it isn't easy to recover.
</details>

<details open>
<summary><b>Step 13 — Timezone</b></summary>

Chosen: **`Asia/Tehran`** ✅ *(pick your own timezone here)*

> 💡 Depending on the timezone chosen, the installer may suggest switching the system language to match the region — this is normal and can always be reverted to English later.
</details>

<details open>
<summary><b>Step 14 — Ready to install</b></summary>

```yaml
Disk setup:        Erase disk and install Ubuntu
Installation disk: nvme0n1
Applications:       Default selection
Disk encryption:   None
Proprietary sw:    Drivers
Partitions:        nvme0n1p1 + nvme0n1p2 (ext4, /)
```

> ⏱️ Installation takes roughly 10–30 minutes. Don't power off the VM or close the VMware window during this time. Seeing Ubuntu's feature slideshow is normal.

Click **Install**.
</details>

<details open>
<summary><b>Step 15 — Installation complete</b></summary>

Chosen: **Restart now** ✅ — the VM reboots and this time boots from the hard disk instead of the ISO.
</details>

<details open>
<summary><b>Steps 16–19 — Ubuntu's welcome wizard</b></summary>

| Step | Choice made | Reason |
|---|---|---|
| Welcome | Next | Start the wizard |
| **Ubuntu Pro** | Skip for now ✅ | Standard 5-year LTS support is plenty for home use |
| **Help improve Ubuntu** | No, don't share system data ✅ | Privacy |
| **Ready to go** | Finish ✅ | Wizard done, land on the desktop |
</details>

<br>

---

## 🔍 Part 3: Post-Install Checks & Initial Setup

### ✅ Final checklist

| # | Item | How to check | Fix if it fails |
|---|---|---|---|
| 1 | 🌐 Internet | Open Firefox and load a site | Check NAT in VMware settings and the `VMware NAT Service` on the host |
| 2 | 🖥️ Display | Check size and resolution | Install `open-vm-tools` |
| 3 | 🖱️ Mouse & keyboard | Test movement, typing, shortcuts | Click inside the window, or press `Ctrl+Alt` to release the mouse |
| 4 | 🧰 VMware tools | Run the command below | Install manually if missing |
| 5 | 🌍 System language | `Settings > Region & Language` | Switch to English if you'd like |
| 6 | 🕒 Time & timezone | Top-bar clock / `Settings > Date & Time` | Make sure Automatic Date & Time is on |
| 7 | 💽 Disk space | `df -h` | Free up space or grow the virtual disk |
| 8 | ⚡ Overall performance | Open several apps at once | Increase RAM/CPU in VMware settings |

**Checking VMware tools (open-vm-tools):**

```bash
dpkg -l | grep open-vm-tools
```

If it's not installed:

```bash
sudo apt update
sudo apt install open-vm-tools open-vm-tools-desktop
```

> ℹ️ These tools smooth out mouse/keyboard input, auto-resize the display, and enable copy/paste and shared folders between host and VM.

### 🔄 Updating the system

**From the terminal:**

```bash
sudo apt update    # refresh the package list
sudo apt upgrade   # upgrade installed packages
```

**From the GUI:** open **Software Updater** and click **Install Now**.

### 📸 Taking a snapshot

A snapshot is an instant capture of the VM's current state — like a save point in a game. If something breaks later, you can roll back to it.

```text
VM > Snapshot > Take Snapshot
Suggested name: "Fresh Install - Clean"
```

> 💡 Snapshots use disk space — don't take too many.

<br>

---

## ⚠️ Important Notes

- 🔑 **Don't lose your password** — recovering it is not simple.
- 🔌 **Shut the VM down properly** — use `Power Off` from Ubuntu's menu, not the window's X button.
- 📸 **Take a snapshot before any major change.**
- 💽 **Manage disk space** — clean up files or grow the virtual disk as needed.
- 🧰 **Keep open-vm-tools up to date.**
- 🖥️ **Don't fear the terminal** — it's your friend, not your enemy.
- 🌍 **System language** can be changed anytime from `Settings > Region & Language`.
- 🌐 **Stick with NAT networking** — it's the most reliable option.
- 💾 **Back things up** — keep a copy of the whole `C:\VMs\Clients\Ubuntu-24.04-Learning\` folder.
- 📚 **Keep learning** — the terminal, software installs, networking, servers, and much more await.

<br>

---

## 🏁 Summary

<table>
<tr><td width="170" valign="top"><b>What we learned</b></td><td>

Core concepts (Linux, Ubuntu, VMware, VMs) • Building a VM from scratch • A full Ubuntu 24.04 LTS installation • Post-install verification

</td></tr>
<tr><td valign="top"><b>What we did</b></td><td>

- Built a VM with VMware Workstation 16 Pro
- Installed Ubuntu 24.04.2 LTS
- Set networking to **NAT**
- Allocated 2 CPU cores and 4 GB of RAM
- Added a 20 GB **NVMe** virtual disk
- Used automatic partitioning
- Created an admin user account

</td></tr>
<tr><td valign="top"><b>What's next</b></td><td>

Getting comfortable with the Linux terminal → installing software from the App Center and the terminal → networking basics → user and group management → filesystem management and mounting → basic Bash scripting

</td></tr>
</table>

<br>

---

## 📚 Useful Resources

**Official docs**
- [help.ubuntu.com](https://help.ubuntu.com) — Ubuntu documentation
- [docs.vmware.com](https://docs.vmware.com) — VMware documentation

**Communities**
- [Ask Ubuntu](https://askubuntu.com)
- [Ubuntu Forums](https://ubuntuforums.org)
- [Reddit — r/Ubuntu](https://reddit.com/r/Ubuntu)

**Tutorials**
- [Ubuntu Tutorials](https://ubuntu.com/tutorials)
- [Linux Journey](https://linuxjourney.com)
- [The Linux Command Line](http://linuxcommand.org)

<br>

---

## 💻 Appendix: Handy Linux Commands

<details>
<summary><b>📦 Package management (apt)</b></summary>

```bash
sudo apt update                 # refresh the package list
sudo apt upgrade                # upgrade installed packages
sudo apt install [package-name] # install a package
sudo apt remove [package-name]  # remove a package
apt search [package-name]       # search for a package
```
</details>

<details>
<summary><b>📁 File management</b></summary>

```bash
pwd                      # print current directory
ls -la                   # list files (detailed, incl. hidden)
cd [directory]           # change directory
mkdir [directory-name]   # create a directory
cp [source] [dest]       # copy a file
mv [source] [dest]       # move or rename
rm [file-name]           # delete a file
rm -r [directory-name]   # delete a directory
```
</details>

<details>
<summary><b>🖥️ System information</b></summary>

```bash
df -h            # disk space
free -h          # memory usage
lscpu            # CPU information
uname -a         # OS/kernel info
lsb_release -a   # Ubuntu version
```
</details>

<details>
<summary><b>🌐 Networking</b></summary>

```bash
ip addr show     # show IP addresses
ping google.com  # test connectivity
wget [url]       # download a file
ss -tuln         # show open ports
```
</details>

<br>

---

<div align="center">

### 🎉 Congratulations!

You've successfully installed Ubuntu on a virtual machine — that's a solid accomplishment.
Now you're free to explore, learn, and experiment with Ubuntu.

**Good luck! 🚀**

<sub>Author: <code>[your name]</code> • Date: <code>[date]</code> • Ubuntu version: 24.04.2 LTS • VMware version: Workstation 16 Pro</sub>

</div>
