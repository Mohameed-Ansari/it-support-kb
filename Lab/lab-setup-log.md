# Home Lab — Setup Log

Build record for the IT support lab used throughout the 8-week programme.
Covers the host, hypervisor choice, VM inventory, lab network, and every problem hit during the build with its fix.

> Items marked `TODO` are still to be completed or confirmed.

---

## 1. Host

| Item | Value |
|---|---|
| Machine | Mac, Apple M4 (Apple Silicon, `arm64`) |
| RAM | `TODO` GB |
| Hypervisor | UTM (free; QEMU-based) |
| Other tools on host | Docker Desktop, Cisco Packet Tracer, Wireshark, Homebrew, `qemu-img` |

### Why UTM

- Microsoft does not ship Windows Server for ARM64, so the server must run as **x64 under emulation**.
- On Apple Silicon, VMware Fusion, Parallels and VirtualBox virtualize ARM guests only (Parallels' x86 emulation is experimental and paid).
- UTM is the only free option that runs both **ARM guests virtualized** and **x64 guests emulated**, and keeping all VMs in one hypervisor lets them share one private network.

---

## 2. VM inventory

| VM | OS | UTM mode | RAM | Cores | Disk | Status |
|---|---|---|---|---|---|---|
| DC01 | Windows Server 2025 Standard Evaluation, Server Core (build 26100) | Emulate (x86_64) | 4 GB | 4 | 60 GB | Installed |
| WIN11-01 | Windows 11 Pro ARM64 | Virtualize | 4 GB | 4 | 64 GB | `TODO` |
| UBU-01 | Ubuntu 26.04.1 ARM64 | Virtualize (QEMU backend) | 3 GB | 2 | 30 GB | Installed |

Run at most two VMs at a time (16 GB host). UBU-01 is only needed from Week 1, Day 5.

### DC01 hardware settings
- **CPU model:** Skylake-Client (the default emulated CPU may lack instructions Server 2025 needs)
- **Network cards:** Intel Gigabit Ethernet (e1000), recognised by Windows without extra drivers
- **System disk:** NVMe interface; **installation ISO:** IDE
- **Guest tools drive:** removed (see Issue 2)
- **Time zone:** (UTC+03:00) Kuwait, Riyadh
- **Diagnostic data:** Required only

---

## 3. Lab network

Every VM has two network cards:

| Card | UTM mode | Purpose | Addressing |
|---|---|---|---|
| 1 | Shared Network | Internet access (macOS NAT) | DHCP from macOS, 192.168.64.0/24, gateway 192.168.64.1 |
| 2 | Host Only | Private lab network | Static, 10.10.10.0/24, **no gateway** |

| VM | Lab IP (card 2) | Card 1 MAC | Card 2 MAC |
|---|---|---|---|
| DC01 | 10.10.10.10 | EE:F1:CE:9F:A2:C1 | 1A:FD:B2:DE:4E:A9 |
| WIN11-01 | 10.10.10.21 | `TODO` | `TODO` |
| UBU-01 | 10.10.10.30 | …:36:B3 | …:AA:FD |

**Known limitation:** macOS runs its own DHCP server on the Host Only network (hands out 192.168.128.0/24 from 192.168.128.1). Static lab addresses work alongside it, but it will compete with DC01 when DC01 becomes the DHCP server in Week 3. Workaround to be decided in Week 3.

### Connectivity test
- [ ] WIN11-01 → `ping 10.10.10.10` (DC01)
- [ ] DC01 → `ping 10.10.10.21` (WIN11-01)
- [ ] UBU-01 → `ping 10.10.10.10`
- [ ] Each VM reaches the internet through card 1

---

## 4. Backups

UTM has no snapshot manager, so disk snapshots are taken with `qemu-img` (installed via Homebrew) with the VM shut down:

```bash
cd ~/Library/Containers/com.utmapp.UTM/Data/Documents/<VM>.utm/Data/
qemu-img snapshot -c clean-install <disk>.qcow2   # create
qemu-img snapshot -l <disk>.qcow2                 # list
qemu-img snapshot -a clean-install <disk>.qcow2   # revert
```
Alternative: right-click the VM in UTM → Clone (full copy, uses more space).

| VM | Snapshot `clean-install` |
|---|---|
| DC01 | `TODO` |
| WIN11-01 | `TODO` |
| UBU-01 | `TODO` |

---

## 5. Problems and fixes

### Issue 1 — Wrong ISO architecture
- **Symptom:** Downloaded `Windows11_Client_x64…iso` and `ubuntu-26.04.1-desktop-amd64.iso`.
- **Cause:** `x64` / `amd64` images would run emulated (very slow) on Apple Silicon.
- **Fix:** Re-downloaded the **ARM64** images. Only Windows Server stays x64, by design.
- **Lesson:** Check the architecture in the filename before building anything.

### Issue 2 — "Setup has failed to validate the product key" (DC01)
- **Symptom:** Windows Server 2025 Setup asked for a product key; choosing "I don't have a product key" ended in this error.
- **Cause:** UTM's guest tools drive, attached by the "Install drivers and SPICE tools" option, carries an answer file meant for Windows client, which Server Setup picked up.
- **Fix:** Shut down, deleted the guest tools drive, set the system disk interface to NVMe (so no extra storage driver is needed), and used **"Previous version of Setup"**.

### Issue 3 — Ctrl+Alt+Del not reaching the VM
- **Symptom:** "Press Ctrl+Alt+Del to unlock"; Control + Option + Fn + Delete had no effect.
- **Notes:** UTM's "capture" notice covers the mouse; the keyboard already reaches the VM, but this combination did not.
- **Options:** full-screen mode; check UTM input settings ("Option as Meta" off); last resort, disable the requirement offline from the Server ISO's recovery command prompt:
  ```
  reg load HKLM\OFF D:\Windows\System32\config\SOFTWARE
  reg add "HKLM\OFF\Microsoft\Windows\CurrentVersion\Policies\System" /v DisableCAD /t REG_DWORD /d 1 /f
  reg unload HKLM\OFF
  ```
  (Drive letter may differ in the recovery environment. Lab only — this weakens a real security control.)
- **Resolution used:** `TODO`

### Issue 4 — Both network cards on the same network (DC01)
- **Symptom:** `Ethernet` and `Ethernet 2` both received 192.168.64.x from 192.168.64.1.
- **Cause:** Second card left on Shared Network; later both were set to Host Only.
- **Fix:** Matched each card by **MAC address** in UTM settings and in `ipconfig /all`; set card 1 to Shared Network and card 2 to Host Only.
- **Lesson:** Identify network cards by MAC, not by their order in the list.

### Issue 5 — VM clock wrong after first boot (DC01)
- **Symptom:** DHCP lease showed "Lease Obtained: Monday, September 22, 1890".
- **Cause:** Emulated hardware clock not set correctly at boot.
- **Fix:** Checked with `Get-Date`; resynced with `w32tm /resync /force` (or `Set-Date`).
- **Why it matters:** Kerberos fails when clocks differ by more than 5 minutes, which will break Active Directory in Week 3. Recheck after reboots.

### Issue 6 — `brew install qemu` failed: disk full
- **Symptom:** `curl: (56) Failure writing output to destination` while downloads were 34–64% complete.
- **Cause:** Host disk out of space (not a network problem).
- **Fix:** Shut down all VMs (a full disk can corrupt a running VM's disk image), deleted the wrong ISOs, emptied the Trash, ran `brew cleanup`, retried.

### Issue 7 — Copying text out of Server Core
- **Cause:** No clipboard sharing without the guest tools.
- **Workaround:** Screenshots (Command + Shift + 4); inside the VM, `<command> | Set-Clipboard`. Guest tools can be re-added after installation if clipboard sharing is needed.

---

## 6. Open items

- [ ] Build WIN11-01 and record its MACs
- [ ] Set static lab IPs on all three VMs and pass the connectivity test
- [ ] Take `clean-install` snapshots of all VMs
- [ ] Week 3: decide how to stop macOS DHCP competing with DC01 on the lab network
