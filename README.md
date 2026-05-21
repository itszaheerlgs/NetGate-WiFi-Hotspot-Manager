# 🌐 NetGate — WiFi Hotspot Manager

A time-based WiFi voucher system for Windows. Sell internet access by the minute — customers connect to your hotspot, open the captive portal, enter a voucher code, and get online. No voucher = no internet.

Built with Python, CustomTkinter, SQLite, and Windows Firewall.

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python) ![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey?logo=windows) ![License](https://img.shields.io/badge/License-MIT-green)

---

## 📸 Features

- **Voucher-based access** — generate time-limited codes (15 min → 24 hrs) and sell them to customers
- **Captive portal** — customers who connect to your WiFi are redirected to `http://192.168.137.1` to enter their code
- **Firewall enforcement** — unpaid devices are blocked from the internet; only the portal is reachable until they pay
- **Pause & resume** — customers can pause their session and resume later without losing time
- **QR code scanner** — customers can scan their voucher QR code directly from the portal using their phone camera
- **Printable vouchers** — generate and print physical voucher cards with QR code, WiFi name, and password
- **Device manager** — see all connected devices, their MAC addresses, and manually allow/block them
- **Dashboard** — live stats: total, unused, active, paused, and expired vouchers at a glance
- **Event log** — full audit trail of activations, blocks, and system events
- **LCD display tab** — scrolling display for a connected LCD/secondary screen
- **Print queue** — manage and reprint voucher batches
- **Settings** — customize SSID, password, voucher prefix, and portal IP

---

## 🖥️ Requirements

- Windows 10 or 11
- Python 3.10+
- A WiFi adapter that supports Windows Mobile Hotspot
- Run as **Administrator**

Install dependencies:

```bash
pip install customtkinter qrcode[pil] pillow
```

---

## 🚀 Quick Start

1. **Enable Windows Mobile Hotspot first**
   Go to `Settings → Network & Internet → Mobile Hotspot` and turn it ON.
   Set the network name and password to match what's in the app settings (default: `NetGate-WiFi` / `netgate123`).

2. **Run as Administrator**
   Right-click `wifi_hotspot_manager.py` → **Run as administrator**
   (Required for firewall rules and binding to port 80.)

3. **Click "Start Portal"** in the sidebar.
   Skip the "Start Hotspot" button — use the Windows Settings toggle instead.

4. **Issue a voucher**
   Go to the **Issue Voucher** tab, pick a duration, and generate a code.

5. **Customer flow**
   - Customer connects to your WiFi (`NetGate-WiFi`)
   - Opens `http://192.168.137.1` in their browser
   - Enters the voucher code → internet access starts
   - Timer counts down; when it expires, they're blocked again

---

## 📁 File Structure

```
wifi_hotspot_manager.py   # Main application (single file)
hotspot_manager.db        # SQLite database (auto-created on first run)
jsqr.min.js               # QR scanner library (auto-downloaded on first portal start)
```

---

## 🔒 How Blocking Works

When a device connects without a valid voucher, four Windows Firewall rules are applied to its IP:

| Rule | Direction | Action | Purpose |
|------|-----------|--------|---------|
| `ALLOW_{ip}` | Inbound | Allow | Portal replies reach the device |
| `ALLOW_{ip}_out` | Outbound | Allow | Device can reach portal on port 80 |
| `BLOCK_{ip}` | Inbound | Block | All other internet traffic blocked |
| `BLOCK_{ip}_out` | Outbound | Block | All other outbound blocked |

Allow rules are processed before block rules by Windows Firewall, so the portal is always reachable while the internet is not. When a valid voucher is activated, all block rules for that device are removed.

---

## ⏱️ Session Durations

| Plan | Duration |
|------|----------|
| Quick | 15 minutes |
| Short | 30 minutes |
| 1 Hour | 60 minutes |
| 2 Hours | 120 minutes |
| 3 Hours | 180 minutes |
| 6 Hours | 360 minutes |
| 12 Hours | 720 minutes |
| 24 Hours | 1440 minutes |

Custom durations can be entered manually in the Issue Voucher tab.

---

## ⚙️ Configuration

Open the **Settings** tab in the app to change:

| Setting | Default | Description |
|---------|---------|-------------|
| SSID | `NetGate-WiFi` | Hotspot network name |
| Password | `netgate123` | Hotspot password |
| Voucher Prefix | `NGT` | Prefix for generated codes (e.g. `NGT-XXXX`) |
| Portal IP | `192.168.137.1` | Windows Mobile Hotspot gateway (don't change unless needed) |

---

## ❓ Troubleshooting

**Portal won't start**
Make sure the app is running as Administrator and Windows Mobile Hotspot is ON before clicking Start Portal.

**Customers can't reach `192.168.137.1`**
Confirm the hotspot is active and the portal is running (both status indicators in the sidebar should be green). Check that no other app (IIS, Apache) is using port 80.

**Firewall rules not applying**
The app must be run as Administrator. Without admin rights, `netsh advfirewall` commands will silently fail.

**Customers still have internet without a voucher**
Open Command Prompt as Admin and run:
```
netsh advfirewall firewall show rule name=all | findstr NetGate
```
If no rules appear, the program is not running as Administrator.

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

> Made for small-scale WiFi reselling — sari-sari stores, boarding houses, internet shops, and school setups.
