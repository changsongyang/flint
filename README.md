# 🌀 Flint — KVM Management, Reimagined

<p align="center">
  <img src="https://i.ibb.co/yj2bFZG/flint-banner.jpg" alt="Flint Logo" width="300"/>
</p>

<p align="center">
  <strong>
    A single &lt;11MB binary with a modern Web UI, CLI, and API for KVM.
    <br/>No XML. No bloat. Just VMs.
  </strong>
</p>

<p align="center">
  <a href="https://github.com/volantvm/flint/releases/latest">
    <img src="https://img.shields.io/github/v/release/volantvm/flint" alt="Latest Release">
  </a>
  <a href="https://github.com/volantvm/flint/blob/main/LICENSE">
    <img src="https://img.shields.io/github/license/volantvm/flint" alt="License">
  </a>
  <a href="https://github.com/volantvm/flint/actions/workflows/release.yml">
    <img src="https://img.shields.io/github/actions/workflow/status/volantvm/flint/.github/workflows/release.yml" alt="Build Status">
  </a>
</p>

---
![Flint Dashboard](https://i.ibb.co/wN9H8WKX/Screenshot-2025-09-07-at-3-51-58-AM.png)
![Flint Library](https://i.ibb.co/Z1k9XBqQ/Screenshot-2025-09-08-at-4-59-46-AM.png)


Flint is a modern, self-contained KVM management tool built for developers, sysadmins, and home labs who want zero bloat and maximum efficiency. It was built in a few hours out of a sudden urge for something better.

---

### 📋 Prerequisites

**System Requirements:**
- Linux host (Debian, Ubuntu, Fedora, RHEL, Arch, etc.)
- libvirt >= 6.10.0 (check with `libvirtd --version`)
- QEMU/KVM virtualization support

**Required Packages:**

<details>
<summary>Debian/Ubuntu</summary>

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-daemon libvirt-clients bridge-utils
sudo systemctl enable --now libvirtd
```
</details>

<details>
<summary>RHEL/Fedora/CentOS</summary>

```bash
sudo dnf install -y qemu-kvm libvirt libvirt-client virt-install
sudo systemctl enable --now libvirtd
```
</details>

<details>
<summary>Arch Linux</summary>

```bash
sudo pacman -S qemu-full libvirt virt-install virt-manager
sudo systemctl enable --now libvirtd
```
</details>

**Note:** If you encounter `libvirt-lxc.so.0: cannot open shared object file`, install the LXC library:
```bash
# Debian/Ubuntu
sudo apt install -y libvirt-daemon-driver-lxc

# RHEL/Fedora
sudo dnf install -y libvirt-daemon-lxc

# Arch
sudo pacman -S libvirt-lxc
```

**Platform Compatibility:**

Flint is built with CGO (libvirt-go bindings) and requires glibc. **Alpine Linux and musl-based systems are not directly supported.**

<details>
<summary>Running Flint on Alpine Linux</summary>

Since Flint requires glibc and Alpine uses musl, you have two options:

**Option 1: Use gcompat (glibc compatibility layer)**
```bash
# Install gcompat on Alpine
apk add gcompat libstdc++

# Download and run Flint
curl -LO https://github.com/volantvm/flint/releases/latest/download/flint-linux-amd64
chmod +x flint-linux-amd64
./flint-linux-amd64 serve
```

**Option 2: Run in a glibc-based container (recommended)**
```bash
# Use Debian/Ubuntu container on Alpine host
docker run -d \
  --name flint \
  --privileged \
  -v /var/run/libvirt:/var/run/libvirt \
  -p 5550:5550 \
  debian:bookworm-slim \
  bash -c "apt update && apt install -y libvirt-clients && ./flint serve"
```

**Why no native musl support?**
- Flint uses CGO extensively through libvirt-go bindings
- Static linking with libvirt is extremely complex due to numerous dependencies
- Cross-compiling CGO code for musl requires a complete musl toolchain
- The maintenance burden for musl support would be significant

For production use on Alpine, we recommend running Flint in a glibc-based container or using a glibc-based Linux distribution.
</details>

---

### 🚀 One-Liner Install

```bash
curl -fsSL https://raw.githubusercontent.com/volantvm/flint/main/install.sh | bash
```
*Auto-detects OS/arch, installs to `/usr/local/bin`, and prompts for web UI passphrase setup.*

---

### 🔐 Security & Authentication

Flint implements a multi-layered security approach:

**Web UI Security:**
- **Passphrase Authentication**: Web interface requires a passphrase login
- **Session-Based**: Secure HTTP-only cookies with 1-hour expiry
- **No API Key Exposure**: Web UI never exposes API keys to browsers

**API Security:**
- **Bearer Token Authentication**: CLI and external tools use API keys
- **Protected Endpoints**: All API endpoints require authentication
- **Flexible Access**: Support for both session cookies and API keys

**Authentication Flow:**
```bash
# First run - set passphrase
flint serve
# 🔐 No web UI passphrase set. Let's set one up for security.
# Enter passphrase: ********

# Web UI access
# Visit http://your-server:5550 → Enter passphrase → Full access

# CLI access (uses API key)
flint vm list --all

# External API access
curl -H "Authorization: Bearer YOUR_API_KEY" http://localhost:5550/api/vms
```

---

### ✨ Core Philosophy

-   🖥️ **Modern UI** — A beautiful, responsive Next.js + Tailwind interface, fully embedded.
-   ⚡ **Single Binary** — No containers, no XML hell. A sub-8MB binary is all you need.
-   🛠️ **Powerful CLI & API** — Automate everything. If you can do it in the UI, you can do it from the command line or API.
-   📦 **Frictionless Provisioning** — Native Cloud-Init support and a simple, snapshot-based template system.
-   🔐 **Secure by Default** — Multi-layered authentication with passphrase protection.
-   💪 **Non-Intrusive** — Flint is a tool that serves you. It's not a platform that locks you in.

---

### 🏎️ Quickstart

**1. Start the Server**
```bash
# Interactive setup (recommended for first run)
flint serve --set-passphrase

# Or set passphrase directly
flint serve --passphrase "your-secure-password"

# Or use environment variable
export FLINT_PASSPHRASE="your-secure-password"
flint serve
```
*On first run, you'll be prompted to set a web UI passphrase for security.*
*   **Web UI:** `http://localhost:5550` (requires passphrase login)
*   **API:** `http://localhost:5550/api` (requires authentication)

**2. Web UI Access**
- Visit `http://localhost:5550`
- Enter your passphrase to access the management interface
- All API calls are automatically authenticated via session

**3. CLI Usage**
```bash
# VM Management
flint vm list                    # List all VMs
flint vm launch my-server        # Create and start a VM
flint vm ssh my-server          # SSH into a VM

# Cloud Images
flint image list                 # Browse cloud images
flint image download ubuntu-24.04 # Download an image

# Networks & Storage
flint network list               # List networks
flint storage volume list default # List storage volumes
```

**4. API Access (for external tools)**
```bash
# Get your API key (requires authentication)
curl -H "Authorization: Bearer YOUR_API_KEY" http://localhost:5550/api/vms
```
---

### 📖 Full Documentation

Complete CLI commands, API reference, and advanced usage:

➡️ **[docs.md](docs.md)** - Complete CLI & API Documentation

---

### 🔧 Tech Stack

-   **Backend:** Go 1.25+
-   **Web UI:** Next.js + Tailwind + Bun
-   **KVM Integration:** libvirt-go
-   **Binary Size:** ~11MB (stripped)

---

<p align="center">
  <b>🚀 Flint is young, fast-moving, and designed for builders.<br/>
  Try it. Break it. Star it. Contribute.</b>
</p>
