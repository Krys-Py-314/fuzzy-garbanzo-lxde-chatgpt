# Raspberry Pi 5 Minimal LXDE / Openbox Setup

Minimal-memory-footprint **LXDE + Openbox** desktop environment for a **Raspberry Pi 5** running **Raspberry Pi OS Lite 64-bit (Trixie / arm64)**.

The goal is a reproducible X11 desktop with the smallest practical idle-memory footprint while still providing:

- LXDE desktop environment
- Openbox window manager
- Development tools for C/C++
- Lightweight terminal, editor, file manager and browser
- Raspberry Pi GPIO development support
- Dropbear SSH server
- Pi-Apps
- Basic desktop utilities
- Dark, square-window visual configuration
- Nerd Font + shell prompt tooling

The installation script is:

```text
install-minimal-lxde-rpi5.sh
```

---

## Design goal

The priority is **minimum idle RAM**, not minimum disk usage.

For that reason the script avoids the complete `lxde` desktop metapackage and instead builds around the smaller LXDE core components.

It also deliberately avoids unnecessary resident desktop services such as:

- LightDM
- Picom / compositors
- screensavers
- full desktop portal stacks
- heavyweight default browsers
- unnecessary desktop utilities

LXDE is started manually from the console using:

```bash
startx
```

---

# Main component choices

| Function | Choice | Reason |
|---|---|---|
| Desktop | `lxde-core` | Minimal LXDE components rather than the complete LXDE metapackage |
| Window manager | Openbox | Native LXDE choice and very small |
| Terminal | LXTerminal | Lightweight LXDE terminal |
| File manager | PCManFM | Native LXDE file manager |
| Text editor | L3afpad | Small GTK text editor |
| Browser | NetSurf GTK | Designed for a small resource footprint |
| Application menu | jgmenu | Lightweight X11 application menu |
| Notifications | dunst | Lightweight notification daemon |
| PDF viewer | qpdfview | Lightweight PDF viewer |
| Video player | mpv | Lightweight and capable media player |
| Screen recorder | SimpleScreenRecorder | X11-oriented screen recorder |
| SSH server | Dropbear | Replaces OpenSSH server |
| GPIO development | `libgpiod-dev`, `gpiod`, Raspberry Pi GPIO development packages | Appropriate GPIO development stack for Raspberry Pi 5 |

The script installs packages using:

```bash
apt-get install --no-install-recommends
```

to avoid pulling in unnecessary recommended packages.

---

# Target system

The script is intended for:

- Raspberry Pi 5
- Raspberry Pi OS Lite
- 64-bit / `arm64`
- Debian Trixie base
- X11

The script deliberately checks the platform rather than silently assuming compatibility with other Raspberry Pi OS releases.

---

# Package verification

Before installing the desktop environment, the script performs:

```bash
sudo apt-get update
```

It then checks the required packages with `apt-cache`.

If one or more packages are not present in the configured Raspberry Pi OS repositories, the script reports the missing packages and aborts before beginning the main installation.

This is intended to prevent an installation from progressing halfway before ending with errors such as:

```text
Error: Unable to locate package ...
```

---

# Desktop environment

The installation uses the minimal LXDE session rather than the complete LXDE desktop package.

The session is based around:

- LXSession
- LXPanel
- PCManFM
- Openbox

No display manager is installed.

After booting to the console, start X11 and LXDE with:

```bash
startx
```

---

# Openbox configuration

Openbox is configured for a deliberately simple, square visual style.

## Window decoration

- 1 pixel window border
- No window title bars
- No rounded corners
- No compositor effects
- Active window border: light blue
- Inactive window border: very dark

The approximate border colours are:

```text
Active:   #5DADE2
Inactive: #17191F
```

Openbox keeps the window border while disabling title decorations.

---

# Desktop colours

The desktop background is configured to exactly:

```text
RGB: 56, 60, 72
HEX: #383C48
```

This replaces a pure black desktop background.

The jgmenu application menu background is configured to approximately:

```text
RGB: 38, 38, 38
HEX: #262626
```

---

# Icons and themes

## Icons

jgmenu and the GTK desktop use:

```text
Numix-Circle
```

where available.

## GTK theme

GTK applications use a dark Numix-based configuration.

## Qt applications

Qt applications use **Kvantum** with an installed dark Kvantum theme.

Kvantum is a Qt style engine and therefore cannot replace the GTK theme directly.

The configuration is therefore intentionally split:

```text
GTK applications -> Numix / dark GTK configuration
Qt applications  -> Kvantum dark theme
Openbox           -> custom PiMinimal theme
jgmenu            -> explicit dark colour configuration
```

The script searches the installed Kvantum themes rather than relying entirely on one hard-coded theme name.

---

# X11 configuration for Raspberry Pi 5

The script explicitly creates:

```text
/etc/X11/xorg.conf.d/99-vc4.conf
```

with:

```text
Section "OutputClass"
    Identifier "vc4"
    MatchDriver "vc4"
    Driver "modesetting"
    Option "PrimaryGPU" "true"
EndSection
```

The file is created with:

```bash
sudo mkdir -p /etc/X11/xorg.conf.d

sudo tee /etc/X11/xorg.conf.d/99-vc4.conf >/dev/null <<'EOF'
Section "OutputClass"
    Identifier "vc4"
    MatchDriver "vc4"
    Driver "modesetting"
    Option "PrimaryGPU" "true"
EndSection
EOF
```

The configuration uses the Xorg **modesetting** driver with the Raspberry Pi VC4 graphics stack.

`xserver-xorg-video-fbdev` is therefore not intentionally used as the primary Raspberry Pi 5 graphics driver.

---

# Applications

## Terminal

```text
LXTerminal
```

Chosen because it integrates naturally with LXDE and has a small footprint.

---

## File manager

```text
PCManFM
```

PCManFM is the native LXDE file manager and also participates in desktop handling.

---

## Text editor

```text
L3afpad
```

Selected as a very small graphical GTK text editor.

Geany is also installed because the requested Pi-Apps **Geany Dark Mode** package requires it.

---

## Web browser

The lightweight browser installed directly from the Raspberry Pi OS / Debian repositories is:

```text
NetSurf GTK
```

The script additionally installs **Min** through Pi-Apps as requested.

---

# Development environment

The script installs the essential C/C++ development toolchain, including:

```text
gcc
g++
make
```

plus the supporting development tools required by the system and requested applications.

The intent is to provide a usable Raspberry Pi C development environment without installing a large IDE stack.

---

# Raspberry Pi utilities

The installation includes Raspberry Pi command-line configuration and utility packages, including the available Raspberry Pi OS equivalents of:

```text
raspi-config
raspberrypi-utils / rasp-utils-core
```

The exact package availability is checked before installation.

---

# Raspberry Pi GPIO development

The Raspberry Pi 5 uses the modern Linux GPIO character-device interface.

The development environment therefore includes the appropriate GPIO packages such as:

```text
gpiod
libgpiod-dev
```

and Raspberry Pi GPIO development support available from the Raspberry Pi OS repositories.

This provides the headers and libraries required for C GPIO development.

---

# Pi-Apps

Pi-Apps is installed using the requested official installation command:

```bash
wget -qO- https://raw.githubusercontent.com/Botspot/pi-apps/master/install | bash
```

After Pi-Apps is installed, the script performs:

```bash
~/pi-apps/manage install Min
~/pi-apps/manage install "Geany Dark Mode"
```

Pi-Apps project:

https://github.com/Botspot/pi-apps

---

# SSH server

The script uses:

```text
Dropbear
```

instead of the OpenSSH server.

`openssh-server` is removed and Dropbear is enabled on the standard SSH port.

The OpenSSH **client** is intentionally retained.

Removing `openssh-client` would save no idle RAM because it has no resident daemon, while removing it would unnecessarily break useful development operations such as:

```bash
git clone git@github.com:...
ssh hostname
scp file hostname:
```

---

# Additional utilities

The setup includes lightweight tools for the common tasks requested.

## Screen recorder

```text
SimpleScreenRecorder
```

## Raspberry Pi Imager

The Raspberry Pi Imager package is installed when available from the configured Raspberry Pi OS repositories.

## PDF viewer

```text
qpdfview
```

## Video player

```text
mpv
```

## Command-line tools

```text
fastfetch
git
curl
wget
```

---

# Nerd Font

The setup installs the **Ubuntu Nerd Font** system-wide.

Fonts are installed beneath:

```text
/usr/local/share/fonts/
```

and the font cache is rebuilt using:

```bash
fc-cache
```

The desktop font size is configured to:

```text
11 pt
```

The script uses fontconfig utilities where useful to identify the installed font family rather than relying only on the downloaded filename.

Nerd Fonts:

https://www.nerdfonts.com/

---

# Bash PATH

The following requested line is added to the user's `.bashrc`:

```bash
export PATH=$PATH:$HOME/.local/bin
```

This allows locally installed user utilities such as shell-prompt programs to be invoked normally.

---

# Oh My Posh and Starship

Both Oh My Posh and Starship can replace the Bash prompt, so initializing both simultaneously would cause them to compete for the same prompt.

The setup therefore uses the following arrangement:

- Oh My Posh is installed
- Oh My Posh is initialized for Bash
- the default Oh My Posh configuration is used
- Starship is installed
- Starship remains available but is not simultaneously initialized

This keeps both requested tools installed without configuring two prompt engines at the same time.

Oh My Posh:

https://ohmyposh.dev/

Starship:

https://starship.rs/

---

# jgmenu

jgmenu is used as the lightweight application launcher.

The menu uses:

```text
Background: #262626
Icons:      Numix-Circle
```

Suggested shortcuts configured by the setup include:

```text
Super + Space  -> jgmenu
Super + Enter  -> LXTerminal
Alt   + F4     -> close current window
Alt + Left drag  -> move window
Alt + Right drag -> resize window
```

---

# Installation

Start with a fresh Raspberry Pi OS Lite 64-bit installation.

Clone or copy the script onto the Raspberry Pi.

Make it executable:

```bash
chmod +x install-minimal-lxde-rpi5.sh
```

Run it as the normal user:

```bash
./install-minimal-lxde-rpi5.sh
```

Do **not** run the script directly as root.

The script itself uses `sudo` when elevated privileges are required.

After installation:

```bash
sudo reboot
```

Log in at the console and start the desktop with:

```bash
startx
```

---

# Progress messages

The script begins with the requested coloured status functions:

```bash
# Colors for output

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

print_status() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

print_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}
```

It also refuses execution as root:

```bash
if [ "$EUID" -eq 0 ]; then
    print_error "Please do not run this script as root. Run as normal user with sudo privileges."
    exit 1
fi
```

and begins with:

```text
Starting Raspberry Pi 5 Minimal LXDE/Openbox Setup...
```

---

# Memory-footprint philosophy

The script follows several rules intended to keep memory consumption low:

1. Do not install a graphical display manager.
2. Use `lxde-core` rather than the complete LXDE metapackage.
3. Use Openbox without a compositor.
4. Avoid heavyweight desktop services.
5. Avoid unnecessary background daemons.
6. Use lightweight applications where practical.
7. Install packages with `--no-install-recommends`.
8. Keep OpenSSH client tools but replace the OpenSSH server daemon with Dropbear.
9. Start X11 only when a graphical session is actually required.

This means the Raspberry Pi can boot to a low-memory console environment and the GUI can be started only when needed.

---

# Measuring memory usage

An exact idle-RAM value is intentionally not stated here because it should be measured on the actual Raspberry Pi 5 after installation.

After booting into LXDE, use:

```bash
free -h
```

and:

```bash
ps -eo pid,comm,rss --sort=-rss | head -25
```

These commands provide both overall memory consumption and the largest resident processes.

This is the correct basis for a second optimization pass if an even smaller runtime footprint is required.

---

# Notes

## No LightDM

There is intentionally no graphical login manager.

Use:

```bash
startx
```

from the console.

## No compositor

No compositor such as Picom is installed by default.

This avoids additional resident memory and preserves the simple X11/Openbox environment.

## Square interface

The configuration deliberately avoids cosmetic effects such as transparency and rounded corners.

The visual design is intentionally simple:

- square windows
- one-pixel borders
- no title bars
- dark neutral surfaces
- blue active-window indication

---

# References

Relevant upstream projects and package information:

- Raspberry Pi documentation: https://www.raspberrypi.com/documentation/
- Raspberry Pi OS: https://www.raspberrypi.com/software/
- Debian packages: https://packages.debian.org/
- LXDE: https://www.lxde.org/
- Openbox: http://openbox.org/
- PCManFM: https://wiki.lxde.org/en/PCManFM
- NetSurf: https://www.netsurf-browser.org/
- jgmenu: https://github.com/johanmalm/jgmenu
- Pi-Apps: https://github.com/Botspot/pi-apps
- Nerd Fonts: https://www.nerdfonts.com/
- Oh My Posh: https://ohmyposh.dev/
- Starship: https://starship.rs/
- libgpiod: https://git.kernel.org/pub/scm/libs/libgpiod/libgpiod.git/

---

# License

No license is imposed by this README.

If this repository is intended to be public, add an appropriate `LICENSE` file separately.

