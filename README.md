<p align="center">
  <img src="we_are_fsociety.png" width="180" alt="fsociety" />
</p>

<h1 align="center">MrRobotOS</h1>

<p align="center">
  <em>"The world itself is just one big machine. You know what the best part is? It can be hacked."</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/OS-MrRobotOS-red?style=flat-square&logo=linux&logoColor=white" />
  <img src="https://img.shields.io/badge/Base-Arch%20Linux-1793D1?style=flat-square&logo=archlinux&logoColor=white" />
  <img src="https://img.shields.io/badge/WM-mrdwm-black?style=flat-square" />
  <img src="https://img.shields.io/badge/Written%20in-C%20%7C%20GTK4%20%7C%20Shell-orange?style=flat-square" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" />
</p>

---

MrRobotOS is a fully custom Arch Linux-based operating system built from the ground up — not configured, not themed, **written**. Every component from the display manager to the status bar is original code, designed around a singular philosophy: total control, zero compromise, and an aesthetic that belongs to no one but itself.

This is not a rice. This is not a dotfiles collection. This is an operating system.

---

## Infrastructure

MrRobotOS is not just a desktop. It runs against a real self-hosted data center backend with elastic storage and compute. When demand grows, the infrastructure expands — n+1 scaling, not hard limits.

- **Unlimited storage** — backed by a self-hosted data center with capacity that scales horizontally on demand
- **Elastic compute** — resources expand as workloads require, no fixed ceiling
- **Home to rack** — the same OS that runs on your desk connects to the same infrastructure that runs your cloud

## Roadmap

### Coming Soon — Cloud & Storage Integration

- **Cloud integration** — manage VMs, containers, and remote services directly from the MrRobotOS desktop
- **Storage replication** — built-in replication across local and remote targets, designed for Proxmox and TrueNAS backends
- **Distributed storage** — redundancy and high availability across nodes
- **Backup and snapshot management** — scheduled snapshots and offsite backup from the desktop environment

> The goal is a desktop that is not just an interface to your local machine, but a control plane for your entire infrastructure — from your room to your rack.

---

## Why MrRobotOS Instead of KDE, GNOME, or Any Other Fat Desktop

Modern desktop environments are architecturally obese. KDE ships with hundreds of megabytes of Qt libraries, plasma frameworks, and services that run whether you need them or not. GNOME requires GNOME Shell, Mutter, dozens of background daemons, and a JavaScript engine embedded in the compositor just to render a panel. Their ISOs exceed 2–4 GB. Their idle RAM usage starts at 600 MB to 1.5 GB before you open a single application.

MrRobotOS does everything they do — window management, status bar, settings GUI, process manager, display manager, quick settings panel, notifications, compositor, launcher, terminal emulator — and does it in a fraction of the space, starting under 200 MB idle RAM.

The difference is not features. The difference is that every component in MrRobotOS was written to do exactly one thing, and nothing else. No plugin systems. No DBus activation for things that don't need it. No JavaScript interpreters embedded in the compositor. No abstract framework layers between the code and the hardware.

KDE and GNOME are platforms. MrRobotOS is a machine.

| | MrRobotOS | GNOME | KDE Plasma |
|---|---|---|---|
| Idle RAM | ~180 MB | ~800 MB | ~600 MB |
| ISO size | ~1.6 GB | ~2.5 GB | ~2.8 GB |
| Installed disk (minimal) | ~3–4 GB | ~4.2 GB | ~1.5–2 GB |
| Installed disk (full) | ~3–4 GB | ~4.2+ GB | ~6–9 GB |
| Display manager | mrlogin (custom C) | GDM | SDDM |
| Window manager | mrdwm (custom C) | Mutter + JS shell | KWin (Qt/C++) |
| Status bar | mrblocks (custom C) | GNOME Shell | Plasmashell |
| Settings app | mrsettings (GTK4 C) | Control Center | System Settings |
| Terminal emulator | mrst (custom C) | GNOME Terminal | Konsole |
| Boot to desktop | Fast | Slow | Slow |
| Fully hackable | Yes — read the source | Barely | Somewhat |

> KDE minimal = plasma-desktop only. KDE full = plasma-meta + kde-applications-meta (~6–9 GB). GNOME full installation footprint ~4.2 GB.

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                         MrRobotOS                            │
├──────────┬───────────┬───────────┬───────────────────────────┤
│ mrlogin  │   mrdwm   │ mrblocks  │        mrsystray          │
│ Display  │  Window   │  Status   │     Quick Settings        │
│ Manager  │  Manager  │   Bar     │         Panel             │
├──────────┴───────────┴───────────┴───────────────────────────┤
│   mrsettings    │   mrmanager   │          mrst              │
│  Settings GUI   │   Process     │      Terminal Emulator     │
│    (GTK4 C)     │   Monitor     │        (Xlib C)            │
├─────────────────┴───────────────┴────────────────────────────┤
│                         mrpanel                              │
│     XFCE4 Panel + 30+ Custom Genmon Scripts                  │
│  Window Overview · App Launcher · System Widgets · Controls  │
├──────────────────────────────────────────────────────────────┤
│                       mrdotfiles                             │
│       dunst · picom · conky · rofi · yazi · zsh              │
│              pulseaudio · xfce4 configuration                │
├──────────────────────────────────────────────────────────────┤
│              System Scripts + Calamares Installer            │
│                      Arch Linux (Base)                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Components

### mrlogin — Display Manager

A fully custom X11 display manager written in C using Xlib and Xft. No LightDM, no GDM — mrlogin owns the screen from the moment the system boots.

**Features:**
- Full-screen login UI with live clock updated every second and an animated dot grid background
- Multi-user support with clickable avatar cards and Tab-key switching
- PAM authentication via `pam_start`, `pam_authenticate`, `pam_acct_mgmt`
- Progressive lockout: 1 minute → 5 minutes → 15 minutes → 1 hour → permanent device disable
- Visual attempt tracker rendered as colored arc dots — one dot fills red per failed attempt
- Password visibility toggle with Font Awesome eye / eye-slash icon
- Cursor-aware hover detection: pointer changes to hand on all interactive elements
- Rounded card UI with accent header bar, drop shadow, and inline error messaging
- Launches user session via `dbus-run-session` under a zsh login shell sourcing `~/.xinitrc`
- Full environment setup: `XDG_RUNTIME_DIR` creation with correct ownership, `initgroups`, `setuid`, `setgid`, `chdir`
- Primary monitor detection via `xrandr` for correct geometry on multi-monitor setups

---

### mrdwm — Window Manager

A heavily patched and deeply extended fork of [dwm](https://dwm.suckless.org/), pushed far beyond the patch ecosystem into original territory. Written in C against Xlib and XCB.

**Layout Engine:**
Tile, Monocle, Deck, Grid, Fibonacci Spiral, Fibonacci Dwindle, Centered Master, Centered Floating Master, Three-Column (TCL). Adjustable gaps with per-monitor state. Master/stack ratio and master count control.

**Bar:**
- Per-client tabbed title area with proportional width distribution and pixel-accurate remainder handling
- Window icons rendered via `_NET_WM_ICON` with Imlib2 rescaling and XRender compositing
- Hover highlighting on title tabs — focus follows mouse over the bar
- Nine color schemes: Normal, Selected, Hover, Hidden, Status, Tags Selected, Tags Normal, Info Selected, Info Normal
- Status text with clickable signal routing to mrblocks via `SIGRTMIN+N`
- Configurable vertical padding, side padding, and per-monitor bar geometry

**Tag Preview System:**
- Hover over any tag to get a live scaled screenshot preview of its contents using Imlib2
- Full window overview triggered via SIGUSR1: captures all client windows using XRender compositing, scales them into a responsive grid, and overlays them as a selectable switcher
- Grid layout adapts to window count — 1 window gets 80% of screen, 2 windows side by side, 3+ arranged in computed rows and columns with configurable gaps
- Click any preview to jump to that window's tag; keyboard press or right-click to dismiss without switching
- Panel-aware: panels are raised above the preview overlay and excluded from the capture grid
- Stale previews are freed when tags become unoccupied

**Window Management:**
- Window swallowing: a terminal spawning a GUI app disappears and the GUI takes its visual place, restoring the terminal on close
- Per-window show/hide using `IconicState` — works as per-window scratchpads
- Panel detection by window name: zero borders, pinned to bottom, always visible across all tags
- Urgent hint propagation, EWMH compliance: fullscreen, active window, window type, client list

**Multi-monitor:**
Full Xinerama support with unique geometry deduplication. Per-monitor independent tag sets, layout state, and bar position. Focus follows mouse across monitor boundaries.

---

### mrblocks — Status Bar Daemon

A from-scratch status bar block runner written in C. Not a shell script scheduler — a proper event-driven daemon with `poll`, `signalfd`, and pipe-based IPC.

**Architecture:**
- Each block forks a subprocess connected via a dedicated pipe pair
- `poll`-based watcher monitors all block read-ends and the signal file descriptor simultaneously with no busy waiting
- Timer computed as GCD of all block intervals, minimizing `SIGALRM` firings
- `signalfd` handles `SIGALRM`, `SIGRTMIN+N`, `SIGINT`, `SIGTERM`, and `REFRESH_SIGNAL` in a unified read loop
- No zombie processes: `waitpid` called after every block read
- UTF-8 aware output truncation per block
- Status string written to X root window name via XCB only when content changes

**Active Blocks:**

| Script | Signal | Function |
|--------|--------|----------|
| `sb-datetime` | SIGRTMIN+1 | Date and time with Nerd Font clock icon |
| `sb-wifi` | SIGRTMIN+2 | Wi-Fi signal strength with 4-level Nerd Font icons, ethernet, VPN |
| `sb-bluetooth` | SIGRTMIN+3 | Bluetooth power state and connected device name |
| `sb-volume` | SIGRTMIN+4 | PulseAudio volume with 4-state icons, scroll to adjust, middle to mute |
| `sb-battery` | SIGRTMIN+5 | Battery level with separate charging/discharging icon sets, critical alert |

**Additional Included Block Scripts:**

| Script | Function |
|--------|----------|
| `sb-brightness` | xrandr brightness with scroll and notify-send |
| `sb-cpu` | CPU temperature via lm-sensors |
| `sb-cpubars` | Per-core load as Unicode block characters (▁▂▃▄▅▆▇█) cached in tmpfs |
| `sb-memory` | RAM usage in GiB |
| `sb-disk` | Disk usage per mountpoint |
| `sb-internet` | Wi-Fi/ethernet/VPN status via `/proc/net/wireless` |
| `sb-packages` | Pending pacman upgrade count |
| `sb-moonphase` | Current moon phase from wttr.in cached daily |
| `sb-weather` | Precipitation chance, daily low and high from wttr.in |
| `sb-doppler` | Animated Doppler radar via mpv for US/EU/Africa/Germany regions |
| `sb-network` | Per-interface RX/TX bytes formatted with `numfmt` |
| `sb-mpd` | mpd now-playing via mpc with SIGUSR1-based update loop daemon |
| `sb-mail` | Unread mail count from local maildir |
| `sb-news` | Unread newsboat RSS item count |
| `sb-tasks` | Background task queue count via tsp |
| `sb-torrents` | Transmission torrent state summary |
| `sb-crypto` | Cryptocurrency price from rate.sx with chart display |
| `sb-kbselect` | Keyboard layout selector via dmenu and setxkbmap |
| `sb-geoip` | Public IP geolocation with country flag emoji |

---

### mrst — Terminal Emulator

A heavily patched fork of [st](https://st.suckless.org/) extended with scrollback history, improved rendering, and mrdwm window swallowing integration. Written in C against Xlib.

**Features:**
- 2000-line scrollback history ring buffer with `kscrollup` / `kscrolldown` keybindings
- History-aware line rendering via `TLINE(y)` macro that reads from the ring buffer when scrolled back
- Selection coordinates adjusted during scroll via `selscroll` — selection survives scrolling
- `tscrollup` / `tscrolldown` extended with `copyhist` parameter controlling whether lines enter the history ring
- XRender-based window surface capture for mrdwm preview system: `getwindowximage` composites the terminal with alpha blending via `XRenderComposite`, then scales via `scaleimagetofit` using nearest-neighbor pixel mapping
- Full UTF-8 decode/encode pipeline with surrogate pair rejection and replacement character fallback
- VT100 graphic character translation table: box drawing, arrows, symbols
- Complete CSI, OSC, DCS, APC, PM escape sequence handling
- True color (24-bit RGB) and 256-color indexed support
- Bracketed paste mode, focus event reporting, mouse reporting (X10, button, motion, SGR extended)
- Alternate screen with independent cursor save/restore
- PID reporting via XCB `xcb_res_query_client_ids` for mrdwm window swallowing PID matching
- PTY resize propagated via `TIOCSWINSZ` on every X configure event

---

### mrsettings — Settings Manager

A full GTK4 settings application for configuring the entire desktop environment. Written in C with a custom GtkListBox sidebar replacing GtkStackSidebar for full visual control.

**Pages:**
- **Welcome** — MrRobotOS branding with fsociety artwork
- **Key Bindings** — Full mrdwm keybinding reference as a scrollable grid with separator rows
- **MrDWM Configurations** — Live color scheme editor for all 9 scheme groups, each with three GtkColorButton pickers (background, foreground, border). Changes written directly to `colors.h` via `sed` in-place substitution
- **Volume & Audio** — Full PulseAudio device management: per-channel volume sliders 0–153% with live dB readout, port dropdown sorted by availability and priority, mute toggle, channel lock, default device selection via `pactl`. Separate tabs for output and input
- **Time & Language** — Timezone picker populated from `find /usr/share/zoneinfo` with search and current timezone pre-selected
- **Applications** — Full installed app list via GIO `g_app_info_get_all` with icons and double-click launch
- **Network Settings** — Live interface status via nmcli with Wi-Fi AP discovery
- **Panel Configuration** — Reads XFCE4 panel XML for RGBA background values, writes back with precise 17-digit float serialization

---

### mrmanager — Process Manager

A GTK4 process manager with live hierarchical tree view. Written in C.

**Features:**
- Full process tree built by matching PPID relationships recursively, rendered as a collapsible GtkTreeView with GtkTreeStore
- All 12 columns sortable with sort state preserved across auto-refreshes: PID, PPID, USER, PRI, NI, VSZ, BSDTIME, S, CLS, PCPU, PMEM, COMM
- Auto-refresh every 3 seconds for both system summary and tree
- System header: uptime, CPU breakdown from `/proc/stat`, memory and swap via `free`
- Process counts: running, sleeping (idle+sleeping), stopped, zombie
- Per-process action dialog: terminate with confirmation, show details
- Recursive tree search across full hierarchy for child PID lookup

---

### mrsystray — Quick Settings Panel

A GNOME-style quick settings overlay written in GTK4 C. Single GApplication instance — subsequent launches toggle visibility.

**Features:**
- User avatar, username, and hostname in the header
- Live clock and date updated every second
- Wi-Fi toggle via `nmcli radio wifi` with active SSID display
- Bluetooth toggle via `bluetoothctl` with connected device name
- Volume slider controlling PulseAudio default sink via `pactl`
- Brightness slider via `brightnessctl`
- Power row: Lock, Logout, Reboot, Shutdown
- Focus-loss dismissal via GtkEventControllerFocus
- X11 positioning via `XMoveWindow` to top-right corner
- Full custom CSS: dark frosted-glass card `rgba(36,36,36,0.95)`, blue active state `#3584e4`, red shutdown accent

---

### mrpanel — Taskbar and Desktop Shell

A custom XFCE4 panel configuration serving as the bottom taskbar and desktop control surface, extended with over 30 original genmon scripts that transform it into a complete desktop shell.

**Window Overview and Application Launcher:**
The panel includes a GNOME Activities / xfdashboard-style window overview and application launcher. A single button triggers a full-screen overlay showing all open windows as live scaled thumbnails — powered by mrdwm's SIGUSR1 preview capture system — combined with an application grid for launching new apps. Task switching and app launching unified in one gesture, without GNOME Shell, without Mutter, without JavaScript.

**Genmon Panel Scripts:**

| Script | Function |
|--------|----------|
| `datetime-panel.sh` | Clock and date with click-to-calendar |
| `datetime.sh` | Compact time format |
| `battery-panel.sh` | Battery with charging state icons |
| `wifi-panel.sh` | Wi-Fi SSID and signal strength |
| `network-panel.sh` | Network interface status |
| `bandwith-panel.sh` | Real-time RX/TX bandwidth |
| `cpu-panel.sh` | CPU usage with color threshold |
| `memory-panel.sh` | Live RAM readout |
| `disk-panel.sh` | Disk usage per mountpoint |
| `kernel-panel.sh` | Running kernel version |
| `pacman-panel.sh` | Pending upgrade count |
| `spotify-panel.sh` | Spotify now-playing via playerctl |
| `spotify.sh` | Compact track info |
| `power-panel.sh` | Shutdown/reboot/suspend quick access |
| `search-panel.sh` | Rofi search launcher |
| `menu.sh` | Application menu |
| `window-title.sh` | Active window title |
| `show-desktop.sh` | Show/hide all windows |
| `minimize-button.sh` | Minimize focused window |
| `close-button.sh` | Close focused window |
| `die-panel.sh` | Force-kill focused window |
| `eject-panel.sh` | Removable media eject |
| `eject.sh` | Eject helper |
| `gamepad-panel.sh` | Gamepad connection status |
| `smart-watch-battery-panel.sh` | Smartwatch battery via Bluetooth |
| `adb-info.sh` | Android device info via ADB |
| `rofi-todo.sh` | Quick todo entry via rofi |
| `clean.sh` / `cleaner-panel.sh` | Cache and temp cleanup |

Each script outputs Pango markup for rich text inside XFCE4 genmon, with click handlers wired to system actions, notify-send notifications, or application launches.

---

### mrdotfiles — Environment Configuration

| Component | Role |
|-----------|------|
| **dunst** | Notification daemon with custom theming matching mrdwm color schemes |
| **picom** | Compositor: shadows, transparency, rounded corners, blur |
| **conky** | System statistics overlay |
| **rofi** | Application launcher, dmenu replacement, todo interface |
| **yazi** | Terminal file manager |
| **zsh** | Shell under `$ZDOTDIR` with custom prompt, completions, and history |
| **pulseaudio** | Audio system configuration |
| **xfce4** | Panel and auxiliary desktop tools |

---

### System Scripts

A collection of system-level utility scripts forming the connective tissue between GUI components and the underlying system. They handle display configuration, network operations, audio routing, backup management, power control, hardware detection, and environment bootstrapping — designed to be called directly by panel widgets, keybindings, and application launchers without unnecessary intermediary processes.

---

### Calamares Installer

MrRobotOS ships with a custom Calamares-based graphical installer for installation from a live ISO onto any compatible x86_64 machine. Handles partitioning, bootloader setup, locale configuration, user creation, and MrRobotOS-specific post-install customization.

---

## Keybindings

<details>
<summary><b>mrdwm keybinding reference</b></summary>

| Action | Binding |
|--------|---------|
| Launch Terminal | `MODKEY + Shift + Enter` |
| Launch Dmenu/Rofi | `MODKEY + p` |
| Close Window | `MODKEY + Shift + c` |
| Toggle Bar | `MODKEY + b` |
| Focus Stack ↓ visible | `MODKEY + j` |
| Focus Stack ↑ visible | `MODKEY + k` |
| Focus Stack ↓ hidden | `MODKEY + Shift + j` |
| Focus Stack ↑ hidden | `MODKEY + Shift + k` |
| Increase Masters | `MODKEY + i` |
| Decrease Masters | `MODKEY + d` |
| Shrink Master | `MODKEY + h` |
| Grow Master | `MODKEY + l` |
| Zoom to Master | `MODKEY + Return` |
| Previous Tag | `MODKEY + Tab` |
| Tiled Layout | `MODKEY + t` |
| Floating Layout | `MODKEY + f` |
| Monocle Layout | `MODKEY + m` |
| Cycle Layout | `MODKEY + Space` |
| Toggle Floating | `MODKEY + Shift + Space` |
| Show Window | `MODKEY + s` |
| Show All | `MODKEY + Shift + s` |
| Hide Window | `MODKEY + Shift + h` |
| Decrease Gaps | `MODKEY + Minus` |
| Increase Gaps | `MODKEY + Equal` |
| Reset Gaps | `MODKEY + Shift + Equal` |
| View All Tags | `MODKEY + 0` |
| Tag All | `MODKEY + Shift + 0` |
| Focus Monitor ← | `MODKEY + Comma` |
| Focus Monitor → | `MODKEY + Period` |
| Tag to Monitor ← | `MODKEY + Shift + Comma` |
| Tag to Monitor → | `MODKEY + Shift + Period` |
| Workspace 1–9 | `MODKEY + 1–9` |
| Quit mrdwm | `MODKEY + Shift + q` |
| Restart mrdwm | `MODKEY + Ctrl + Shift + q` |

</details>

---

## Philosophy

MrRobotOS exists at the intersection of two truths: that a computer is a machine, and that machines can be owned completely. Every line in this repository was written with the understanding that abstraction is a choice — and that choosing to go deeper is always available to those willing to read the source.

KDE gives you settings dialogs. MrRobotOS gives you `colors.h`. GNOME gives you extensions. MrRobotOS gives you `dwm.c`.

The aesthetic is intentional. The Mr. Robot theme is not decoration. It is a reminder that the systems we interact with every day are not fixed — they are written by people, and they can be rewritten.

This is one person's complete vision of what a desktop operating system should be: minimal in dependency, maximal in control, and honest about every decision it makes.

---

## Author

<p align="center">
  <b>Merih Bora Poçan</b> — Istanbul, Turkey<br/>
  Built across years of late nights, early mornings, and the kind of focus that makes the outside world feel optional.
</p>

<p align="center">
  <em>"Hello, friend."</em>
</p>
