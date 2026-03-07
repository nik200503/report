# GSoC 2026 — Build a COSMIC-based Wayland Session for Regolith

## Setting up Testing Environment

1. Since COSMIC DE is not natively available on Ubuntu, use this PPA to install it:

   ```bash
   sudo add-apt-repository ppa:hepp3n/cosmic-epoch
   sudo apt update
   ```

2. Add last year's forked cosmic-settings:

   ```bash
   git clone https://github.com/sandptel/cosmic-settings
   cd cosmic-settings
   just install
   ```

3. Add last year's regolith-session and regolith-wm-config, and move these files to `/etc` and `/usr` respectively:

   ```bash
   git clone https://github.com/sandptel/regolith-session
   git clone https://github.com/sandptel/regolith-wm-config
   ```

4. Create a `.desktop` file at `/usr/share/wayland-sessions/` to register the session in GDM, then launch with:

   ```bash
   cosmic-session sway --config <path to cloned regolith-wm-config dir>/etc/regolith/sway/config -d
   ```

---

## Verifying Previous Year's Changes

### 1. Display Settings (including Appearance and UI)


**Status:** Partial

Display settings are not fully working due to a communication gap between COSMIC and Sway — resolution and refresh rate changes don't reach Sway's output management. This can be correctly implemented using glue code, for example via Sway IPC:

```
output <name> mode|resolution|res [--custom] <width>x<height>[@<rate>Hz]
```

A settings daemon like **cosmolith** can bridge this by translating COSMIC config changes into Sway IPC commands.

As for **Appearance and UI** — these are working as intended. Theme, accent colors, gaps, and active window hints are consistent and **persist across reboots**.

---

### 2. Input Settings


**Status:** Completed

Input settings for **touchpad**, **mouse**, and **keyboard** are working as intended. The forked `cosmic-settings` generates config files for any setting that is changed, and these are applied on every system reboot since they are included in the `regolith-wm-config` files via:

```
include $HOME/.config/regolith3/sway/cosmic-settings/generated-config.d/*
```

---

### 3. Power Settings


**Status:** Partial

Different power modes are working, but the **sleep timer** is not connected properly with `cosmic-idle`, which is responsible for implementing it. As for `cosmic-idle` itself — it is working perfectly and locks the screen after a period of inactivity.

---

### 4. regolith-wm-config


**Status:** Partial

Out of all planned changes, the following were completed and are working correctly as seen in the video:

**Completed:**
- Includes Generated Configs
- `cosmic-osd`
- `swaybg` → `cosmic-bg` (working with `cosmic-settings`)
- `sway-idle` → `cosmic-idle`
- `xdg-desktop-portal` → `xdg-desktop-portal-cosmic`

**To be implemented:**
- `ilia` → `cosmic-launcher`
- `swaybar` → `cosmic-panel`
- Use `cosmic-dock`
- Use `cosmic-applets`

---
[Here you can see recording of above changes](https://drive.google.com/drive/folders/1MLd2YbVoYVbVOEEtwLhbKebVQ5E6Ai5B?usp=drive_link)

## Way Ahead

### Cosmolith — A Better Architecture for the Bridge

Last year's GSoC work used the **generated-config** approach: the forked `cosmic-settings` app writes Sway config files and sends IPC commands directly. While functional for input settings, this approach has limitations:

- **Tightly coupled** — Every new setting requires modifying the `cosmic-settings` fork itself, making it harder to maintain as upstream cosmic-settings evolves.
- **Sway-only** — The IPC calls and config files are Sway-specific. Supporting any other compositor means duplicate work.
- **Fragile** — `SwayConnection::new().expect(…)` panics if Sway isn't running. No graceful fallback.

[**Cosmolith**](https://github.com/sandptel/cosmolith)  watches COSMIC's config store (`~/.config/cosmic/`) for changes and translates them into compositor-specific IPC calls in real time — **without modifying cosmic-settings at all**.

#### What's Already Done in Cosmolith

All input settings from last year's work are **already implemented** in cosmolith, so there is no need to start from scratch:

- **Touchpad:** acceleration, click method, disable-while-typing, left-handed, natural scroll, scroll config (method, factor), tap-to-click
- **Mouse:** acceleration, left-handed, natural scroll, scroll config
- **Keyboard:** layout, variant, XKB options, repeat rate/delay, numlock

### Remaining Work from GSoC 2025

The following items from the [GSoC 2025 checklist](https://github.com/sandptel/gsoc25) were not completed and are planned for this year:

**cosmic-settings → Sway bridge (via cosmolith):**
- [ ] Display Settings (resolution, refresh rate, output arrangement)
- [ ] Power Settings (sleep timer ↔ `cosmic-idle` integration)
- [ ] Workspace Management (fetch/manipulate Sway workspaces via IPC)
- [ ] Keyboard Shortcuts (complete implementation)
- [ ] Custom Settings Pages: Startup Applications, Window Rules, Default Applications

**regolith-wm-config:**
- [ ] `ilia` → `cosmic-launcher` (currently broken under Sway — may need glue code or a Sway-compatible fallback)
- [ ] `swaybar` → `cosmic-panel` (panel applets currently violate Sway drawing area rules)
- [ ] `cosmic-dock` integration
- [ ] `cosmic-applets` integration

**Other:**
- [ ] Trawl integration — bridge COSMIC settings with Regolith's configuration management system
- [ ] Packaging (`.deb` packages for Ubuntu and Debian)
- [ ] End-to-end testing and beta release


