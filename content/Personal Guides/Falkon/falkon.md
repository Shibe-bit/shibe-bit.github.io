---
title: "Falkon guide to build"
date: 2026-02-5
tags: [Git, KDE, Mentorship, OpenSource]
draft: false
---

# 1. Environment Preparation

First, install all necessary libraries for the Qt6/KF6 stack. This ensures the browser engine and UI components are available.

```Bash
sudo pacman -S cmake ninja base-devel extra-cmake-modules \
qt6-base qt6-webengine qt6-webchannel qt6-svg \
karchive ki18n kconfig kcoreaddons kwidgetsaddons kcompletion kio kwallet \
purpose breeze-icons
```

### 2. The Clean Build Process

```bash
# Move to your source directory
cd ~/falkon

# Clean up previous failed attempts
mkdir build && cd build

# Configure with the local prefix and Quick Install toggle
cmake -DCMAKE_INSTALL_PREFIX=./install -DQUICK_INSTALL=ON ..

# Compile using all available CPU cores
make -j$(nproc)

# Install to the local ./install folder
make install
```
### 3. The "Visual Fix" Launcher

Create a script or run these lines to launch:

```bash
# 1. Define where the build lives
export FALKON_PREFIX=$HOME/falkon/build/install

# 2. Tell Falkon where to find icons/themes (Fixes the Hamburger Menu)
export XDG_DATA_DIRS="$FALKON_PREFIX/share:$XDG_DATA_DIRS"

# 3. Tell Falkon where the plugins/extensions are
export QT_PLUGIN_PATH="$FALKON_PREFIX/lib/plugins:$QT_PLUGIN_PATH"

# 4. Launch the binary
"$FALKON_PREFIX/bin/falkon"
```