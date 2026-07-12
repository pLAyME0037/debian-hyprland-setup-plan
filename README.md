# Debian Hyprland Setup Plan
## document date 12 Jul 2026 - redo, not confirm working
---
1. Install Dependencies
Added libsdbus-c++-dev (for xdg-portal), libdisplay-info-dev (for aquamarine),
and fixed missing build tools.

```sh
sudo apt update && sudo apt install -y \
  build-essential git meson ninja-build pkg-config cmake bison flex \
  libwayland wayland-protocols \
  libdrm libgbm libxkbcommon libpixman-1 \
  libegl libgles2 libseat seatd libinput \
  libxcb-composite0 libxcb-render0 libxcb-shape0 \
  libxcb-xfixes0 libxcb-dri3 libxcb-present \
  libxcb-sync libxcb-randr0 libxcb-cursor libxcb-icccm4 \
  libpango1.0 libudev hwdata glslang-tools spirv-tools \
  libtomlplusplus libcairo2 \
  libgl1-mesa libgles2-mesa mesa-common \
  libdisplay-info libsdbus-c++ libliftoff \
  libxcb-xinerama0 libxcb-xinput \
  libpugixml libmagic libxcb-xkb \
  libxcursor libre2 libmuparser uuid \
  libcairo2 libpango1.0 libpangocairo-1.0-0 \
  libinput libglib2.0 libpipewire-0.3 libspa-0.2 \
  pipewire-jack pipewire-audio-client-libraries \
  libiniparser pkg-config
```

- for dev pkgs
```sh
sudo apt update && sudo apt install -y \
  build-essential git meson ninja-build pkg-config cmake bison flex \
  libwayland-dev wayland-protocols \
  libdrm-dev libgbm-dev libxkbcommon-dev libpixman-1-dev \
  libegl-dev libgles2 libseat-dev seatd libinput-dev \
  libxcb-composite0-dev libxcb-render0-dev libxcb-shape0-dev \
  libxcb-xfixes0-dev libxcb-dri3-dev libxcb-present-dev \
  libxcb-sync-dev libxcb-randr0-dev libxcb-cursor-dev libxcb-icccm4-dev \
  libpango1.0-dev libudev-dev hwdata glslang-tools spirv-tools \
  libtomlplusplus-dev libcairo2-dev \
  libgl1-mesa-dev libgles2-mesa-dev mesa-common-dev \
  libdisplay-info-dev libsdbus-c++-dev libliftoff-dev \
  libxcb-xinerama0-dev libxcb-xinput-dev \
  libpugixml-dev libmagic-dev libxcb-xkb-dev \
  libxcursor-dev libre2-dev libmuparser-dev uuid-dev \
  libcairo2-dev libpango1.0-dev libpangocairo-1.0-0 \
  libinput-dev libglib2.0-dev libpipewire-0.3-dev libspa-0.2-dev \
  pipewire-jack pipewire-audio-client-libraries \
  libiniparser-dev pkg-config
```

## You may need to reboot or re-login for the group change to apply.
3. Build Core Hyprland Components (Strict Order)
Hyprland has split its code into several sub-libraries. You must build them in
this specific order.
### 1. Hyprwayland-scanner (REQUIRED FIRST)
```sh
git clone https://github.com/hyprwm/hyprwayland-scanner.git
cd hyprwayland-scanner
cmake --no-warn-unused-cli -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . -B ./build
cmake --build ./build --config Release --target all -j`nproc 2>/dev/null || getconf NPROCESSORS_CONF`
sudo cmake --install build
cd ..
```

### 2. Hyprutils
```sh
git clone https://github.com/hyprwm/hyprutils.git
cd hyprutils/
cmake --no-warn-unused-cli -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . -B ./build
cmake --build ./build --config Release --target all -j`nproc 2>/dev/null || getconf NPROCESSORS_CONF`
sudo cmake --install build
cd ..
```

### 3. Hyprlang
```sh
git clone https://github.com/hyprwm/hyprlang
cd hyprlang
cmake --no-warn-unused-cli -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . -B ./build
cmake --build ./build --config Release --target hyprlang -j`nproc 2>/dev/null || getconf _NPROCESSORS_CONF`
sudo cmake --install build
cd ..
```

### 4. Hyprcursor
```sh
git clone https://github.com/hyprwm/hyprcursor
cd hyprcursor
cmake --no-warn-unused-cli -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . -B ./build
cmake --build ./build --config Release --target all -j`nproc 2>/dev/null || getconf _NPROCESSORS_CONF`
sudo cmake --install build
cd ..
```

### 5. Hyprgraphics (NEW REQUIREMENT)

```sh
git clone https://github.com/hyprwm/hyprgraphics
cd hyprgraphics/
cmake --no-warn-unused-cli -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . -B ./build
cmake --build ./build --config Release --target all -j`nproc 2>/dev/null || getconf NPROCESSORS_CONF`
sudo cmake --install build
cd ..
```

### 6. Aquamarine (NEW BACKEND - REQUIRED)
```sh
git clone https://github.com/hyprwm/aquamarine.git
cd aquamarine
cmake --no-warn-unused-cli -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . -B ./build
cmake --build ./build --config Release --target all -j`nproc 2>/dev/null || getconf _NPROCESSORS_CONF`
sudo cmake --install build
cd ..
```

### 13. Build Hyprland
```sh
git clone --recursive https://github.com/hyprwm/Hyprland
cd Hyprland
make all
sudo make install
cd ..
```

---

4. xdg-desktop-portal-hyprland
Do not skip this, or screensharing and OBS will not work.
```
sudo apt install -y xdg-desktop-portal-hyprland
```

## Final Config
### Standard tools
```sh
sudo apt install -y kitty rofi libxml2 waybar wl-clipboard grim slurp swaybg polkit-kde-agent-1
```

### Create config
```sh
mkdir -p ~/.config/hypr
cp /usr/local/share/hyprland/examples/hyprland.conf ~/.config/hypr/
```

5. Other if needed
- hyprlauncher
```sh
sudo apt install libqalculate-dev pkg-config
git clone https://github.com/hyprwm/hyprlauncher.git
cd hyprlauncher
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
sudo make install
```

- sddm
```sh
sudo apt install sddm
sudo dpkg-reconfigure sddm  # Select SDDM as default
sudo systemctl enable sddm
reboot
```
