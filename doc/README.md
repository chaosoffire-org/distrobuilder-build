# Distrobuilder Automation Build

This project provides an automated build system for creating LXC and Incus/VM images using [Distrobuilder](https://github.com/lxc/distrobuilder). It utilizes GitHub Actions to automatically detect configuration files, build images, and manage releases.

## 📁 Configuration (`config/`)

The `config/` directory is the core of this build system. It is designed to be **dynamic and extensible**:

- **Automatic Detection**: You do not need to register new configuration files anywhere. simply drop a new `.yaml` or `.yml` file into the `config/` folder, and the build system will automatically pick it up.
- **Naming Convention**: Build artifacts are named based on the configuration filename. For example, `config/my-distro.yaml` will produce:
  - `dist/my-distro-lxc-rootfs.tar.xz` (LXC)
  - `dist/my-distro-vm.qcow2` (VM)

**Note**: The folder currently contains examples like `aghome-alpine.yaml`, but it is intended to hold any number of configuration files for different distributions or build variants.

## ⚙️ Workflows

The project includes several GitHub Actions workflows to handle the build lifecycle:

### 1. `triggered-prerelease.yml` (On-Push Validation)

- **Trigger**: Runs automatically on pushes to `main` or `master` when changes are detected in `config/*.yaml` or `config/*.yml`.
- **Behavior**:
  - **Smart Build**: It ONLY builds the specific configuration files that were changed in the commit.
  - **Artifacts**: Uploads built images (renamed to naming convention) to GitHub Actions run artifacts (30 days retention).
  - **Purpose**: Rapid validation of changes without wasting resources building the entire suite of images.

### 2. `full-release.yml` (Manual Release)

- **Trigger**: Manually triggered via the "Run workflow" button in GitHub Actions.
- **Inputs**:
  - `version`: The release version (e.g., `v1.0.0`).
  - `prerelease`: Checkbox to mark as a pre-release.
- **Behavior**:
  - Builds **ALL** configurations found in the `config/` folder.
  - Creates a GitHub Release with the specified version.
  - Uploads all built artifacts (rootfs, metadata, disks) to the release.

### 3. `nightly-prerelease.yml` (Scheduled)

- **Trigger**: Runs on a schedule (e.g., weekly or nightly).
- **Behavior**: Builds all configurations and updates a rolling `pre-release` tag.

## 🛠️ Customization

You can control the specific versions of the tools used in the build process by editing the file `.distrobuilder-version` in the project root:

```bash
DISTROBUILDER_VERSION=04b679e91ccfc60ba6a061fed94d4801f7a1036f
GO_VERSION=1.25
```

- **DISTROBUILDER_VERSION**: Commit hash, tag, or branch name of `lxc/distrobuilder` to compile.
- **GO_VERSION**: Version of Go to use for building Distrobuilder.

## 🚀 Usage

Once images are built and released, you can import them into your environment.

### LXC

```bash
lxc image import <name>-lxc-metadata.tar.xz <name>-lxc-rootfs.tar.xz --alias <alias>
```

### Incus (VM)

```bash
incus image import <name>-vm-metadata.tar.xz <name>-vm.qcow2 --alias <alias>
```
