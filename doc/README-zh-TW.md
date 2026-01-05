# Distrobuilder 自動化構建

[English](/doc/README-en.md) | [繁體中文](/doc/README-zh-TW.md) | [日本語](/doc/README-ja.md)

> **注意**：本文件為翻譯版本。若有任何歧義，請以[英文版本](README-en.md)為準。

本專案提供了一個自動化構建系統，使用 [Distrobuilder](https://github.com/lxc/distrobuilder) 建立 LXC 和 Incus/VM 映像。它利用 GitHub Actions 自動偵測設定檔、構建映像並管理發布。

## 📁 設定檔 (`config/`)

`config/` 目錄是此構建系統的核心。它的設計宗旨是 **動態且可擴充**：

- **自動偵測**：您無需在任何地方註冊新的設定檔。只需將新的 `.yaml` 或 `.yml` 檔案放入 `config/` 資料夾，構建系統就會自動選取它。
- **命名慣例**：構建產物會根據設定檔名稱命名。例如，`config/my-distro.yaml` 會產生：
  - `dist/my-distro-lxc-rootfs.tar.xz` (LXC)
  - `dist/my-distro-vm.qcow2` (VM)

**注意**：該資料夾目前包含 `aghome-alpine.yaml` 等範例，但它的目的是容納任意數量的不同發行版或構建變體的設定檔。

## ⚙️ 工作流程 (Workflows)

本專案包含數個 GitHub Actions 工作流程來處理構建生命週期：

### 1. `triggered-prerelease.yml` (推送時驗證)

- **觸發條件**：當 `main` 或 `master` 分支上的推送被偵測到 `config/*.yaml` 或 `config/*.yml` 有變更時自動執行。
- **行為**：
  - **智慧構建**：它**只會**構建在提交中被變更的特定設定檔。
  - **產物**：將構建好的映像（依命名慣例重新命名）上傳至 GitHub Actions 執行產物（保留 30 天）。
  - **目的**：快速驗證變更，而無需浪費資源構建整套映像。

### 2. `full-release.yml` (手動發布)

- **觸發條件**：透過 GitHub Actions 中的 "Run workflow" 按鈕手動觸發。
- **輸入**：
  - `version`：發布版本（例如 `v1.0.0`）。
  - `prerelease`：核取方塊以標記為預發布。
- **行為**：
  - 構建 `config/` 資料夾中找到的 **所有** 設定檔。
  - 使用指定的版本建立 GitHub Release。
  - 將所有構建產物（rootfs, metadata, disks）上傳至該發布版本。

### 3. `nightly-prerelease.yml` (排程執行)

- **觸發條件**：依排程執行（例如每週或每晚）。
- **行為**：構建所有設定檔並更新滾動的 `pre-release` 標籤。

## 🛠️ 自訂

您可以透過編輯專案根目錄中的 `.distrobuilder-version` 檔案來控制構建過程中使用的工具版本：

```bash
DISTROBUILDER_VERSION=04b679e91ccfc60ba6a061fed94d4801f7a1036f
GO_VERSION=1.25
```

- **DISTROBUILDER_VERSION**：要編譯的 `lxc/distrobuilder` 的提交雜湊、標籤或分支名稱。
- **GO_VERSION**：用於構建 Distrobuilder 的 Go 版本。

## 🚀 使用方法

一旦映像構建並發布完成，您可以將它們匯入您的環境中。

### LXC

```bash
lxc image import <name>-lxc-metadata.tar.xz <name>-lxc-rootfs.tar.xz --alias <alias>
```

### Incus (VM)

```bash
incus image import <name>-vm-metadata.tar.xz <name>-vm.qcow2 --alias <alias>
```
