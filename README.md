# RootOptimizer

A Windows/Linux system debloat & performance tool, shipped as four independent
scripts instead of one bundled program — pick the tier that fits.

| File | Platform | Tier | Notes |
|---|---|---|---|
| `optimizer_v1.bat` | Windows | v1 — basic | Fixed sequence, no menu. Safest, smallest feature set. |
| `optimizer_v2.cmd` | Windows | v2 — mid | Menu-driven CMD, shells out to PowerShell only where CMD has no equivalent. |
| `optimizer_v3.ps1` | Windows | v3 — full | Most capable tier: full menu, process trimming, driver reports, live monitor. |
| `optimizer_linux.sh` | Linux | — | Closest equivalent feature set, run with `sudo`. |

## ⚠️ Before you run anything

- **All three Windows scripts require Safe Mode.** They check this on launch
  and exit immediately if you're not in it (`msconfig.exe` → Boot tab →
  Safe boot, then reboot). Several features (Wi-Fi/Bluetooth toggling,
  bloatware removal, network-dependent driver scans) won't work correctly in
  Safe Mode, since those services aren't loaded there — that's a tradeoff of
  the Safe Mode requirement, not a bug.
- **All scripts require admin/root.** The Windows ones self-elevate via UAC;
  the Linux one must be run with `sudo`.
- **The process-trim option force-closes apps and can lose unsaved work.**
  It only ever asks for confirmation, never runs silently, and never touches
  a hard-coded protected list of core OS/session processes
  (`explorer.exe`, `lsass.exe`, `svchost.exe`, Defender, etc. on Windows;
  `systemd`, `sshd`, `NetworkManager`, etc. on Linux). It does **not** reduce
  the system to a fixed process count — Windows/Linux can't run on that few
  processes; it minimizes to "essential only" instead.
- **Bloatware removal intentionally excludes security components.** Windows
  Defender, Security Center, and the Store are never touched — only cosmetic
  bloat (Xbox apps, Bing widgets, Solitaire, Copilot, etc.).
- **The boot tweaks never disable driver signature enforcement.** Only safe,
  non-security settings (boot menu policy, timeout) are applied.

## Usage

**Windows (any tier):** right-click → Run as administrator, or double-click
and accept the UAC prompt (the script elevates itself).

```
optimizer_v1.bat
optimizer_v2.cmd
optimizer_v3.ps1   (if blocked by execution policy, run from an elevated
                     PowerShell prompt: powershell -ExecutionPolicy Bypass -File optimizer_v3.ps1)
```

**Linux:**

```bash
chmod +x optimizer_linux.sh
sudo ./optimizer_linux.sh
```

## What each tier can do

- Ultimate Performance power plan / CPU governor set to performance
- Full temp & cache cleanup
- Bloatware removal (v2/v3 only — keeps security apps)
- Disable Windows Copilot
- Dark mode
- Safe boot menu tweaks (v2/v3 only)
- Wi-Fi / Bluetooth adapter toggle
- Live process monitor window
- Driver/hardware report + update scan
- Disable pagefile / swap (advanced, off by default, flagged as risky)
- Trim to essential processes only (advanced, asks for confirmation)

Logs are written to `%ProgramData%\SystemOptimizer\optimizer.log` on Windows
and `/var/log/system-optimizer/optimizer.log` on Linux.

## License

MIT — see [LICENSE](LICENSE).
