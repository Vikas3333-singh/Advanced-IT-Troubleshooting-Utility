# Advanced IT Troubleshooting Utility (Batch Scripting)

> A two-script enterprise maintenance toolkit for Windows and macOS — built and deployed for L1/L2 IT support operations at Femmella Fashions India Limited.

---

## Problem

L1/L2 support engineers were running the same maintenance tasks manually on every endpoint — temp cleanup, disk repair, driver updates, Office repair — one by one, every time. No consistency, no speed, no way to automate.

---

## Solution

FixForge is a two-script toolkit that covers the full maintenance lifecycle for both platforms:

- **Windows** — a smart batch script with natural language issue routing and full CLI argument support
- **macOS** — an enterprise-grade shell script with dry-run mode, atomic locking, log rotation, and a pass/fail summary

Both scripts work interactively for end users and non-interactively for automated/scripted deployment.

---

## Scripts

| Script | Platform | Mode |
|--------|----------|------|
| `windows_maintenance_final.bat` | Windows 10/11 | Interactive menu + CLI args |
| `maintenance.sh` | macOS (Ventura+) | Interactive menu + CLI flags |

---

## Windows — `windows_maintenance_final.bat`

### Smart Issue Routing

The script opens with a natural language prompt — the user types their issue in plain English. The script parses keywords and shows only the relevant options. No IT knowledge required from the end user.

| Keywords typed | Routed to |
|----------------|-----------|
| `slow`, `lag`, `hang`, `freeze` | Performance optimization menu |
| `crash`, `error`, `app` | Crash/error fixes menu |
| `disk`, `space`, `storage`, `full` | Disk cleanup menu |
| `battery`, `power` | Battery options |
| `update`, `software` | Update menu |
| `driver`, `device`, `hardware` | Driver/hardware menu |
| `store`, `microsoft` | Microsoft Store repair |
| `office`, `word`, `excel` | Office repair menu |
| `confused`, `full scan` | Shows all options |

### CLI Arguments (Automated / Non-Interactive)

```cmd
REM Interactive
windows_maintenance_final.bat

REM Automated — runs one task silently
windows_maintenance_final.bat temp
windows_maintenance_final.bat sfc
windows_maintenance_final.bat dism
```

| Argument | Action |
|----------|--------|
| `temp` | Delete `%TEMP%` and `C:\Windows\Temp` |
| `sfc` | Run System File Checker (`sfc /scannow`) |
| `dism` | Run `DISM /Online /Cleanup-Image /RestoreHealth` |
| `drivers` | Scan and update drivers via `pnputil` |
| `software` | Windows and app updates |
| `diskcleanup` | Silent disk cleanup via registry flags + `cleanmgr` |
| `services` | Disable 20+ non-essential services |
| `store` | Repair Microsoft Store |
| `office` | Repair Microsoft Office installation |
| `appdata` | Clear app caches (Slack, Chrome, Firefox, Teams, Discord) |
| `battery` | Battery health report with wear level and recommendation |
| `defrag` | Disk defragmentation (C: drive) |
| `eventlog` | Clear System, Application, and Security event logs |
| `power` | Set power plan to High Performance |
| `hardware` | Basic disk and memory hardware check |

### Services Disabled (`services` argument)

Stops and disables 20+ non-essential Windows services:

`SysMain` · `DiagTrack` · `WSearch` · `XboxAccessoryManagementService` · `XboxLiveAuthManager` · `XboxLiveGameSave` · `XboxLiveNetAuthManager` · `MapsBroker` · `lfsvc` · `PushToInstall` · `GameInputSvc` · `RetailDemo` · `Fax` · `fhsvc` · `TrkWks` · `CscService` · `DiagSvc` · `TBS` · `vmcompute` · `vmicprovider`

### Requirements

- Windows 10 / 11
- Run as Administrator (auto-checks on launch, exits with clear error if not)
- Optional: `maintenance_config.ini` in the same directory for custom settings

---

## macOS — `maintenance.sh`

Version 2.1 — written with `set -Eeuo pipefail` (strict error handling), atomic process locking, log rotation, NO_COLOR compliance, and a `--dry-run` mode that previews every action before execution.

### Usage

```bash
# Interactive menu
sudo ./maintenance.sh

# Full maintenance, no prompts
sudo ./maintenance.sh --full --auto

# Full maintenance, silent (log only)
sudo ./maintenance.sh --full --auto --quiet

# Preview what would run — zero changes made
sudo ./maintenance.sh --dry-run

# System report only (no root required)
./maintenance.sh --report

# Help
./maintenance.sh --help
```

### CLI Flags

| Flag | Effect |
|------|--------|
| `--full` | Runs all 13 maintenance tasks sequentially |
| `--auto` | Skips all confirmation prompts |
| `--quiet` | Silent mode — writes to log only, no console output |
| `--dry-run` | Logs what would happen, makes zero changes |
| `--report` | System report only; no root required |

### Menu Options

| # | Task | What It Does |
|---|------|--------------|
| 1 | Update Homebrew | `brew update && brew upgrade && brew cleanup` |
| 2 | Clean Caches | Clears user cache; skips Outlook, Slack, Google, Adobe |
| 3 | Empty Trash | Empties trash for the invoking user |
| 4 | Clean Downloads | Removes files older than 30 days from Downloads |
| 5 | Clean Temp Files | Clears `/tmp` files older than 7 days |
| 6 | Clean Logs | Removes log files older than 30 days |
| 7 | Verify Disk | Runs `diskutil verifyVolume /` |
| 8 | Memory Check | Reports memory pressure |
| 9 | Battery Health | Parses `system_profiler` — Cycle Count, Condition, Max Capacity |
| 10 | Large File Scan | Finds files >1GB in home directory (excludes Library, Trash) |
| 11 | Create Backup | `rsync` backup of Documents with pre-check for available disk space |
| 12 | Generate Report | JSON system report → `/Library/Application Support/Femella/reports/` |
| 13 | Printer Status | `lpstat -p` printer queue check |
| 14 | Full Maintenance | Runs all above tasks with pass/fail summary |

### Full Maintenance Summary Output

```
================================================
 Full Maintenance Summary
================================================
  ✓ Homebrew Update          OK
  ✓ Cache Cleanup            OK
  ✓ Battery Health           OK
  ✗ Create Backup            FAILED
================================================
```

### Safety Features

| Feature | Detail |
|---------|--------|
| Atomic lock file | Prevents two instances running simultaneously; stores PID for diagnostics |
| Disk space pre-check | Verifies available space before backup; aborts if insufficient |
| Excluded cache paths | Outlook, Slack, Google, Adobe caches are never touched |
| Dry-run mode | Every destructive action goes through `safe_exec()` — logs instead of runs |
| Log rotation | Auto-rotates at 10MB; keeps last 5 rotated logs |
| Trap on EXIT | Lock file always cleaned up, even on crash or SIGINT |
| NO_COLOR compliance | Disables ANSI colors when piped or when `NO_COLOR` env var is set |

### Logging

| Path | Purpose |
|------|---------|
| `/Library/Logs/Femella/mac_maintenance.log` | Rolling operational log (10MB max per file) |
| `/Library/Application Support/Femella/reports/` | JSON system reports (retained 90 days) |
| `/Library/Application Support/Femella/backups/` | Document backups |

Also writes to macOS system log via `logger -t FemellaMaintenance` for Console.app visibility.

### Requirements

- macOS Ventura or above
- `sudo` / root for most operations
- Homebrew (optional — update task skips gracefully if not installed)

---

## Repo Structure

```
fixforge/
├── windows_maintenance_final.bat   # Windows all-in-one maintenance script
├── maintenance.sh                  # macOS enterprise maintenance script
├── maintenance_config.ini          # Optional config for Windows script
└── README.md
```

---

## Roadmap

- [ ] PowerShell rewrite of Windows script for richer output and remote execution
- [ ] GUI wrapper for non-technical end users
- [ ] Intune / SCCM deployment packaging (silent mode already supported)
- [ ] Centralised logging to shared network path or SIEM
- [ ] Linux support (`maintenance_linux.sh`)

---

## Impact

- Standardised endpoint maintenance across Windows and macOS fleet at Femmella Fashions
- L1/L2 engineers run full maintenance in one command instead of manual step-by-step
- CLI argument support enables silent deployment via Intune/SCCM with zero user interaction
- Dry-run mode lets engineers preview impact before touching production endpoints

---

## Author

**Vikas Singh**
IT Systems Specialist | Endpoint Management | EUC | L1/L2 Support
[LinkedIn](#) · [GitHub](#)

---

## License

MIT
