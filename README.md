# 🛠️ San Kernel CI & Defconfig Workflows

This repository contains GitHub Actions workflows designed to automate the building, packaging, and configuration management of **San Kernel** for Xiaomi SDM660 devices (specifically **Lavender** and **Whyred**).

---

## 🚀 1. Build Kernel (`build.yml`)

The primary automation script used to compile the custom Android kernel, package it into an AnyKernel3 flashable ZIP, and dispatch status notifications straight to Telegram.

### ✨ Key Features & Inputs
When triggering this workflow manually via the GitHub Actions tab, you can configure:
* **👤 Telegram Username:** Your username to credit the build notification.
* **📱 Select Device:** Choose between target devices (`lavender` or `whyred`).
* **🌿 Select Kernel Branch:** Pick the source branch (e.g., `PLTS`, `st74`, `st78`, `ksun-33024`, `ksun-33068`, `resukisu`, `resukisu-susfs`).
* **🏷️ Custom Kernel Name (`LOCALVERSION`):** Define a custom string suffix for your kernel version (e.g., `MyKernel-v1.0`).
* **🦊 KernelSU Injection (`INJECT_KSU`):** Option to integrate KernelSU variants (`kowsu`, `xxksu`, or `none`).
* **📳 QTI Haptics:** Toggle support for advanced vibrator drivers.
* **📷 Build Newcam:** Option to use Xiaomi's new camera blobs.
* **📦 Flashable Type:** Choose between `original` (GZIP compression / standard zip layout) or `san-kernel` (XZ compression).
* **🛠️ Compiler:** Select your preferred toolchain (`zyc` Clang or `proton` Clang).

### ⚙️ What the Workflow Does
1. **Source Synchronization:** Clones the San Kernel source repository based on your selected branch.
2. **Environment Setup:** Installs required build dependencies and configures **ccache** and **Clang toolchain caching** to significantly speed up compilation times.
3. **Kernel Configuration:** Applies device-specific defconfigs, injects KernelSU if requested, and formats version/local version strings.
4. **Compilation:** Compiles the kernel using multiple CPU threads (`nproc --all`) with optimized LLVM/Clang toolchains.
5. **Packaging:** Automatically packages the compiled image into an **AnyKernel3** flashable ZIP file.
6. **Telegram Notification:** Uploads the final flashable `.zip` file directly to the designated Telegram group chat if successful, or forwards the `build.log` file if an error occurs.

---

## ⚙️ 2. Optimize Defconfig (`optimize.yml`)

A utility workflow designed to automatically fine-tune, clean up, and push configuration adjustments directly to specific branches of the kernel source repository.

### ✨ Key Features & Inputs
* **👤 Telegram Username:** Your identifier for tracking who ran the optimization.
* **📱 Select Device:** Target device defconfig to modify (`lavender` or `whyred`).
* **🌿 Select Kernel Branch:** Branch where the updated configuration will be committed and pushed.
* **🔑 GitHub Personal Access Token (PAT):** Requires a valid token with repository write permissions to commit changes back upstream.

### ⚙️ What the Workflow Does
1. **Repository Checkout:** Clones the repository using the provided Personal Access Token on the specified target branch.
2. **Defconfig Tweaks:** Uses kernel configuration utilities (`./scripts/config`) to streamline performance parameters:
   * Disables unnecessary or power-heavy CPU governors (`powersave`, `userspace`, `ondemand`, `conservative`).
   * Sets **Deadline** as the default I/O scheduler.
   * Strips out heavy debugging and tracing overhead (`FTRACE`, `TRACING`, `SCHED_DEBUG`, etc.) to reduce bloat and latency.
   * Forces a **1000Hz timer tick (`HZ_1000`)** for improved system responsiveness.
   * Adjusts logging buffer and input boost durations.
3. **Automated Commit & Push:** Automatically stages changes, commits them under the GitHub Actions bot profile, and pushes the optimized configuration back to the target kernel branch.
4. **Telegram Alert:** Sends a confirmation alert message to the Telegram group notifying team members that the defconfig has been updated.
