# Distrobuilder 自動ビルド

[English](/doc/README-en.md) | [繁體中文](/doc/README-zh-TW.md) | [日本語](/doc/README-ja.md)

> **注意**: このドキュメントは翻訳版です。齟齬がある場合は、[英語版](README-en.md)を正としてください。

このプロジェクトは、[Distrobuilder](https://github.com/lxc/distrobuilder) を使用して LXC および Incus/VM イメージを作成するための自動ビルドシステムを提供します。GitHub Actions を利用して、設定ファイルの自動検出、イメージのビルド、およびリリースの管理を行います。

## 📁 設定 (`config/`)

`config/` ディレクトリはこのビルドシステムの中核です。**動的かつ拡張可能**に設計されています。

- **自動検出**: 新しい設定ファイルをどこかに登録する必要はありません。新しい `.yaml` または `.yml` ファイルを `config/` フォルダに配置するだけで、ビルドシステムが自動的に認識します。
- **命名規則**: ビルド成果物は設定ファイル名に基づいて命名されます。例えば、`config/my-distro.yaml` は以下を生成します:
  - `dist/my-distro-lxc-rootfs.tar.xz` (LXC)
  - `dist/my-distro-vm.qcow2` (VM)

**注**: 現在、フォルダには `aghome-alpine.yaml` などの例が含まれていますが、異なるディストリビューションやビルドバリアントの設定ファイルをいくつでも保持することを意図しています。

## ⚙️ ワークフロー

プロジェクトには、ビルドライフサイクルを処理するためのいくつかの GitHub Actions ワークフローが含まれています。

### 1. `triggered-prerelease.yml` (プッシュ時の検証)

- **トリガー**: `main` または `master` へのプッシュ時に、`config/*.yaml` または `config/*.yml` の変更が検出されると自動的に実行されます。
- **動作**:
  - **スマートビルド**: コミットで変更された特定の設定ファイル**のみ**をビルドします。
  - **成果物**: ビルドされたイメージ（命名規則に従ってリネーム済み）を GitHub Actions の実行アーティファクトにアップロードします（30 日間保持）。
  - **目的**: すべてのイメージをビルドしてリソースを無駄にすることなく、変更を迅速に検証します。

### 2. `full-release.yml` (手動リリース)

- **トリガー**: GitHub Actions の "Run workflow" ボタンから手動でトリガーします。
- **入力**:
  - `version`: リリースバージョン（例: `v1.0.0`）。
  - `prerelease`: プレリリースとしてマークするためのチェックボックス。
- **動作**:
  - `config/` フォルダにある**すべて**の設定をビルドします。
  - 指定されたバージョンで GitHub Release を作成します。
  - すべてのビルド成果物（rootfs, metadata, disks）をリリースにアップロードします。

### 3. `nightly-prerelease.yml` (スケジュール実行)

- **トリガー**: スケジュール（週次や毎晩など）で実行されます。
- **動作**: すべての設定をビルドし、ローリング `pre-release` タグを更新します。

## 🛠️ カスタマイズ

プロジェクトルートにある `.distrobuilder-version` ファイルを編集することで、ビルドプロセスで使用されるツールの特定のバージョンを制御できます。

```bash
DISTROBUILDER_VERSION=04b679e91ccfc60ba6a061fed94d4801f7a1036f
GO_VERSION=1.25
```

- **DISTROBUILDER_VERSION**: コンパイルする `lxc/distrobuilder` のコミットハッシュ、タグ、またはブランチ名。
- **GO_VERSION**: Distrobuilder のビルドに使用する Go のバージョン。

## 🚀 使用方法

イメージがビルドされリリースされると、環境にインポートできます。

### LXC

```bash
lxc image import <name>-lxc-metadata.tar.xz <name>-lxc-rootfs.tar.xz --alias <alias>
```

### Incus (VM)

```bash
incus image import <name>-vm-metadata.tar.xz <name>-vm.qcow2 --alias <alias>
```
