# RK35xx Kernel Patch: CVE-2026-31431 ("Copy Fail") 
RK35xx CopyFail Hotfix: CVE-2026-31431 Patch for Ubuntu 24.04

A community-provided, pre-compiled kernel patch for **all RK35xx single-board computers (Rock 5, Orange Pi 5, NanoPC, etc.)** running Joshua Riek's Ubuntu **24.04** (Kernel 6.1.0-1025-rockchip).

## 🚨 The Vulnerability
On April 29, 2026, a high-severity local privilege escalation (LPE) vulnerability known as **"Copy Fail" (CVE-2026-31431)** was disclosed. The flaw resides in the kernel's cryptographic subsystem (`crypto/algif_aead.c`). By chaining an `AF_ALG` socket operation with the `splice()` system call, an unprivileged local user can force a zero-copy write directly into the kernel's page cache, allowing them to overwrite setuid-root binaries in memory and gain instant root access.

Due to the kernel configuration (`CONFIG_CRYPTO_USER_API_AEAD=y`), standard `modprobe` mitigation strategies **do not work** on this build. A source-level kernel patch is required.

## 🛠 Why This Repository?
With the upstream Joshua Riek repository currently inactive, users are left stranded on a vulnerable kernel.<br>
As long-time supporters of the SBC and edge AI community, Qengineering has manually backported the mainline Linux fix (commit `a664bf3d603d`), recompiled the kernel from the exact `Ubuntu-rockchip-6.1.0-1025.25` source tag, and packaged it here for immediate deployment.

**All original NPU, VPU, and networking hardware acceleration features are 100% preserved.**

## 📦 Installation Instructions

Joshua Riek's Ubuntu images use a strictly customized U-Boot environment that requires raw (uncompressed) kernel images and specific device tree mapping. We have handled the compilation; you just need to run the following commands to install the patch.

**1. Download the required files:**
Download these **three** files from the [Releases page](#) to your board:
* `linux-image-6.1.75-copyfail-patched_*.deb`
* `linux-headers-6.1.75-copyfail-patched_*.deb`
* `Image` (The raw, uncompressed kernel)

**2. Open a terminal in the folder containing your downloads and run this command block:**

```bash
# 1. Install the Debian packages (This configures modules and initramfs)
sudo dpkg -i linux-image-6.1.75-copyfail-patched_*.deb
sudo dpkg -i linux-headers-6.1.75-copyfail-patched_*.deb

# 2. Overwrite the compressed kernel with the raw uncompressed Image required by U-Boot
sudo cp Image /boot/vmlinuz-6.1.75-copyfail-patched

# 3. Port over your existing hardware device trees
sudo mkdir -p /lib/firmware/6.1.75-copyfail-patched/
sudo cp -r /lib/firmware/6.1.0-1025-rockchip/device-tree /lib/firmware/6.1.75-copyfail-patched/

# 4. Regenerate the bootloader configuration
sudo u-boot-update

# 5. Reboot your system
sudo reboot
```

🔍 Transparency: Build It Yourself

For users who prefer not to install pre-compiled binaries, you can reproduce this exact build on your own Rock 5C. Due to Rockchip SDK modifications, the upstream mainline patch does not apply cleanly via git, requiring a manual code edit.
