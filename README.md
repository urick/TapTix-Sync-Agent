# Taptix Sync Agent

Automated sync service that keeps your POS data synchronized with Taptix Cloud. Runs as a Windows service, starts on boot, and updates itself automatically.

**Requirements:** Windows 10 or later (Windows 11, Server 2016+ also supported)

---

## Download

**[Download Latest Version](https://github.com/urick/TapTix-Sync-Agent/releases/latest)**

Download the `.zip` file and extract it.

---

## Installation

### Step 1: Copy Files

Extract the zip to a **local** folder:

```
C:\TaptixSyncAgent\
```

> **Important:** Do NOT run from a network/UNC path (`\\server\share\...`). Copy to a local drive first.

The zip contains:
- `Taptix.SyncAgent.exe` — the agent (single file)
- `appsettings.json` — configuration
- `install-service.bat` — service installer
- `uninstall-service.bat` — service uninstaller

### Step 2: Configure

Open `appsettings.json` and update these three settings:

```json
"DataSource": {
    "ConnectionString": "Server=YOUR_SQL_SERVER;Database=resto;User Id=sa;Password=YOUR_PASSWORD;TrustServerCertificate=True;",
    "Timezone": "Europe/Tirane"
},
"Agent": {
    "InstanceId": "YOUR-CLIENT-NAME"
}
```

| Setting | What to enter | Example |
|---|---|---|
| `Server=` | POS SQL Server IP or name | `192.168.1.100` or `.\SQLEXPRESS` |
| `Password=` | SQL Server password | `MyPassword123` |
| `Timezone` | POS machine timezone | `Europe/Tirane` |
| `InstanceId` | Unique name for this client | `CAFE-ROMA` or `HOTEL-BEACH` |

Everything else is pre-configured (API URL, application code, sync schedule, auto-update).

### Step 3: Setup Credentials

Open **PowerShell as Administrator**:

```powershell
cd C:\TaptixSyncAgent
.\Taptix.SyncAgent.exe setup <username> <password>
```

This will:
- Test login against Taptix API
- Encrypt and store credentials locally
- Register the device
- Check license validity
- Test POS database connection

### Step 4: Health Check

```powershell
.\Taptix.SyncAgent.exe check
```

All checks should show green. This also creates sync tracking tables in the POS database.

### Step 5: Install as Windows Service

```powershell
.\install-service.bat
```

> **Must run as Administrator.** If you see "This script must be run as Administrator", right-click PowerShell → Run as Administrator.

If `install-service.bat` fails due to a UNC path error, install manually:

```powershell
sc.exe create TaptixSyncAgent binPath= "C:\TaptixSyncAgent\Taptix.SyncAgent.exe" DisplayName= "Taptix Sync Agent" start= auto
sc.exe failure TaptixSyncAgent reset= 86400 actions= restart/60000/restart/60000/restart/60000
sc.exe start TaptixSyncAgent
```

### Step 6: Verify

```powershell
sc.exe query TaptixSyncAgent
```

Should show `STATE: RUNNING`. Done — the agent now syncs automatically and updates itself.

---

## CLI Commands

All commands require `.\` prefix in PowerShell.

| Command | Description |
|---|---|
| `.\Taptix.SyncAgent.exe` | Start agent (console mode) |
| `.\Taptix.SyncAgent.exe setup <user> <pass>` | Configure credentials and register device |
| `.\Taptix.SyncAgent.exe check` | Full health check (DB, API, auth, license) |
| `.\Taptix.SyncAgent.exe status` | Show config and validate authentication |
| `.\Taptix.SyncAgent.exe report` | Show sync statistics and failures |
| `.\Taptix.SyncAgent.exe update` | Check for and apply updates immediately |
| `.\Taptix.SyncAgent.exe update --check` | Check for updates without applying |
| `.\Taptix.SyncAgent.exe version` | Show agent version and runtime info |
| `.\Taptix.SyncAgent.exe device` | Show device registration info |
| `.\Taptix.SyncAgent.exe backfill <start> <end>` | Sync historical data for date range |
| `.\Taptix.SyncAgent.exe clear` | Delete all stored data (credentials, device) |
| `.\Taptix.SyncAgent.exe help` | Show all commands |

Add `--verbose` or `-v` to any command for debug-level logging.

---

## Auto-Update

The agent checks for new versions every hour and updates automatically. No action needed.

- Updates download from this repo's releases
- The exe is replaced; your `appsettings.json` and credentials are never touched
- The service restarts automatically after applying an update
- If an update fails 3 times, it rolls back to the previous version

**Force an update:**

```powershell
sc.exe stop TaptixSyncAgent
.\Taptix.SyncAgent.exe update
sc.exe start TaptixSyncAgent
```

**Disable auto-update** (if needed): add to `appsettings.json`:

```json
"AutoUpdate": {
    "Enabled": false
}
```

---

## Service Management

| Action | Command |
|---|---|
| Check status | `sc.exe query TaptixSyncAgent` |
| Stop | `sc.exe stop TaptixSyncAgent` |
| Start | `sc.exe start TaptixSyncAgent` |
| Restart | `sc.exe stop TaptixSyncAgent && sc.exe start TaptixSyncAgent` |
| Uninstall | `.\uninstall-service.bat` (as Admin) |
| View in GUI | `services.msc` → find "Taptix Sync Agent" |

The service starts automatically on boot and restarts on crash (after 60 seconds).

---

## Changing Configuration

The agent reads `appsettings.json` once at startup. Any changes require a service restart to take effect.

**To change settings:**

```powershell
sc.exe stop TaptixSyncAgent
notepad C:\TaptixSyncAgent\appsettings.json
sc.exe start TaptixSyncAgent
```

**Common changes:**

| Setting | When to change |
|---|---|
| `Server=` in ConnectionString | SQL Server IP or password changed |
| `Timezone` | POS machine moved to a different timezone |
| `InstanceId` | Renaming the client |
| `AutoUpdate:Enabled` | Disable auto-updates (set to `false`) |
| `AutoUpdate:CheckIntervalMinutes` | Change how often the agent checks for updates (default: 60) |

> **Tip:** After changing settings, verify the agent started correctly: `sc.exe query TaptixSyncAgent` (should show `RUNNING`). If it fails, check the logs.

**To change credentials (username/password):**

```powershell
sc.exe stop TaptixSyncAgent
.\Taptix.SyncAgent.exe setup <new-username> <new-password>
sc.exe start TaptixSyncAgent
```

---

## Viewing Logs

Logs are in the `logs\` folder inside the installation directory.

**View recent logs:**

```powershell
Get-Content -Path "C:\TaptixSyncAgent\logs\sync-agent-*.txt" -Tail 30
```

**Follow logs in real-time:**

```powershell
Get-Content -Path "C:\TaptixSyncAgent\logs\sync-agent-*.txt" -Tail 30 -Wait
```

Press `Ctrl+C` to stop following.

---

## Troubleshooting

### "This script must be run as Administrator"

Right-click PowerShell → **Run as Administrator**, then retry.

### "The term 'Taptix.SyncAgent.exe' is not recognized"

PowerShell requires `.\` prefix:

```powershell
.\Taptix.SyncAgent.exe check    # correct
Taptix.SyncAgent.exe check      # wrong
```

### Service fails to start (error 1053)

1. Check you're running from a **local path** (not `\\server\share\...`)
2. Try running directly to see the error: `.\Taptix.SyncAgent.exe`
3. Check logs: `Get-Content -Path "logs\sync-agent-*.txt" -Tail 50`

### "Login failed" during setup

1. Verify the username and password are correct
2. Check if the API is reachable: open `https://api.taptix.al` in a browser
3. Ask your administrator if the sync user account is active

### "SEATS_FULL" license error

All license seats are occupied. Contact your administrator to release a seat in LicenseHub, or wait — the cache expires in 5 minutes.

### "No credentials stored"

Run setup again:

```powershell
.\Taptix.SyncAgent.exe setup <username> <password>
```

### "Date in the future" validation errors

The `Timezone` setting in `appsettings.json` doesn't match the POS machine's timezone. Albania/Kosovo uses `Europe/Tirane`.

### No data syncing

```powershell
.\Taptix.SyncAgent.exe report
```

Check for failed records. If the agent just started, run a backfill for historical data:

```powershell
.\Taptix.SyncAgent.exe backfill 2025-01-01 2026-03-23
```

### Agent version shows "unknown"

You're running an old build. Download the latest version from the releases page.

---

## Common Timezones

| Region | Timezone setting |
|---|---|
| Albania / Kosovo | `Europe/Tirane` |
| Greece | `Europe/Athens` |
| Italy | `Europe/Rome` |
| UK | `Europe/London` |
| Germany | `Europe/Berlin` |

---

## Support

Open an issue on this repo for bug reports, or contact your Taptix administrator.
