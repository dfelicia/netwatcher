# README
***

## netwatcher
[id]: don@effinthing.com "Don Feliciano"
Author: [Don Feliciano][id]<br />

### Overview
A macOS daemon that performs actions based on your network location (work/home).

### Requirements

- macOS 10.13 or later
- Bash 4.0 or later (typically installed via Homebrew)
- terminal-notifier (optional, for notifications)

### Features

- Automatically detects work vs home location based on:
  - Network SSID
  - IP address pattern
  - Domain suffix pattern
  - Ethernet connection (optional)

- Performs the following actions when location changes:
  - Sets default printer
  - Updates NTP server
  - Toggles notification center alerts
  - Manages background services and apps

### Installation

1. Install bash 4+ if not already installed:
   ```bash
   brew install bash
   ```

2. Run the setup script:
   ```bash
   ./setup
   ```

### Configuration

Setup will prompt for:
- Work and home SSIDs
- Work IP address pattern (e.g., "10.[0-9][0-9][0-9]")
- Work and home domain patterns
- Work and home printers
- Work and home NTP servers
- Various optional features

Configuration is stored in `~/.netwatcher/config`

### Options

Setup supports the following flags:
- `-u` Uninstall
- `-r` Reinstall (re-run setup)
- `-t secs` Throttle time between network changes
- `-p secs` Pause time to allow interfaces to settle

### How It Works

Netwatcher monitors network changes by watching:
- /var/run/resolv.conf
- Network interface status
- SSID changes
- Domain suffix changes

When changes occur, it evaluates the current network state against your configured work/home patterns to determine your location and performs the appropriate actions.

### Troubleshooting

- Check `~/Library/Logs/netwatcher.log` for errors
- Use `sudo touch /var/run/resolv.conf` to trigger a check
- Run `defaults read ~/Library/LaunchAgents/local.netwatcher.plist` to verify settings