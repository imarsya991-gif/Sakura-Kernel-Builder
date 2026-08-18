# 🔨 Kernel Builder CI

A fully automated **GitHub Actions** workflow to build Android kernels straight from your source repo — no local toolchain, no VPS, no headaches. Just fill in the form, hit run, and get a flashable zip delivered to a GitHub Release (and optionally straight to your Telegram).

<p align="center">
  <img src="https://img.shields.io/badge/platform-Android-3DDC84?style=for-the-badge&logo=android&logoColor=white" alt="Platform">
  <img src="https://img.shields.io/badge/CI-GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white" alt="GitHub Actions">
  <img src="https://img.shields.io/badge/runner-ubuntu--22.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu 22.04">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/kernel-4.4%20%7C%204.14%20%7C%204.19-blue?style=flat-square" alt="Kernel versions">
  <img src="https://img.shields.io/badge/toolchain-Clang%20%2B%20GCC-orange?style=flat-square" alt="Toolchain">
  <img src="https://img.shields.io/badge/KernelSU-Next%20%7C%20Original-purple?style=flat-square" alt="KernelSU support">
  <img src="https://img.shields.io/badge/packaging-AnyKernel3-brightgreen?style=flat-square" alt="AnyKernel3">
  <img src="https://img.shields.io/badge/notifications-Telegram-26A5E4?style=flat-square&logo=telegram&logoColor=white" alt="Telegram notifications">
  <img src="https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square" alt="License">
</p>

---

## ✨ Features

- 🚀 **One-click builds** via `workflow_dispatch` — no CLI, no local machine needed
- 🧠 **Auto toolchain selection** — picks the right Clang/GCC pair based on your kernel version (4.4, 4.14, 4.19)
- 🩺 **Self-healing KernelSU wiring check** — detects broken/missing KernelSU driver references in `drivers/Kconfig` before the build even starts
- 🔓 **Optional KernelSU patching** — supports both `KernelSU-Next` and the original `KernelSU`, with the matching Manager APK auto-downloaded and bundled
- 🎁 **AnyKernel3 packaging** (optional) — produces a ready-to-flash zip, or a raw `Image.gz-dtb` if you prefer
- 📦 **Automatic GitHub Release** — build output is published with commit hash, SHA256, and duration info
- 📣 **Telegram integration** — get build **start / success / failure** notifications, complete with the finished file sent directly to your chat
- 🧹 **Disk space cleanup + ccache** — keeps the runner from running out of space and speeds up repeat builds

---

## 📋 Requirements

Before running the workflow, make sure you have:

1. A **GitHub repository** (fork or your own) with this workflow file placed at:
   ```
   .github/workflows/build-kernel.yml
   ```
2. `Actions` enabled, and **write permissions** for `GITHUB_TOKEN` (the workflow already requests `contents: write` for creating releases).
3. A kernel source tree that is buildable with `make` + `defconfig` (standard AOSP-style kernel layout).
4. *(Optional)* A **Telegram Bot Token** and **Chat ID** if you want build notifications.

---

## ⚙️ Setup

### 1. Add the workflow to your repo

Copy `build-kernel.yml` into `.github/workflows/` in the repository that will host your build:

```
your-repo/
└── .github/
    └── workflows/
        └── build-kernel.yml
```

### 2. (Optional) Set up a Telegram bot

1. Create a bot via [@BotFather](https://t.me/BotFather) and grab the **Bot Token**.
2. Get your **Chat ID** (personal chat, group, or channel — use something like [@userinfobot](https://t.me/userinfobot) or `getUpdates`).
3. You can either paste these directly into the workflow inputs each run, or store them as **GitHub Secrets** (`BOT_TOKEN`, `CHAT_ID`) and reference them if you modify the workflow to pull from secrets instead of manual input.

### 3. Run the build

Go to your repo → **Actions** tab → **Build Kernel** → **Run workflow**, then fill in the inputs below.

---

## 🧾 Workflow Inputs

| Input | Required | Description | Example |
|---|:---:|---|---|
| `KERNEL_SOURCE` | ✅ | Git URL of the kernel source | `https://github.com/user/android_kernel_xiaomi_lavender` |
| `KERNEL_BRANCH` | ✅ | Branch to clone | `lineage-20` |
| `KERNEL_CONFIG` | ✅ | Defconfig name (no path) | `lavender_defconfig` |
| `KERNEL_NAME` | ❌ | Name used in output filenames | `Sakura` (default) |
| `KERNEL_VERSION` | ✅ | Kernel major version — determines toolchain | `4.4` / `4.14` / `4.19` |
| `CODENAME` | ✅ | Device codename | `lavender` |
| `BOT_TOKEN` | ❌ | Telegram Bot Token for notifications | — |
| `CHAT_ID` | ❌ | Telegram Chat/Group/Channel ID | — |
| `USE_ANYKERNEL3` | ✅ | Package output as an AnyKernel3 flashable zip | `yes` / `no` |
| `PATCH_KERNELSU` | ❌ | Auto-patch KernelSU into the source | `none` / `ksu-next` / `ksu-original` |

> 💡 **Tip:** `KERNEL_CONFIG` is just the filename (e.g. `lavender_defconfig`), not the full path — the workflow runs `make <config>` from the kernel root.

---

## 🔄 How It Works

```
┌─────────────────────┐
│ 1. Notify: Started  │  (Telegram, if configured)
└──────────┬───────────┘
           ▼
┌─────────────────────┐
│ 2. Free disk space   │
│    + install deps    │
└──────────┬───────────┘
           ▼
┌─────────────────────┐
│ 3. Clone kernel src  │  (shallow + submodules)
└──────────┬───────────┘
           ▼
┌─────────────────────┐
│ 4. Setup toolchain   │  Clang + GCC, version-matched
└──────────┬───────────┘
           ▼
┌─────────────────────┐
│ 5. Patch KernelSU    │  (optional)
│    + sanity check    │
└──────────┬───────────┘
           ▼
┌─────────────────────┐
│ 6. Build kernel       │  make defconfig → build → Image.gz-dtb
└──────────┬───────────┘
           ▼
┌─────────────────────┐
│ 7. Package            │  AnyKernel3 zip and/or raw image
│    + fetch KSU Manager│
└──────────┬───────────┘
           ▼
┌─────────────────────┐
│ 8. Publish to Release │  tagged build-<run_id>
└──────────┬───────────┘
           ▼
┌─────────────────────┐
│ 9. Notify: Success/  │  (Telegram, file sent directly)
│    Failure            │
└─────────────────────┘
```

**Toolchain matrix** (auto-selected by `KERNEL_VERSION`):

| Kernel Version | Clang Branch | GCC (aarch64 + arm) |
|---|---|---|
| `4.4` | `9.0.0-release` | AOSP GCC 4.9 (both) |
| `4.14` | `12.0.0-release` | AOSP GCC 4.9 (both) |
| `4.19` | `14.0.0-release` | Not used (full Clang) |

---

## 📦 Output

On a successful build, you'll get a GitHub Release tagged `build-<run_id>` containing:

- `<KernelName>-<codename>-<date>[-KSU/-KSU-NEXT].zip` — AnyKernel3 flashable zip (if `USE_ANYKERNEL3=yes`)
- `<KernelName>-<codename>-<date>[-KSU/-KSU-NEXT]-Image.gz-dtb` — raw kernel image
- KernelSU Manager APK (if `PATCH_KERNELSU` is set)

If Telegram is configured, the same file lands directly in your chat with build duration, file size, SHA256, and the commit hash.

---

## 🛠️ Troubleshooting

| Problem | Likely Cause |
|---|---|
| `KernelSU reference found... driver folder is empty/missing` | Your source already references KernelSU in `drivers/Kconfig` but the submodule failed to init. Check your kernel source's submodules, or set `PATCH_KERNELSU` correctly. |
| `Kernel image not found, build failed` | `make` didn't produce an `Image.gz-dtb`/`Image` — check the uploaded `build-log` artifact for the actual compiler error. |
| `ld: cannot represent machine 'arm'` | Already patched automatically by disabling `CONFIG_COMPAT_VDSO` — if you still hit this, your defconfig may need manual adjustment. |
| No Telegram messages | Double-check `BOT_TOKEN`/`CHAT_ID` and that the bot has permission to post in that chat. |

On failure, the last lines of `build.log` are sent to Telegram (if configured) and the full log is uploaded as a workflow artifact named `build-log`.

---

## 🙏 Credits

- **[AnyKernel3](https://github.com/osm0sis/AnyKernel3)** by osm0sis — flashable zip packaging
- **[KernelSU](https://github.com/tiann/KernelSU)** by tiann — root solution for Android kernels
- **[KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next)** — actively maintained KernelSU fork
- **[ZyCromerZ Clang](https://github.com/ZyCromerZ/Clang)** / **[proton-clang](https://github.com/kdrag0n/proton-clang)** — prebuilt Clang toolchains
- **AOSP GCC prebuilts** — Google's `prebuilts/gcc` toolchains for legacy 32/64-bit assembler support
- Workflow maintained by **Ryu** ([oneofrisuofc](https://t.me/oneofrisuofc)) under **[NMX Project](https://t.me/nmxproject)**

---

## 📄 License

This workflow is released under the MIT License. Kernel source code and third-party components retain their own respective licenses (GPLv2 for Linux kernel sources, and whatever license each bundled tool/toolchain ships with).

---

<p align="center">Made with 🔧 by <b>NMX Project</b></p>
