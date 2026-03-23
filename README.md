# Taptix Sync Agent

Automated sync service for Taptix POS Integration. Downloads and installs on Windows machines to sync POS data to Taptix Cloud.

## Download

Get the latest release: [Latest Release](https://github.com/urick/TapTix-Sync-Agent/releases/latest)

## Installation

1. Download and extract the zip
2. Edit `appsettings.json` with your database connection and instance ID
3. Run `Taptix.SyncAgent.exe setup <username> <password>`
4. Run `Taptix.SyncAgent.exe check`
5. Run `install-service.bat` as Administrator

The agent auto-updates after initial install.

## Support

Open an issue on this repo for bug reports.
