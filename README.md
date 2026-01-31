# Debian Hyprland Setup Plan 
## document date 29 Jun 2026
---
1. Install Dependencies
Added libsdbus-c++-dev (for xdg-portal), libdisplay-info-dev (for aquamarine), and fixed missing build tools.
```
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
  libpugixml-dev \
  libmagic-dev \
  libxcb-xkb-dev \
  libxcursor-dev \
  libre2-dev \
  libmuparser-dev uuid-dev \
  libcairo2-dev \
  libpango1.0-dev \
  libpangocairo-1.0-0 \
  libinput-dev \
  libglib2.0-dev \
```
2. Setup Seatd (Optional but Recommended)
Hyprland usually works with systemd-logind (standard on Debian) without this, but if you want to force seatd:
```
sudo systemctl enable seatd --now
sudo usermod -aG seat $USER
```
## You may need to reboot or re-login for the group change to apply.
3. Build Core Hyprland Components (Strict Order)
Hyprland has split its code into several sub-libraries. You must build them in this specific order.
### 1. Hyprwayland-scanner (REQUIRED FIRST)
```
git clone https://github.com/hyprwm/hyprwayland-scanner
cd hyprwayland-scanner
cmake -B build
cmake --build build -j `nproc`
sudo cmake --install build
cd ..
```
### 2. Hyprutils
```
git clone https://github.com/hyprwm/hyprutils
cd hyprutils
cmake -B build
cmake --build build -j `nproc`
sudo cmake --install build
cd ..
```
### 3. Hyprlang
```
git clone https://github.com/hyprwm/hyprlang
cd hyprlang
cmake -B build
cmake --build build -j `nproc`
sudo cmake --install build
cd ..
```
### 4. Hyprcursor
```
git clone https://github.com/hyprwm/hyprcursor
cd hyprcursor
cmake -B build
cmake --build build -j `nproc`
sudo cmake --install build
cd ..
```
### 5. Hyprgraphics (NEW REQUIREMENT)
```
git clone https://github.com/hyprwm/hyprgraphics
cd hyprgraphics
cmake -B build
cmake --build build -j `nproc`
sudo cmake --install build
cd ..
```
### 6. Aquamarine (NEW BACKEND - REQUIRED)
```
git clone https://github.com/hyprwm/aquamarine.git
cd aquamarine

cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_PREFIX_PATH=/usr/local
cmake --build build
sudo cmake --install build
sudo ldconfig

```
### 7. libxkbcommon
```
git clone https://github.com/xkbcommon/libxkbcommon
cd libxkbcommon
meson setup build --prefix=/usr/local
meson compile -C build
sudo meson install -C build
sudo ldconfig
```
### 8. wayland-protocols
```
git clone https://gitlab.freedesktop.org/wayland/wayland-protocols
cd wayland-protocols
meson setup build
meson compile -C build
sudo meson install -C build
```
# Example: install GCC 15 via PPA (adjust for your distro/version)
```
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install g++-15

export CC=gcc-15
export CXX=g++-15
rm -rf build && mkdir build && cd build
cmake ..
make
```
### 9. libdisplay-info
```
git clone https://gitlab.freedesktop.org/emersion/libdisplay-info.git
cd libdisplay-info

meson setup build
meson compile -C build
sudo meson install -C build
sudo ldconfig
```
### 10. hyprwire
```
git clone https://github.com/hyprwm/hyprwire
cd hyprwire
cmake -B build
cmake --build build -j$(nproc)
sudo cmake --install build
```
### 11. Build Hyprland
```
git clone --recursive https://github.com/hyprwm/Hyprland
cd Hyprland
make all
sudo make install
cd ..
```
---
4. Build xdg-desktop-portal-hyprland
Do not skip this, or screensharing and OBS will not work.
```
git clone https://github.com/hyprwm/xdg-desktop-portal-hyprland
cd xdg-desktop-portal-hyprland
cmake -B build
cmake --build build -j `nproc`
sudo cmake --install build
cd ..
```
## Final Config
### Standard tools
```
sudo apt install -y foot rofi-wayland waybar wl-clipboard grim slurp swaybg polkit-kde-agent-1
```
### Create config
```
mkdir -p ~/.config/hypr
cp /usr/local/share/hyprland/examples/hyprland.conf ~/.config/hypr/
```
