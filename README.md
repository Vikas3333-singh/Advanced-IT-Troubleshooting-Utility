# FixForge

A two-script maintenance toolkit built for enterprise IT environments — one for Windows endpoints, one for macOS. Both deployed internally at **Femmella Fashions** for L1/L2 support operations.

---

## Scripts

| Script | Platform | Mode |
|---|---|---|
| `windows_maintenance_final.bat` | Windows 10/11 | Interactive menu + CLI args |
| `maintenance.sh` | macOS | Interactive menu + CLI flags |

---

## Windows — `windows_maintenance_final.bat`

### How It Works

The script opens with a **natural language prompt** — the user types their issue in plain English (or Hinglish). The script parses keywords and routes them to a context-specific menu showing only relevant options. If the issue is "slow performance", it shows performance tools. "Office crashes" shows repair tools. No IT knowledge required from the end user.

Supports **CLI argument mode** for scripted/automated deployment — pass an option directly and it runs non-interactively.

```cmd
REM Interactive (shows menu)
windows_maintenance_final.bat

REM Automated (runs one task, no prompts)
windows_maintenance_final.bat temp
windows_maintenance_final.bat sfc
windows_maintenance_final.bat dism
```

### CLI Arguments

| Argument | Action |
|---|---|
| `temp` | Delete `%TEMP%` and `C:\Windows\Temp` |
| `sfc` | Run System File Checker (`sfc /scannow`) |
| `dism` | Run `DISM /Online /Cleanup-Image /RestoreHealth` |
| `drivers` | Scan and update drivers via `pnputil` |
| `software` | Windows and app updates |
| `diskcleanup` | Silent disk cleanup via registry flags + `cleanmgr` |
| `services` | Disable 20+ non-essential services |
| `store` | Repair Microsoft Store |
| `office` | Repair Microsoft Office installation |
| `appdata` | Clear app caches (Slack, Chrome, Firefox, Discord, Teams) |
| `battery` | Battery health report with wear level and recommendation |
| `defrag` | Disk defragmentation (`C:` drive) |
| `eventlog` | Clear System, Application, and Security event logs |
| `power` | Set power plan to High Performance |
| `hardware` | Basic disk and memory hardware check |

### What `services` Disables

Stops and disables 20+ non-essential Windows services including:

`SysMain` · `DiagTrack` · `WSearch` · `XboxAccessoryManagementService` · `XboxLiveAuthManager` · `XboxLiveGameSave` · `XboxLiveNetAuthManager` · `MapsBroker` · `lfsvc` · `PushToInstall` · `GameInputSvc` · `RetailDemo` · `Fax` · `fhsvc` · `TrkWks` · `CscService` · `DiagSvc` · `TBS` · `vmcompute` · `vmicprovider`

### Smart Issue Routing

The main menu parses the user's typed issue and routes to a filtered menu:

| Keywords | Routed To |
|---|---|
| `slow`, `lag`, `hang`, `freeze`, `performance`, `speed` | Performance optimization menu |
| `crash`, `error`, `app`, `application` | Crash/error fixes menu |
| `disk`, `space`, `storage`, `full` | Disk cleanup menu |
| `battery`, `power` | Battery options |
| `update`, `software` | Update menu |
| `driver`, `device`, `hardware` | Driver/hardware menu |
| `store`, `microsoft` | Microsoft Store repair |
| `office`, `word`, `excel` | Office repair menu |
| `samjh nahi aata`, `understand`, `confused`, `full scan` | Shows all options |

### Requirements

- Windows 10 / 11
- Run as Administrator (auto-checks, exits with clear error if not)
- Optional: `maintenance_config.ini` in the same directory for custom settings

---

## macOS — `maintenance.sh`

### Enterprise-Grade Shell Script

Version 2.1 — written with `set -Eeuo pipefail` (strict error handling), atomic process locking, log rotation, NO_COLOR standard compliance, and a `--dry-run` mode that previews every action before execution.

### Usage

```bash
# Interactive menu
sudo ./maintenance.sh

# Full maintenance, no prompts
sudo ./maintenance.sh --full --auto

# Full maintenance, silent (log only)
sudo ./maintenance.sh --full --auto --quiet

# Preview what would run (no changes made)
sudo ./maintenance.sh --dry-run

# System report only (no root required)
./maintenance.sh --report

# Help
./maintenance.sh --help
```

### CLI Flags

| Flag | Effect |
|---|---|
| `--full` | Runs all 13 maintenance tasks sequentially |
| `--auto` | Skips all confirmation prompts |
| `--quiet` | Silent mode — writes to log only, no console output |
| `--dry-run` | Logs what would happen, makes zero changes |
| `--report` | Generates system report only; no root required |

### Menu Options (Interactive)

| # | Task | What It Does |
|---|---|---|
| 1 | Update Homebrew | `brew update && brew upgrade && brew cleanup` |
| 2 | Clean Caches | Clears user cache, skips excluded apps (Outlook, Slack, Google, Adobe) |
| 3 | Empty Trash | Empties trash for the invoking user |
| 4 | Clean Downloads | Removes files older than 30 days from Downloads |
| 5 | Clean Temp Files | Clears `/tmp` files older than 7 days |
| 6 | Clean Logs | Removes log files older than 30 days |
| 7 | Verify Disk | Runs `diskutil verifyVolume /` |
| 8 | Memory Check | Reports memory pressure |
| 9 | Battery Health | Parses `system_profiler` — Cycle Count, Condition, Max Capacity |
| 10 | Large File Scan | Finds files >1GB in home directory (excludes Library, Trash) |
| 11 | Create Backup | `rsync` backup of Documents with pre-check for available disk space |
| 12 | Generate Report | JSON system report saved to `/Library/Application Support/Femella/reports/` |
| 13 | Printer Status | `lpstat -p` printer queue check |
| 14 | Full Maintenance | Runs all above tasks with pass/fail summary |

### Full Maintenance Summary

When run with `--full`, the script tracks each task's pass/fail state and prints a summary at the end:

```
================================================
 Full Maintenance Summary
================================================
  ✓ Homebrew Update          OK
  ✓ Cache Cleanup            OK
  ✓ Battery Health           OK
  ✗ Create Backup            FAILED
  ...
================================================
```

### Safety Features

- **Atomic lock file** — prevents two instances running simultaneously; stores PID for diagnostics
- **Disk space pre-check** — verifies available space before running backup; aborts if insufficient
- **Excluded cache paths** — Outlook, Slack, Google, Adobe caches are never touched
- **Dry-run mode** — every destructive action goes through `safe_exec()` which logs instead of runs
- **Log rotation** — auto-rotates at 10MB, keeps last 5 rotated logs
- **Trap on EXIT** — lock file is always cleaned up even on crash or SIGINT
- **NO_COLOR compliance** — disables ANSI colors when piped or when `NO_COLOR` env var is set

### Logging

| Path | Purpose |
|---|---|
| `/Library/Logs/Femella/mac_maintenance.log` | Rolling operational log (10MB max per file) |
| `/Library/Application Support/Femella/reports/` | JSON system reports (retained 90 days) |
| `/Library/Application Support/Femella/backups/` | Document backups |

Logs also write to macOS system log via `logger -t FemellaMaintenance` for syslog/Console.app visibility.

### Requirements

- macOS (tested on macOS Ventura and above)
- `sudo` / root for most operations
- Homebrew (optional — update task skips gracefully if not installed)

---

## Repo Structure

```
remote-support-toolkit/
├── windows_maintenance_final.bat   # Windows all-in-one maintenance script
├── maintenance.sh                  # macOS enterprise maintenance script
├── maintenance_config.ini          # Optional config for Windows script
└── README.md
```

---

## Roadmap

- [ ] PowerShell rewrite of Windows script for richer output and remote execution
- [ ] GUI wrapper for non-technical users
- [ ] Intune / SCCM deployment packaging (silent mode already supported)
- [ ] Centralised logging to shared network path or SIEM
- [ ] Linux support (`maintenance_linux.sh`)

---

## Author

**Vikas Singh** — IT Systems Specialist · Femmella Fashions India Limited  
`Endpoint Management` `EUC` `L1/L2 Support` `PowerShell` `Bash`
