# Debian Hyprland Setup Plan
## document date 12 Jul 2026 - I can not confirm that everything work
---
1. Install Dependencies
Added libsdbus-c++-dev (for xdg-portal), libdisplay-info-dev (for aquamarine),
and fixed missing build tools.
```sh
# for dnf pkg
sudo dnf install -y epel-release
sudo dnf config-manager --set-enabled crb
sudo dnf install -y \
   meson ninja-build libinput-devel xcb-util-cursor-devel xcb-util-wm-devel \
   spirv-tools-devel libdisplay-info-devel file-devel iniparser-devel \
   git pkgconf-pkg-config cmake bison flex gcc gcc-c++ \
   glslang-devel libxkbcommon-devel lcms2-devel readline-devel \
   wayland-devel wayland-protocols-devel \
   libdrm-devel mesa-libgbm-devel libxkbcommon-devel pixman-devel \
   mesa-libEGL-devel mesa-libGLES-devel libseat-devel seatd \
   libxcb-devel librsvg2-devel libzip-devel \
   xcb-util-image-devel xcb-util-keysyms-devel xcb-util-renderutil-devel \
   pango-devel systemd-devel hwdata glslang \
   tomlplusplus-devel cairo-devel mesa-libGL-devel \
   sdbus-cpp-devel libliftoff-devel \
   pugixml-devel libXcursor-devel re2-devel libuuid-devel \
   glib2-devel pipewire-devel pipewire-jack-audio-connection-kit-devel

curl -LO https://pkgs.sysadmins.ws/el9/base/x86_64/muParser-2.3.4-1.el9.x86_64.rpm
curl -LO https://pkgs.sysadmins.ws/el9/base/x86_64/muParser-devel-2.3.4-1.el9.x86_64.rpm
sudo dnf install -y ./muParser-2.3.4-1.el9.x86_64.rpm ./muParser-devel-2.3.4-1.el9.x86_64.rpm
```

- for newer wayland, xkbcommon and wayland-protocols (Rocky Linux ships old versions,
Hyprland needs wayland>=1.25.0, xkbcommon>=1.11.0 and wayland-protocols>=1.49)
```sh
sudo dnf install -y expat-devel

cd /tmp

# wayland 1.25.0 (system has 1.24.0, too old for wayland-protocols 1.49)
git clone --depth 1 --branch 1.25.0 https://gitlab.freedesktop.org/wayland/wayland.git
cd wayland
meson setup build --prefix=/usr --buildtype=release \
  -Ddocumentation=false -Dtests=false
sudo meson install -C build
cd ..

# wayland-protocols 1.49 (needs wayland-scanner >= 1.25.0)
git clone --depth 1 --branch 1.49 https://gitlab.freedesktop.org/wayland/wayland-protocols.git
cd wayland-protocols
meson setup build --prefix=/usr --buildtype=release
sudo meson install -C build
cd ..

# xkbcommon 1.11.0 (system has 1.7.0, too old)
git clone --depth 1 --branch xkbcommon-1.11.0 https://github.com/xkbcommon/libxkbcommon.git
cd libxkbcommon
meson setup build --prefix=/usr --buildtype=release \
  -Denable-docs=false \
  -Denable-tools=false
sudo meson install -C build
sudo ldconfig
cd ..

# verify
pkg-config --modversion wayland           # should print 1.25.0
pkg-config --modversion wayland-protocols  # should print 1.49
pkg-config --modversion xkbcommon          # should print 1.11.0
```

- for Lua 5.5 (Rocky Linux only has 5.4, Hyprland requires 5.5)
```sh
cd /tmp
curl -L -o lua-5.5.0.tar.gz https://www.lua.org/ftp/lua-5.5.0.tar.gz
tar xzf lua-5.5.0.tar.gz
cd lua-5.5.0
make linux MYCFLAGS="-fPIC"
sudo make install
cd ..

# Lua's Makefile doesn't install a pkg-config file, create it manually
# (Hyprland cmake uses pkg_search_module to find Lua)
sudo tee /usr/lib64/pkgconfig/lua55.pc << 'EOF'
prefix=/usr/local
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: Lua
Description: An Extensible Extension Language
Version: 5.5.0
Libs: -L${libdir} -llua -lm -ldl
Cflags: -I${includedir}
EOF

# verify
lua -v                          # should print Lua 5.5.0
pkg-config --modversion lua55   # should print 5.5.0
```

```sh
# for apt pkg
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
# for apt pkg in dev branch
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

### 7. Hyprwire (required by hyprctl, not in wiki dep list)
```sh
git clone --depth 1 --branch v0.3.1 https://github.com/hyprwm/hyprwire.git
cd hyprwire

# GCC 14.x libstdc++ lacks std::vector::append_range (C++23 ranges).
# Apply compatibility patch that replaces append_range with insert(end, begin, end).
git apply /path/to/hyprwire-v0.3.1-cpp23-compat.patch

cmake --no-warn-unused-cli -DCMAKE_BUILD_TYPE:STRING=Release -DCMAKE_INSTALL_PREFIX:PATH=/usr -S . -B ./build
cmake --build ./build --config Release --target all -j`nproc 2>/dev/null || getconf NPROCESSORS_CONF`
sudo cmake --install build
sudo ldconfig
cd ..
```

### 8. Build Hyprland
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

