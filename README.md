# RootOptimizer

A Windows/Linux system debloat and performance-tuning toolkit, shipped as
four **independent, single-file scripts** instead of one bundled program.
Pick the tier that matches your comfort level — each one stands alone and
does not depend on any of the others.

| File | Platform | Tier | What it is |
|---|---|---|---|
| `optimizer_v1.bat` | Windows | v1 — basic | A fixed, linear sequence. No menu, no choices to make. The smallest and safest feature set. |
| `optimizer_v2.cmd` | Windows | v2 — mid | A menu-driven CMD script. Pure batch, with a handful of calls out to PowerShell only where CMD genuinely has no equivalent (Appx package removal, network adapter toggling). |
| `optimizer_v3.ps1` | Windows | v3 — full | The most capable tier. Full menu, structured logging, process trimming, driver reporting, a live process monitor, and every other feature in this toolkit. |
| `optimizer_linux.sh` | Linux | — | The closest equivalent feature set for Linux, run as root via `sudo`. |

Each script is self-contained: you can copy just the one you want and run it
on its own machine without the others.

---

## Table of contents

- [Requirements](#requirements)
- [Entering Safe Mode (Windows)](#entering-safe-mode-windows)
- [Running the scripts](#running-the-scripts)
- [What each tier can do](#what-each-tier-can-do)
- [Safety design](#safety-design)
- [Logging](#logging)
- [FAQ / troubleshooting](#faq--troubleshooting)
- [License](#license)

---

## Requirements

- **Windows tiers (`v1`, `v2`, `v3`):** Windows 10 or 11, administrator
  rights, and the machine booted into **Safe Mode** (see below). PowerShell
  5.1+ is required for `v2`'s PowerShell calls and for `v3` itself; it ships
  by default on all supported Windows versions.
- **Linux script:** any modern distro with `bash`, `sudo`/root access.
  Some features are best-effort depending on what's installed — the script
  checks for `cpupower`, `rfkill`, `nmcli`, `gsettings`, `htop`, etc. and
  skips gracefully if a given tool isn't present.

## Entering Safe Mode (Windows)

All three Windows scripts check whether the machine is currently running in
Safe Mode, and refuse to continue if it isn't — this is intentional (see
[Safety design](#safety-design)). You only need to be in Safe Mode while
you're actually running the script; you can reboot normally as soon as it
finishes. Three ways to get there:

**Method 1 — Shift + Restart (fastest, works from a normal desktop session)**

1. Open the Start menu.
2. Hold **Shift** on your keyboard, then click **Power → Restart**.
3. Windows reboots into the **Choose an option** blue screen.
4. Select **Troubleshoot**.
5. Select **Advanced options**.
6. Select **Startup Settings**.
7. Click **Restart**.
8. After the reboot, you'll see a numbered list of startup options. Press
   **4** (or **F4**) for **Enable Safe Mode**. Press **5** / **F5** instead
   if you specifically need Safe Mode with Networking (not required by any
   script here, but harmless if you choose it).
9. Windows boots into Safe Mode. Log in, then run the script.

**Method 2 — `msconfig` (if you can't get to the Start menu normally)**

1. Press **Win + R**, type `msconfig`, press Enter.
2. Go to the **Boot** tab.
3. Check **Safe boot**, leave it set to **Minimal**.
4. Click **OK**, then **Restart** when prompted.
5. Windows now boots into Safe Mode on every restart **until you undo this**
   — go back into `msconfig` → Boot tab → uncheck **Safe boot** → Restart,
   once you're done running the script, or your PC will keep booting into
   Safe Mode.

**Method 3 — From a black/unbootable screen (recovery environment)**

If Windows won't boot normally at all, interrupting the boot process three
times in a row (hold the power button to force a shutdown while the Windows
logo is showing, three times) will trigger **Automatic Repair**, which drops
you into the same **Choose an option** screen as Method 1 — from there,
follow steps 4–9 above.

## Running the scripts

**Windows (any tier) — from Safe Mode:**

```bat
:: Right-click the file and choose "Run as administrator", or just double-click
:: and accept the UAC prompt — all three scripts self-elevate automatically.
optimizer_v1.bat
optimizer_v2.cmd
optimizer_v3.ps1
```

If `optimizer_v3.ps1` is blocked by PowerShell's execution policy, run it
from an elevated PowerShell prompt instead:

```powershell
powershell -ExecutionPolicy Bypass -File optimizer_v3.ps1
```

**Linux:**

```bash
chmod +x optimizer_linux.sh
sudo ./optimizer_linux.sh
```

## What each tier can do

| Feature | v1 (bat) | v2 (cmd) | v3 (ps1) | Linux (sh) |
|---|---|---|---|---|
| Ultimate Performance power plan / CPU governor | ✅ | ✅ | ✅ | ✅ (performance governor) |
| Force max CPU performance state | ❌ | ❌ | ✅ | — (covered by governor) |
| Full temp/cache cleanup | ✅ | ✅ | ✅ | ✅ |
| Bloatware removal (security apps excluded) | ❌ | ✅ | ✅ | — (distro-dependent, not included) |
| Disable Windows Copilot | ✅ | ✅ | ✅ | n/a |
| Dark mode | ✅ | ✅ | ✅ | ✅ (GNOME only) |
| Safe boot menu tweaks (timeout, menu policy) | ✅ | ✅ | ✅ | n/a |
| Wi-Fi / Bluetooth adapter toggle | ❌ | ✅ | ✅ | ✅ |
| Live process monitor window | ❌ | ✅ | ✅ | ✅ |
| Driver/hardware report + update scan | ❌ | ✅ | ✅ | ✅ (lspci/lsmod dump) |
| Disable pagefile / swap (advanced, risky) | ❌ | ❌ | ✅ | ✅ |
| Trim to essential processes only (advanced) | ❌ | ✅ | ✅ | ✅ |
| Menu-driven interface | ❌ (linear) | ✅ | ✅ | ✅ |
| "Run recommended set" one-shot option | ❌ | ❌ | ✅ | ❌ |

## Safety design

These are deliberate design choices, not omissions — please read before
assuming a missing feature is a bug:

- **Safe Mode is required on Windows, every run.** Several background
  services and protections are reduced in Safe Mode, which is also when
  stubborn bloatware/locked files are easiest to remove cleanly. The
  tradeoff is that some features (Wi-Fi/Bluetooth toggling, Appx removal,
  network-dependent driver scans) can behave inconsistently in Safe Mode
  since those services aren't loaded there — that's expected.
- **Security components are never removed.** Windows Defender, Security
  Center, and the Microsoft Store are explicitly excluded from the
  bloatware list in every tier — only cosmetic bloat (Xbox apps, Bing
  widgets, Solitaire, Copilot, Teams, etc.) is targeted.
- **Boot tweaks never touch signature enforcement.** The `bcdedit` /
  boot-menu changes applied are limited to cosmetic/UX settings (menu
  policy, timeout). None of the scripts enable test-signing or disable
  driver signature enforcement — that specific setting is how unsigned
  (including malicious) kernel drivers get loaded, so it's intentionally
  left alone.
- **Process trimming always asks first.** The "trim to essential
  processes" option force-closes open apps and can lose unsaved work, so
  every tier requires you to type `YES` to confirm before it runs. It also
  never touches a hard-coded protected list of core OS/session processes
  (`explorer.exe`, `lsass.exe`, `svchost.exe`, Defender, etc. on Windows;
  `systemd`, `sshd`, `NetworkManager`, `Xorg`/Wayland, etc. on Linux) no
  matter how aggressive the trim — the goal is "close everything
  non-essential," not "crash the session."
- **Pagefile/swap disabling is opt-in and flagged.** It's a legitimate
  tweak on machines with a lot of RAM, but risky on machines without much,
  so it's never part of any "run everything" batch option and is logged as
  a warning when used.

## Logging

Every script writes a timestamped log of everything it did:

- Windows: `%ProgramData%\SystemOptimizer\optimizer.log`
- Linux: `/var/log/system-optimizer/optimizer.log`

Driver reports are saved alongside the log as `drivers.txt`.

## FAQ / troubleshooting

**"It says NOT RUNNING IN SAFE MODE — ABORTING, but I'm sure I'm in Safe
Mode."**
Make sure you're in true Safe Mode (the desktop background says "Safe Mode"
in each corner, taskbar is black) and not just a minimal/clean boot via
`msconfig`'s Selective startup — that's a different thing and isn't
detected as Safe Mode by Windows itself.

**"UAC isn't prompting me / the script just closes."**
Right-click the file explicitly and choose **Run as administrator** instead
of double-clicking — some Group Policy configurations suppress the
self-elevation prompt on double-click.

**"Can I run just one feature instead of the whole menu?"**
Yes — `v2` and `v3` are both menu-driven; pick the single numbered option
you want and it runs on its own. `v1` is the only linear, all-in-one tier.

**"Why isn't bloatware removal in v1?"**
`v1` is intentionally the minimal tier — it covers the lowest-risk tweaks
only (temp cleanup, power plan, dark mode, Copilot, boot menu). Use `v2` or
`v3` for bloatware removal.

**"Does the Linux script need Safe Mode too?"**
No — Linux has no direct equivalent of Windows Safe Mode, so the script
doesn't try to fake-enforce one. If you want a similar safety net, boot
into a rescue/single-user target first: `sudo systemctl isolate
rescue.target`.

## License

MIT — see [LICENSE](LICENSE).
