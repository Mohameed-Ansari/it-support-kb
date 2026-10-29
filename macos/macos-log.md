# macOS Support Log

Running log of every Mac-side task done during the IT support lab programme.
Each entry notes what was done, how, what went wrong, and how it applies when supporting other people's Macs.
In Week 8 this log becomes the basis for `macos-quick-reference.md`.

**Entry format**
- **Task:** what I did
- **How:** steps, commands, menu paths
- **Gotcha:** what went wrong or surprised me (skip if nothing)
- **Helpdesk use:** when a user's Mac would need this

---

## 2026-10-28 — Week 0, Day 1

### Checked Mac hardware before choosing a hypervisor
- **Task:** Identified chip architecture, RAM and free disk space.
- **How:** Apple menu → About This Mac; or in Terminal:
  - `uname -m` → `arm64` = Apple Silicon, `x86_64` = Intel
  - `sysctl -n hw.memsize` → RAM in bytes
  - `df -h ~` → free disk space
- **Helpdesk use:** First check before installing software, VMs or large updates; it decides which installer build to download.

### Installed UTM (hypervisor)
- **Task:** Installed UTM to run Windows Server, Windows 11 and Ubuntu VMs.
- **How:** Downloaded from the UTM website, moved to Applications, approved the first-launch prompt.
- **Gotcha:** On Apple Silicon, ARM guests run virtualized (fast); x64 guests such as Windows Server run emulated (slow).
- **Helpdesk use:** Explaining why x64-only software won't run, or runs slowly, on Apple Silicon Macs.

### Learned Mac keyboard equivalents inside Windows VMs
- **Task:** Sent Windows key combinations from a Mac keyboard.
- **How:** Option = Alt; Fn + Delete = forward Delete; Control + Option + Fn + Delete = Ctrl+Alt+Del (with the VM window active).
- **Gotcha:** Ctrl+Alt+Del did not reach the VM through UTM; worked around it by disabling the requirement offline in the lab only.
- **Helpdesk use:** Mac users connecting to Windows machines via RDP or VMs.

### Took screenshots for documentation
- **How:** Command + Shift + 4, then drag over an area (Command + Shift + 4, then Space, captures a single window).
- **Helpdesk use:** Asking users to send a screenshot of an error.

---

## 2026-10-29 — Week 0, Day 2

### Installed Docker Desktop
- **Task:** Installed Docker Desktop (Apple Silicon build) to host GLPI in Week 2.
- **How:** <!-- fill in: download source, permissions or prompts approved -->
- **Helpdesk use:** Developer-machine support; approving privileged helper prompts.

### Installed Wireshark
- **Task:** Installed Wireshark for packet captures.
- **How:** <!-- fill in: installer used; whether ChmodBPF / capture-permission prompt appeared -->
- **Helpdesk use:** Capture permissions on macOS need an extra component; without it, Wireshark opens but shows no interfaces.

### Installed Cisco Packet Tracer
- **How:** Downloaded via a Cisco Networking Academy account. <!-- fill in: any security prompt -->

### Installed Homebrew
- **Task:** Installed the Homebrew package manager.
- **How:** Ran the install script from brew.sh, then added it to PATH (Apple Silicon path):
  - `echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile`
  - `eval "$(/opt/homebrew/bin/brew shellenv)"`
- **Gotcha:** Apple Silicon installs to `/opt/homebrew`; Intel Macs use `/usr/local`.
- **Helpdesk use:** "Command not found" right after installing a tool usually means PATH wasn't updated.

### Diagnosed failed `brew install qemu` — disk full
- **Symptom:** `curl: (56) Failure writing output to destination`; downloads reached 34–64% and then failed.
- **Cause:** Not a network problem; the Mac had run out of disk space.
- **Resolution:** Checked with `df -h ~`, deleted unused ISOs, emptied the Trash, ran `brew cleanup`, then retried.
- **Helpdesk use:** Read the error before blaming the network. "Failure writing" points at disk space or permissions.

### Located UTM VM files
- **Path:** `~/Library/Containers/com.utmapp.UTM/Data/Documents/<VM>.utm/Data/`
- **Gotcha:** `~/Library` is hidden in Finder; open it with Go → Go to Folder (Command + Shift + G).
- **Helpdesk use:** Finding app data in sandboxed containers when a user asks where an app keeps its files.

### Shared a folder with an Ubuntu VM
- **Task:** Created `~/Lab/share` and shared it with UBU-01 via VirtFS.
- **Gotcha:** Shared a dedicated folder only, not the whole home folder (least privilege).

---

<!-- New entries go below. One entry per task; keep it short. -->
