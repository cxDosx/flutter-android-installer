# Flutter + Android Build Environment Installer

[![Shell](https://img.shields.io/badge/shell-bash-4eaa25)](https://www.gnu.org/software/bash/)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS-blue)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Set up an environment that can build Flutter projects into APK / AAB with a single command — on Linux remote servers and on macOS local machines.

Great for cloud CI servers, remote development environments (GCP / AWS / Alibaba Cloud ECS), and bootstrapping a macOS machine from scratch.

---

## Quick Start

### Linux (Debian / Ubuntu / CentOS / RHEL / Rocky / AlmaLinux / Fedora)

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/cxDosx/flutter-android-installer/main/install-flutter-android.sh)
```

### macOS (Intel / Apple Silicon)

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/cxDosx/flutter-android-installer/main/install-macos.sh)
```

> Prefer the `bash <(...)` form over `curl ... | bash`. The former supports interactive input; the latter cannot read the keyboard in some shells.

The script will prompt you for:

```
Enter the Android SDK version to install [33]:
Enter the Android NDK version to install [28.2.13676358]:
```

Press Enter to accept the defaults. The whole process takes about 10-15 minutes (depending on your network); once done you can run `flutter build apk`.

---

## What It Does

| Component | Version | Linux source | macOS source |
|---|---|---|---|
| JDK | 17 | apt / dnf | Homebrew |
| Android command-line tools | latest | Google official (linux package) | Google official (mac package) |
| Android SDK Platform | user-specified (default 33) | sdkmanager | sdkmanager |
| Android Build Tools | matches the SDK version | sdkmanager | sdkmanager |
| Android NDK | user-specified (default 28.2.13676358) | sdkmanager | sdkmanager |
| Flutter | stable channel | git | git |

It also configures the `PATH`, `JAVA_HOME`, `ANDROID_HOME` and related environment variables automatically.

---

## Supported Systems

### Linux (`install-flutter-android.sh`)

Detected automatically via `/etc/os-release`. The following distributions are verified:

- ✅ Debian 11 / 12
- ✅ Ubuntu 20.04 / 22.04 / 24.04
- ✅ CentOS Stream 9
- ✅ RHEL 8 / 9
- ✅ Rocky Linux 8 / 9
- ✅ AlmaLinux 8 / 9
- ✅ Fedora

The package manager is selected automatically (`apt` / `dnf` / `yum`).

### macOS (`install-macos.sh`)

- ✅ macOS 12+ (Monterey and later)
- ✅ Apple Silicon (M1 / M2 / M3 / M4)
- ✅ Intel Mac

Requires [Homebrew](https://brew.sh/); the script installs it automatically if it is missing.

---

## Which Script to Use

| Your environment | Use this |
|---|---|
| Remote Linux server (GCP / AWS / Alibaba Cloud, etc.) | `install-flutter-android.sh` |
| Local macOS | `install-macos.sh` |
| Windows | **Not supported** — use WSL2 + `install-flutter-android.sh` |

---

## Usage

### Option 1: Run online (recommended)

```bash
# Linux
bash <(curl -fsSL https://raw.githubusercontent.com/cxDosx/flutter-android-installer/main/install-flutter-android.sh)

# macOS
bash <(curl -fsSL https://raw.githubusercontent.com/cxDosx/flutter-android-installer/main/install-macos.sh)
```

### Option 2: Download, then run

```bash
# Linux
curl -fsSL https://raw.githubusercontent.com/cxDosx/flutter-android-installer/main/install-flutter-android.sh -o install-flutter-android.sh
chmod +x install-flutter-android.sh
./install-flutter-android.sh

# macOS
curl -fsSL https://raw.githubusercontent.com/cxDosx/flutter-android-installer/main/install-macos.sh -o install-macos.sh
chmod +x install-macos.sh
./install-macos.sh
```

### Option 3: Clone the repository

```bash
git clone https://github.com/cxDosx/flutter-android-installer.git
cd flutter-android-installer

# Pick the script for your system
bash install-flutter-android.sh   # Linux
bash install-macos.sh             # macOS
```

---

## Input Parameters

Both scripts take identical input:

| Parameter | Default | Validation rule |
|---|---|---|
| Android SDK version | `33` | Integer, between 21 and 99 |
| Android NDK version | `28.2.13676358` | Must be three numeric segments `X.Y.Z` |

**The script exits immediately on invalid input** and will not install a wrong version. For example:

```
$ bash install-flutter-android.sh
Enter the Android SDK version to install [33]: abc
[ERROR] SDK version must be an integer, but got: 'abc'
```

---

## Install Locations

| Item | Path (identical for both scripts) |
|---|---|
| Android SDK | `~/android-sdk` |
| Flutter | `~/flutter` |
| Environment variables (Linux bash) | `~/.bashrc` |
| Environment variables (Linux zsh) | `~/.zshrc` |
| Environment variables (macOS, default zsh) | `~/.zshrc` |
| Environment variables (macOS legacy bash) | `~/.bash_profile` |

After installation, open a new terminal or `source` the relevant file to apply the environment variables.

---

## Verify the Installation

```bash
# Linux
source ~/.bashrc

# macOS
source ~/.zshrc

flutter doctor
```

Look at these two lines:

```
[✓] Flutter
[✓] Android toolchain
```

As long as those two show ✓, you can build APK / AAB normally. The other items (Chrome, Linux desktop, Android Studio) are irrelevant to command-line builds and can be ignored.

---

## Build Your Project

```bash
cd ~/your-flutter-project
flutter pub get

# Debug APK (fast, unsigned)
flutter build apk --debug

# Release APK (split per ABI)
flutter build apk --release --split-per-abi

# AAB (the format required for Google Play)
flutter build appbundle --release
```

Output locations:

- `build/app/outputs/flutter-apk/*.apk`
- `build/app/outputs/bundle/release/*.aab`

---

## Idempotency

The scripts are **idempotent** — running them again will not reinstall things:

- Existing cmdline-tools / Flutter directories are skipped
- An NDK that is already fully installed is skipped
- An incomplete leftover NDK directory (missing `source.properties`) is cleaned up and reinstalled
- The environment-variable block is wrapped in markers, so re-runs replace it instead of appending duplicates

---

## FAQ

### Q: Interactive input does not work when run via `curl ... | bash`

That is expected. Use the `bash <(curl ...)` form instead:

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/cxDosx/flutter-android-installer/main/install-flutter-android.sh)
```

Or download the script locally and run it.

### Q: Installation fails with "NDK installation failed"

This is usually caused by a network interruption that left the download incomplete, or by a version number that does not exist in the Google repository.

The NDK version number must be **exact** (including the long trailing digits). Check available versions in the [Android NDK Revision History](https://developer.android.com/ndk/downloads/revision_history).

### Q: Installation fails with "could not find `build-tools;XX.0.0`"

By default the script installs Build Tools version `<SDK version>.0.0`. Almost every modern SDK version has a matching `.0.0` Build Tools, but a few do not. If you hit this, look up an existing version in the [Android SDK Build Tools list](https://developer.android.com/tools/releases/build-tools) and re-run the script with the corresponding SDK version.

### Q: `Flutter requires Android SDK XX and Build Tools XX.X.X`

A Flutter SDK upgrade may require a newer Android SDK. Just re-run the script and enter the new SDK version number. Multiple SDK versions can coexist.

### Q: Homebrew installs slowly on macOS

You can configure a faster regional mirror before running the script. See [Homebrew mirror help — TUNA mirror](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/).

### Q: macOS says `java_home cannot find 17`

This usually happens when Homebrew installs the JDK but does not link it into the system directory. The script handles this step automatically (`ln -sfn` into `/Library/Java/JavaVirtualMachines`), which requires your `sudo` password.

### Q: Can it run non-interactively (all defaults)?

In an environment with no TTY at all (such as CI), the script automatically skips the prompts and uses the defaults. If you want to force non-interactive mode in an environment that does have a terminal, feel free to open an issue requesting a `--non-interactive` flag.

---

## Uninstall

### Linux

```bash
rm -rf ~/android-sdk ~/flutter
# Then manually edit ~/.bashrc / ~/.zshrc and remove the block wrapped
# by the flutter-android-env markers
```

### macOS

```bash
rm -rf ~/android-sdk ~/flutter

# Uninstall the JDK (optional)
brew uninstall openjdk@17
sudo rm -f /Library/Java/JavaVirtualMachines/openjdk-17.jdk

# Then manually edit ~/.zshrc and remove the block wrapped
# by the flutter-android-env markers
```

---

## License

[MIT](LICENSE)
