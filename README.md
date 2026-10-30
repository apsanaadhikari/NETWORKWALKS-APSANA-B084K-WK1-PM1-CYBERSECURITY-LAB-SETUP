# Cybersecurity Lab Setup (WK1-PM1)

**Name:** Apsana Adhikari
**Program:** Networkwalks Cybersecurity Internship, Week 1

## Objective
Build a cybersecurity practice lab on my own laptop using VirtualBox, with Kali Linux as the attacker machine on an isolated NAT Network (`10.0.0.0/24`).

This is my own lab environment. All testing happens inside my own virtual machines.

## Lab Overview

| Item | Value |
|------|-------|
| Hypervisor | Oracle VirtualBox |
| Attacker VM | Kali Linux |
| Network type | NAT Network (`NatNetwork`) |
| Subnet | `10.0.0.0/24` |
| Kali IP | `10.0.0.2/24` |
| Gateway | `10.0.0.1` |
| DNS | `8.8.8.8` |

## Steps

### 1. Create the NAT Network
In VirtualBox: `File > Tools > Network > NAT Networks > Create`. Name `NatNetwork`, IPv4 prefix `10.0.0.0/24`, DHCP enabled.

![NAT Network](screenshots/01-natnetwork.png)

### 2. Attach Kali to the NAT Network
Kali VM `Settings > Network > Adapter 1`: Attached to **NAT Network**, name `NatNetwork`.

![Network adapter](screenshots/02-network-adapter.png)

### 3. Enable clipboard and drag & drop
`Settings > General > Advanced`: Shared Clipboard and Drag'n'Drop set to **Bidirectional**.

![Clipboard and drag and drop](screenshots/03-clipboard-dragdrop.png)

### 4. Shared folder
`Settings > Shared Folders`: shared my host Downloads folder with Auto-mount enabled.

![Shared folder](screenshots/04-shared-folder.png)

### 5. Set the static IP in Kali
`Edit Connections > Wired connection 1 > IPv4 Settings`: Method Manual, address `10.0.0.2`, netmask `24`, gateway `10.0.0.1`, DNS `8.8.8.8`.

![IPv4 settings](screenshots/05-ipv4-settings.png)

### 6. Test connectivity
```bash
ip a
ping -c 4 8.8.8.8
ping -c 4 google.com
```
Kali has the IP `10.0.0.2/24` and full internet access.

![IP and ping test](screenshots/06-ping-test.png)

### 7. Snapshot
Took a snapshot of the clean working lab so I can restore it at any time.

![Snapshot](screenshots/07-snapshot.png)

## Troubleshooting

### Problem 1: No internet / IP could not be reserved
After setting the static IP, `nmcli connection up` failed with:

`Error: Connection activation failed: IP configuration could not be reserved (no available address, timeout, etc.)`

**Cause:** a known issue with VirtualBox 7 and newer Kali versions, where the duplicate address detection (DAD) check times out.

**Fix:**
```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```
After this the connection activated and the pings worked.

![Error](screenshots/08-error.png)

### Problem 2: Forgot my Kali password
I could not log in to my own Kali VM.

**Fix:** rebooted the VM, pressed `e` at the GRUB menu, added `init=/bin/bash` to the end of the `linux` line, booted with `Ctrl+X`, then ran:
```bash
mount -o remount,rw /
passwd <username>
exec /sbin/init
```
Tip: the GRUB menu only shows for about 2 seconds, so tap the Down arrow repeatedly right after the VM starts to stop the countdown. I also had to check the real username with `ls /home`, because it was not `kali`.

## What I learned
- How to build an isolated NAT Network in VirtualBox
- How to set a static IP on Kali with NetworkManager
- How to troubleshoot network and login problems on a Linux VM
- Why snapshots matter before experimenting
