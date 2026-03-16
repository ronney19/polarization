# Bluetooth Phone Communication on NanoPi Neo Core (Ubuntu 16.04)
## CSR 4.0 USB Dongle — Offline Packages, Source Code & References

This document lists **offline packages**, **source code**, **specifications**, and **references** needed to realize phone communication (calls, audio) by connecting a CSR 4.0 USB Bluetooth dongle to a NanoPi Neo Core running Ubuntu 16.04.

---

## 1. Hardware & Kernel Support

### CSR 4.0 USB Dongle
- **Chip**: Often **CSR8510 A10** (Bluetooth 4.0). Linux uses the **btusb** (USB Bluetooth) driver in the kernel.
- **Kernel driver**: `drivers/bluetooth/btusb.c` — no separate vendor driver needed for basic operation.
- **Vendor IDs**: CSR dongles typically use `0a12` (Cambridge Silicon Radio). Kernel already has IDs in `btusb.c`.

### NanoPi Neo Core
- **SoC**: Allwinner H5 (ARM Cortex-A53).
- **Ubuntu 16.04**: Use **arm64** or **armhf** images for Neo Core. Ensure kernel is ≥ 4.x for good Bluetooth 4.0 support.

**Offline kernel / driver reference (source)**  
- Kernel Bluetooth subsystem:  
  `https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git`  
  Paths: `net/bluetooth/`, `drivers/bluetooth/`  
- For offline: clone or download the kernel tree and use `net/bluetooth` and `drivers/bluetooth`.

---

## 2. Software Stack Overview

```
[Phone] <--Bluetooth--> [CSR Dongle] <--USB--> [NanoPi Neo Core]
                              |
                    BlueZ (bluetoothd, D-Bus)
                              |
              +---------------+---------------+
              |               |               |
           ofono          PulseAudio    (optional: ModemManager)
        (phone/modem)     (audio I/O)
```

- **BlueZ**: Bluetooth stack (HFP, A2DP, etc.).
- **ofono**: Telephony daemon; implements modem/phone logic and works with BlueZ HFP.
- **PulseAudio**: Audio routing (headset, speaker, mic) for HFP/A2DP.

---

## 3. Offline Debian/Ubuntu Packages (Ubuntu 16.04)

Install (or download for offline use with `apt-get download` / `apt-offline`) on **arm64** or **armhf** as appropriate.

### Core Bluetooth
```text
bluez
bluez-tools
libbluetooth3
libbluetooth-dev
```

### Telephony (phone/modem over Bluetooth)
```text
ofono
ofono-scripts
```

### Audio
```text
pulseaudio
pulseaudio-module-bluetooth
pulseaudio-utils
```

### Build / development (if compiling from source)
```text
build-essential
libdbus-1-dev
libglib2.0-dev
libical-dev
libreadline-dev
libudev-dev
libdbus-glib-1-dev
libcap-ng-dev
libell-dev
```

### Optional (utilities / testing)
```text
bluetooth
rfkill
dbus
```

**Offline install method (on a machine with internet)**  
```bash
# Use a directory _apt can write to (e.g. /tmp) to avoid sandbox warnings; then move .deb to desired location
cd /tmp
apt-get download bluez bluez-tools libbluetooth3 libbluetooth-dev ofono pulseaudio pulseaudio-module-bluetooth
mv *.deb /root/   # or your target directory
# Copy .deb files to NanoPi and: sudo dpkg -i *.deb
```

Use **apt-offline** or **apt-get download** with dependency resolution for a full offline bundle.

---

## 4. Source Code Repositories

### BlueZ (Linux Bluetooth stack)
- **URL**: https://git.kernel.org/pub/scm/bluetooth/bluez.git  
- **Mirror**: https://github.com/bluez/bluez  
- **Relevant for phone**: `profiles/audio/`, `profiles/network/`, `src/`, HFP in `profiles/audio/hfp*.c`.
- **Tag for Ubuntu 16.04 era**: e.g. `bluez-5.37` or match the version in Xenial (e.g. 5.37).

### ofono (telephony daemon, modem/phone over Bluetooth)
- **URL**: https://git.kernel.org/pub/scm/network/ofono/ofono.git  
- **Mirror**: https://kernel.googlesource.com/pub/scm/network/ofono/ofono  
- **Relevant**: HFP driver in `drivers/`, D-Bus API for calls/SMS.

### PulseAudio (audio routing, Bluetooth A2DP/HFP)
- **URL**: https://gitlab.freedesktop.org/pulseaudio/pulseaudio  
- **Relevant**: `src/modules/bluetooth/` for Bluetooth audio.

### Linux kernel (Bluetooth subsystem and btusb)
- **URL**: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git  
- **Paths**: `net/bluetooth/`, `drivers/bluetooth/btusb.c`.

**Offline**: Clone each repo (or download ZIP/tarball from GitHub/GitLab) and transfer to the NanoPi for reference or local builds.

---

## 5. Specifications & Technology Papers / References

### Bluetooth SIG (profiles used for “phone communication”)
- **HFP (Hands-Free Profile)**  
  - Used for: voice calls (headset/hands-free).  
  - Spec: https://www.bluetooth.com/specifications/specs/hands-free-profile-1-6/  
  - PDF: search “HFP 1.6” or “Hands-Free Profile” on bluetooth.com.
- **HSP (Headset Profile)**  
  - Simpler headset profile; often used with HFP.  
  - Spec: https://www.bluetooth.com/specifications/specs/headset-profile-1-2/
- **A2DP (Advanced Audio Distribution Profile)**  
  - High-quality audio (e.g. music); often used alongside HFP for media.  
  - Spec: https://www.bluetooth.com/specifications/specs/advanced-audio-distribution-profile-1-3/
- **AVRCP (Audio/Video Remote Control)**  
  - Optional; remote control (play/pause) for A2DP.

### Core Bluetooth (baseband / host)
- **Core Specification**: https://www.bluetooth.com/specifications/specs/core-specification/  
  - Version 4.0 matches your CSR 4.0 dongle.

### Linux/stack documentation
- **BlueZ**:  
  - https://git.kernel.org/pub/scm/bluetooth/bluez.git/tree/doc  
  - D-Bus API: `doc/*.txt` in the BlueZ tree (e.g. `api.txt`, `media-api.txt`).
- **Kernel Bluetooth**:  
  - https://www.kernel.org/doc/html/latest/networking/bluetooth.html  
  - (Download kernel docs tarball for offline.)
- **ofono**:  
  - https://ofono.org/documentation.html  
  - D-Bus API and architecture docs.

### Papers / application notes (conceptual)
- **“Bluetooth Protocol Stack”** — overview of host stack (e.g. BlueZ) vs controller (dongle).
- **“Hands-Free Profile Implementation”** — application notes from Bluetooth SIG or embedded Linux conferences (e.g. ELC).
- **Linux Plumbers Conference / ELC** — search “BlueZ”, “ofono”, “Bluetooth HFP” for slides/talks.

### How to download these (offline)

**BlueZ documentation** (D-Bus API, profile docs):
```bash
git clone https://git.kernel.org/pub/scm/bluetooth/bluez.git
# Docs are in bluez/doc/ (e.g. api.txt, media-api.txt, adapter-api.txt)
# Or: git clone --depth 1 https://git.kernel.org/pub/scm/bluetooth/bluez.git
```

**Kernel Bluetooth documentation** (HTML):
- Full kernel source tarball: `https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.x.x.tar.xz` — then use `Documentation/networking/bluetooth.rst`.
- Or clone kernel and read RST in tree:
```bash
git clone --depth 1 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
# See linux/Documentation/networking/bluetooth.rst
```

**ofono documentation** (web page + in-tree):
- ofono source already has README and D-Bus interfaces in code. Save the website for offline:
```bash
wget -p -k -E https://ofono.org/documentation.html
# Creates ofono.org/ with documentation.html and assets
```

**Bluetooth SIG specs (HFP, A2DP, Core)** — PDFs:
- Open https://www.bluetooth.com/specifications/specs/ , choose each spec, use "Download" / "Get PDF" on the spec page.
- Or inspect the page for the PDF URL and: `wget <pdf_url>`.

**Papers / talks** (no single bundle):
- Events: https://events.linuxfoundation.org/ (ELC, Plumbers) — search "BlueZ", "ofono", "Bluetooth"; download session PDFs/slides from session pages.
- Or use browser "Save as PDF" / "Save page as" for HTML talks.

---

## 6. Minimal Offline Checklist

| Item | Purpose |
|------|--------|
| **Kernel** ≥ 4.x with `CONFIG_BT_USB`, `CONFIG_BT_HCIBTUSB` | CSR dongle (btusb) |
| **bluez** (≈ 5.37 for Xenial) | HFP/A2DP, D-Bus API |
| **ofono** | Phone/modem logic over HFP |
| **pulseaudio** + **pulseaudio-module-bluetooth** | Audio in/out for calls |
| **BlueZ source** | Debugging, patches, HFP logic |
| **ofono source** | Telephony logic, HFP driver |
| **HFP 1.6 spec (PDF)** | Protocol behavior, AT commands |
| **Bluetooth Core 4.0 spec** | Baseband/controller reference |

---

## 7. Quick Test Commands (after packages are installed)

```bash
# List USB devices (confirm CSR dongle is seen)
lsusb

# Load Bluetooth kernel modules if needed
sudo modprobe btusb

# Start/restart Bluetooth
sudo systemctl start bluetooth
sudo systemctl start ofono

# Scan and pair (replace XX:XX:XX with phone address)
bluetoothctl
power on
scan on
pair XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
```

---

## 8. Summary

- **Hardware**: CSR 4.0 USB dongle is supported by the mainline kernel **btusb** driver; no extra vendor driver needed.
- **Offline packages**: **bluez**, **ofono**, **pulseaudio**, **pulseaudio-module-bluetooth**, plus dependencies (libbluetooth, dbus, etc.); use `apt-get download` or apt-offline for offline install.
- **Source code**: **BlueZ**, **ofono**, **PulseAudio**, and **kernel** (net/bluetooth, drivers/bluetooth) from the URLs above.
- **References**: **HFP 1.6**, **HSP**, **A2DP**, **Bluetooth Core 4.0** from bluetooth.com; **BlueZ** and **ofono** docs for D-Bus and architecture.

With these packages, sources, and specs you can realize phone communication (voice call over Bluetooth) on NanoPi Neo Core with Ubuntu 16.04 and a CSR 4.0 USB dongle, fully from offline resources once everything is downloaded.
