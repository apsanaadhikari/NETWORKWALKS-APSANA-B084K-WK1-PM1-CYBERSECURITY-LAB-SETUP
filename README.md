# 🔐 Cybersecurity Lab Setup

> A private, isolated virtual lab for ethical hacking practice, built with VirtualBox and Kali Linux.

**Author:** Apsana Adhikari | **Program:** Networkwalks Cybersecurity Internship | **Week:** 01 | **Project:** WK1-PM1

---

## 📌 Project Overview

I built a personal cybersecurity practice lab on my own laptop. Kali Linux runs as a virtual machine on a private NAT Network, so more machines can be added later as practice targets.

Everything in this project was done inside my own lab environment.

## 🎯 Objectives

- Install and configure VirtualBox
- Import Kali Linux as a virtual machine
- Create a private NAT Network (`10.0.0.0/24`)
- Give Kali a fixed IP address
- Enable clipboard, drag & drop and a shared folder
- Confirm full internet access
- Take a snapshot as a clean recovery point
- Document the process, including problems and fixes

## 🏗️ Lab Configuration

| Component | Configuration |
|-----------|---------------|
| 🧰 Hypervisor | Oracle VirtualBox |
| 🐉 Attacker machine | Kali Linux (64-bit) |
| 🧠 Kali resources | 4 CPUs, ~8.9 GB RAM, 50 GB disk |
| 🌐 Network type | NAT Network (`NatNetwork`) |
| 📡 Subnet | `10.0.0.0/24` |
| 🐧 Kali IP | `10.0.0.2/24` |
| 🚪 Gateway | `10.0.0.1` |
| 🌍 DNS | `8.8.8.8` |

## 🪜 Setup Steps

### Step 1: Create the NAT Network
In VirtualBox: `File > Tools > Network > NAT Networks > Create`.
Name `NatNetwork`, prefix `10.0.0.0/24`, DHCP enabled.

A NAT Network lets several VMs talk to each other and still reach the internet, which suits a multi-machine lab.

<img width="760" height="442" alt="01-natnetwork" src="https://github.com/user-attachments/assets/d59550b2-c9f3-4ec5-901c-bf197b45e957" />


### Step 2: Attach Kali to the NAT Network
`Settings > Network > Adapter 1`: Attached to **NAT Network**, name `NatNetwork`.
<img width="770" height="642" alt="02-network-adapter" src="https://github.com/user-attachments/assets/a280c05a-9dd8-40d1-bb87-d8bdb1542d29" />


### Step 3: Clipboard and drag & drop
`Settings > General > Advanced`: Shared Clipboard and Drag'n'Drop set to **Bidirectional**.
<img width="762" height="642" alt="03-clipboard-dragdrop" src="https://github.com/user-attachments/assets/b86960fa-3192-410b-9bf3-a7fca82ac294" />

### Step 4: Shared folder
`Settings > Shared Folders`: added my host Downloads folder with **Auto-mount** enabled, for moving files between the host and Kali.

<img width="767" height="641" alt="04-shared-folder" src="https://github.com/user-attachments/assets/0a31eaa4-3a0a-4095-9b23-be58ebd3d6c4" />
### Step 5: Static IP in Kali
`Edit Connections > Wired connection 1 > IPv4 Settings`:

| Field | Value |
|-------|-------|
| Method | Manual |
| Address | `10.0.0.2` |
| Netmask | `24` |
| Gateway | `10.0.0.1` |
| DNS | `8.8.8.8` |

A fixed IP makes the lab easier to document and reuse.
<img width="577" height="432" alt="05-ipv4-settings" src="https://github.com/user-attachments/assets/e41d6e0d-ab1c-4dab-a228-ac5a769c3ec9" />

### Step 6: Snapshot
Took a VirtualBox snapshot (**Lab Setup Done**) of the working lab. If a later experiment breaks the VM, I can restore this baseline.

<img width="828" height="806" alt="07-snapshot-list" src="https://github.com/user-attachments/assets/20ff680b-923d-49f8-bf94-aa53d12702f6" />

## ✅ Verification

| Test | Command | Result |
|------|---------|--------|
| Check IP address | `ip a` | `10.0.0.2/24` on `eth0` |
| Internet by IP | `ping -c 4 8.8.8.8` | Replies, 0% packet loss |
| Internet and DNS | `ping -c 4 google.com` | Replies, 0% packet loss |

<img width="713" height="758" alt="06-ping-test" src="https://github.com/user-attachments/assets/9f93a47a-570f-4d54-826e-0e1364cf4bf1" />

## 🐞 Problems and Solutions

### Problem 1: Connection would not activate after setting the static IP
`nmcli connection up` failed with:

```
Error: Connection activation failed: IP configuration could not be reserved
(no available address, timeout, etc.)
```

**Cause:** a known issue with VirtualBox 7 and newer Kali versions. The duplicate address detection (DAD) check times out.

**Fix:**
```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```
The connection then activated and the pings worked.

<img width="570" height="294" alt="08-error" src="https://github.com/user-attachments/assets/0e0eb486-160f-4ace-904e-bd1bbd30aedb" />

### Problem 2: Forgot my Kali login password
I was locked out of my own VM.

**Fix:** reset it from GRUB.
1. Restart the VM and press `e` at the GRUB menu.
2. Add `init=/bin/bash` to the end of the `linux` line and boot with `Ctrl+X`.
3. Run:
   ```bash
   mount -o remount,rw /
   ls /home            # find the real username
   passwd <username>
   exec /sbin/init
   ```

**Tips:** the GRUB menu only shows for about 2 seconds, so tap the Down arrow right after the VM starts to stop the countdown. My username was not the default `kali`, so `ls /home` was needed.

## 💡 What I Learned

1. **NAT vs NAT Network:** a NAT Network lets VMs reach each other and the internet, plain NAT does not.
2. **Static IPs:** how to set and verify address, netmask, gateway and DNS in Kali.
3. **Troubleshooting:** reading error messages and searching for the real cause led to both fixes.
4. **Snapshots:** take one before experimenting, so mistakes are cheap.
5. **Documentation:** screenshots and notes make a project repeatable and useful to others.

## 🛡️ Ethical Use

This lab is for learning only. Only test systems I own or have written permission to test.

## 🔗 Tools

- 7-Zip: https://7-zip.org/download.html
- VirtualBox: https://virtualbox.org/wiki/Downloads
- Kali Linux: https://kali.org/get-kali
