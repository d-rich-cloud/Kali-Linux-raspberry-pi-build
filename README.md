# Kali-Linux-raspberry-pi-build

## 🚀 Project Overview
This project documents the deployment of a stable, high-performance security research environment on the Raspberry Pi 5. 

**The Challenge:** Native Kali Linux images for the Pi 5 currently face a known firmware bug where the Wi-Fi radio defaults to "World Mode" (Country 99 / DFS-UNSET), rendering the wireless interface unable to scan or connect to local networks (specifically Starlink 5GHz/2.4GHz environments).

**The Solution:** A "Hybrid Build" using Raspberry Pi OS (64-bit) as the stable firmware/kernel base, with the full Kali Linux toolset layered on top via the `kali-rolling` repositories.

---

## 🛠️ Hardware Stack
*   **Device:** Raspberry Pi 5 (8GB RAM)
*   **Power:** Official Raspberry Pi 27W USB-C Power Supply (Required for full radio initialization)
*   **Storage:** 128GB MicroSD (Upgraded from 64GB to accommodate full toolset)
*   **Network:** Starlink Dual-Band (2.4GHz "Rosie" / 5GHz "home")

---

## 🔍 Troubleshooting Log: The "Country 99" Bug
During the initial build, the following symptoms were identified and resolved:

| Symptom | Diagnostic Command | Result |
| :--- | :--- | :--- |
| Wi-Fi Interface | `rfkill list` | Unblocked (Software & Hardware) |
| Network Scanning | `nmcli dev wifi list` | **Empty List** |
| Regulatory Domain | `sudo iw reg get` | `phy#0 country 99: DFS-UNSET` |
| Hardware Scan | `sudo iw dev wlan0 scan` | SSIDs visible (Hardware functional) |

### Root Cause Analysis
The Raspberry Pi 5's RP1 I/O controller enforces strict regulatory domains. If the firmware fails to handshake with the router's 802.11d beacon or lacks the correct local configuration at first boot, it locks the radio to prevent illegal frequency usage. Standard Linux commands like `iw reg set` are ignored by the firmware in this state.

---

## ⚡ The Build Process (Step-by-Step)

### 1. Base OS Installation
1. Flash **Raspberry Pi OS (64-bit)** using Raspberry Pi Imager.
2. **Critical:** Use the "Advanced Settings" (Gear icon) to pre-configure the Wi-Fi Country to **US** and enter SSID credentials. This ensures the RP1 controller initializes correctly on the first boot.

### 2. Filesystem Expansion
To ensure the 128GB card is fully utilized:
```bash
sudo raspi-config
# Navigate to: Advanced Options -> Expand Filesystem
df -h / # Verify capacity

## Using Kali Tools (Terminal-First Workflow)

Because this build uses **Raspberry Pi OS** as the base and layers Kali tools on top, not every Kali tool will appear in the desktop menus. Many Kali tools are **CLI-only by design** (they launch in a terminal, not as GUI apps).

### Verify tools are installed
Use `which` to confirm a command exists:
```bash
which nmap
which msfconsole
which aircrack-ng

nmap --version
msfconsole --version
aircrack-ng --help

nmap scanme.nmap.org
msfconsole

dpkg -l | grep kali

apt search metasploit
apt search aircrack

lxterminal -e msfconsole
