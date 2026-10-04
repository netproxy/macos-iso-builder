# mkmaciso — macOS 安装器 ISO/DMG 制作工具

> **语言 / Language**：[中文](#中文说明) | [English](#english)

---

<a id="中文说明"></a>
## 中文说明

直接从 Apple 官方服务器构建可启动的 macOS 安装器 ISO 和 DMG — 无需 Mac。

本项目包含两部分：
1. 脚本（`mkmaciso`）：仅使用 macOS 内置工具和命令，从 Apple 服务器下载完整的 macOS 安装器到 **/Applications**，然后制作可启动的 ISO/DMG 镜像。
2. GitHub Action 工作流：如果没有 macOS，可在 Azure 数据中心托管的 Mac mini 上运行 `mkmaciso`。

### 语言切换

脚本输出支持中英文双语，检测优先级：`--lang` 参数 > `MKMACISO_LANG` 环境变量 > 自动检测（`$LANG`/`$LC_ALL`、macOS 系统语言）> 默认英文。

```bash
mkmaciso --lang zh              # 本次运行强制中文
MKMACISO_LANG=zh mkmaciso       # 环境变量方式
```

## 开始之前

先看一下 [Release 页面](https://github.com/LongQT-sea/mkmaciso/releases/latest)——也许已经有人构建好了你需要的版本。如果没有，参考[使用方法](#使用方法)自己构建。

> [!Important]
> GitHub 托管的 runner 是免费公共资源——请合理使用。

## 磁盘镜像格式

| | ISO | DMG |
|---|---|---|
| 最适用 | 虚拟机 | 启动 U 盘 |
| 虚拟机支持 | 以虚拟 DVD 方式挂载 | 以虚拟硬盘方式挂载 |
| 布局 | 混合 UDF/HFS | 原始 GPT 磁盘镜像 |

**ISO 文件**——非常适合虚拟机 *（Proxmox、QEMU、VMware）*，像虚拟 DVD 一样挂载即可。在 Windows 下也能直接挂载查看内容。

**DMG 文件**——用 [Rufus](https://rufus.ie/en/#download) *（Windows）*、`dd` *（Linux）* 或 `asr` *（macOS）* 烧录到 U 盘即可制作启动安装盘。用于虚拟机时，先用 `qemu-img` 转换为 `.vhd` *（Hyper-V）* 或 `.vmdk` *（VMware）*。QEMU/Proxmox 可直接使用原始磁盘镜像，无需转换。
> 注意：DMG 文件名自带 `.img` 后缀 *（例如 macOS_Sequoia.dmg.img）*，这样 Rufus 在资源管理器中无需切换"所有文件"就能找到它。

## 支持的版本

从 OS X Lion（10.7，2011 年）到最新的 macOS Tahoe（26，2025 年），基本全覆盖。完整列表：

Lion、Mountain Lion、Mavericks、Yosemite、El Capitan、Sierra、High Sierra、Mojave、Catalina、Big Sur、Monterey、Ventura、Sonoma、Sequoia、Tahoe。

---

## 使用方法

### 没有 macOS？用 GitHub Actions

> [!TIP]
> <details>
> <summary>点击查看图文教程（GIF）</summary>
>
> ![如何 Fork 并运行工作流](https://raw.githubusercontent.com/LongQT-sea/macos-iso-builder/main/.github/how_to_fork_and_run_workflow.gif)
>
> </details>

1. 先 Star 再 [Fork](https://github.com/LongQT-sea/macos-iso-builder/fork) 本仓库（需要 GitHub 账号）。<br>
   <img src="https://raw.githubusercontent.com/LongQT-sea/macos-iso-builder/main/.github/star_and_fork.jpg" width="500">
2. 在 Fork 后的仓库中打开 **Actions** 标签页。
3. 点击绿色的 **"I understand my workflows, go ahead and enable them"** 按钮。
4. 从左侧边栏选择一个工作流：
   * **Recovery ISO** *（推荐）*——精简的恢复镜像，构建只需 2-5 分钟，适合虚拟机。
   * **Full Installer**——完整离线安装器，5-18GB，构建需要 5-60 分钟。
5. 点击 **"Run workflow"** 按钮并配置参数：

   * **macOS version**——选择版本（*Sequoia*、*Sonoma* 等）。
   * **Image format**——虚拟机选 `iso`，启动 U 盘选 `dmg`。
6. 点击绿色的 **"Run workflow"** 按钮开始构建，等待工作流完成。
7. 完成后刷新页面，滚动到 **Artifacts** 区域，点击产物链接开始下载（例如 `macOS_Sequoia_15.7.4.iso`）。
8. **Recovery ISO** 的产物是 zip 压缩包——使用前请先解压。

---

### 有 macOS？本地运行 `mkmaciso`

用 Terminal.app 快速运行（把 `tahoe` 换成你想要的版本）：
```bash
curl -s https://raw.githubusercontent.com/LongQT-sea/mkmaciso/main/mkmaciso | bash -s tahoe
```

或先下载脚本，再带参数运行：
```bash
curl -O https://raw.githubusercontent.com/LongQT-sea/mkmaciso/main/mkmaciso
chmod +x mkmaciso
./mkmaciso --help
```

不带参数运行 `./mkmaciso` 会进入交互式菜单。

---

## 使用技巧

虚拟机用户：把 ISO 像虚拟光驱一样挂载即可。Proxmox 用户如果想要更好性能，可以研究一下 GPU 直通。我还有个仓库（[OpenCore-ISO](https://github.com/LongQT-sea/OpenCore-ISO)）可能对安装有帮助，Intel 核显直通看[这里](https://github.com/LongQT-sea/intel-igpu-passthru)。

启动 U 盘：用 DMG 烧录后，U 盘上会有剩余空间，可以拿来建一个 FAT32 分区放 EFI 文件夹。

在 Linux 上用 `dd` 时，务必反复确认目标设备，`dd` 不会二次确认。

## mkmaciso 系统要求

- macOS 10.9 或更高版本（11+ 更佳）
- 推荐 Intel Mac，Apple 芯片可用但有一些限制
- 构建时需要 20-40GB 可用空间
- 互联网连接
- sudo 权限

## 致谢

感谢 Apple 提供的 macOS 和更新服务器，[Mavericks Forever](https://mavericksforever.com/) 记录的 Mavericks 恢复协议，以及 [InsanelyMac 社区](https://www.insanelymac.com/forum/topic/338810-create-legit-copy-of-macos-from-apple-catalog/) 对 Apple 目录直链下载的研究。

## 法律声明

本工具直接从 Apple 官方服务器下载 macOS 镜像，用户有责任遵守 [Apple 软件许可协议](https://www.apple.com/legal/sla/)。macOS 是 Apple Inc. 的商标。

本项目基于 GPLv3 开源。

---

<a id="english"></a>
## English

Build bootable macOS installer ISOs and DMGs directly from Apple's servers — no Mac required.

This project has two parts:
1. A script (`mkmaciso`) that uses only macOS built-in tools and commands to download and install the full macOS installer from Apple's servers into **/Applications**, and then creates bootable ISO/DMG images.
2. GitHub Action workflows that run `mkmaciso` on Azure datacenter-hosted Mac minis if you don't have macOS.

### Language

Script output is bilingual (Chinese/English). Detection priority: `--lang` flag > `MKMACISO_LANG` env var > auto-detect (`$LANG`/`$LC_ALL`, macOS system language) > English default.

```bash
mkmaciso --lang zh              # force Chinese for this run
MKMACISO_LANG=zh mkmaciso       # via environment variable
```

## Before you start

Check [Release page](https://github.com/LongQT-sea/mkmaciso/releases/latest) first - someone might've already built what you need. If not, see [How to use](#how-to-use) to build your own.

> [!Important]
> GitHub-hosted runners are a free public resource — please use them responsibly.

## Disk Image Formats

| | ISO | DMG |
|---|---|---|
| Best for | Virtual Machines | Bootable USB |
| VM Support | Attach as virtual DVD | Attach as virtual hard disk |
| Layout | Hybrid UDF/HFS | Raw GPT disk image |

**ISO files** - These work great for VMs *(Proxmox, QEMU, VMware)*. Just attach them like a virtual DVD. They'll even mount in Windows if you need to poke around inside.

**DMG files** - Flash these to a USB drive with [Rufus](https://rufus.ie/en/#download) *(Windows)*, `dd` *(Linux)*, or `asr` *(macOS)* to make bootable installation media. For VM use, convert them to `.vhd` *(for Hyper-V)* or `.vmdk` *(for VMware)* with `qemu-img`. QEMU/Proxmox can use the raw disk image without conversion.
> Note: DMG files ship with a `.img` suffix *(e.g. macOS_Sequoia.dmg.img)* so Rufus can find them without switching to "All files" in Explorer.

## Supported versions

Pretty much everything from OS X Lion (10.7, 2011) through the latest macOS Tahoe (26, 2025). Full list:

Lion, Mountain Lion, Mavericks, Yosemite, El Capitan, Sierra, High Sierra, Mojave, Catalina, Big Sur, Monterey, Ventura, Sonoma, Sequoia, Tahoe.

---

## How to use

### Don't have macOS? Use GitHub Actions

> [!TIP]
> <details>
> <summary>Click here to watch a visual guide (GIF)</summary>
>
> ![How to fork and run workflow](https://raw.githubusercontent.com/LongQT-sea/macos-iso-builder/main/.github/how_to_fork_and_run_workflow.gif)
>
> </details>

1. Star and then [Fork](https://github.com/LongQT-sea/macos-iso-builder/fork) this repository (requires a GitHub account).<br>
   <img src="https://raw.githubusercontent.com/LongQT-sea/macos-iso-builder/main/.github/star_and_fork.jpg" width="500">
2. Navigate to the **Actions** tab in your forked repository.
3. Click the green **"I understand my workflows, go ahead and enable them"** button.
4. Select a workflow from the left sidebar:
   * **Recovery ISO** *(recommended)* - Small recovery image, builds in 2-5 minutes. Good for VMs.
   * **Full Installer** - Complete offline installer, 5-18GB, takes 5-60 minutes to build.
5. Click the **"Run workflow"** button and configure the workflow inputs:

   * **macOS version** – Choose a version (*Sequoia*, *Sonoma*, etc.).
   * **Image format** – Choose `iso` for virtual machines or `dmg` for bootable USB drives.
6. Click the green **"Run workflow"** button to start the build, then wait for the workflow to complete.
7. Once completed, reload the page and scroll down to the **Artifacts** section. Click the artifact link to start downloading (e.g., `macOS_Sequoia_15.7.4.iso`).
8. **Recovery ISO** artifacts are zipped — unzip before use.

---

### Already have macOS? Run `mkmaciso` locally

Quick run using Terminal.app (change `tahoe` to whatever you want):
```bash
curl -s https://raw.githubusercontent.com/LongQT-sea/mkmaciso/main/mkmaciso | bash -s tahoe
```

Or download the script first, then run with parameters:
```bash
curl -O https://raw.githubusercontent.com/LongQT-sea/mkmaciso/main/mkmaciso
chmod +x mkmaciso
./mkmaciso --help
```

Running `./mkmaciso` without arguments gives you an interactive menu.

---

## Tips

For VMs, just attach the ISO as a virtual CD drive. Proxmox users — if you want better performance, look into GPU passthrough. I have another repo ([OpenCore-ISO](https://github.com/LongQT-sea/OpenCore-ISO)) that might help with installation, and one for [Intel iGPU passthrough](https://github.com/LongQT-sea/intel-igpu-passthru) specifically.

For bootable USB drives, after you flash the DMG there will be leftover space on the drive. You can use that to create a FAT32 partition for your EFI folder if you need one.

If you're using `dd` on Linux, triple-check your target device. `dd` doesn't ask for confirmation.

## Requirements for mkmaciso

- macOS 10.9 or newer (11+ is better)
- Intel Mac recommended, Apple Silicon works but with some limitations
- 20-40GB of free space while building
- Internet connection
- sudo access

## Credits

Apple for macOS and their update servers, [Mavericks Forever](https://mavericksforever.com/) for documenting the Mavericks recovery protocol, and the [InsanelyMac community](https://www.insanelymac.com/forum/topic/338810-create-legit-copy-of-macos-from-apple-catalog/) for their research on downloading macOS directly from Apple's catalog.

## Legal stuff

This tool downloads macOS images directly from Apple's official servers. Users are responsible for complying with [Apple's Software License Agreement](https://www.apple.com/legal/sla/). macOS is a trademark of Apple Inc.

Licensed under GPLv3.
