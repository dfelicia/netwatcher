# README
***

## netwatcher
[id]: don@effinthing.com "Don Feliciano"
Author: [Don Feliciano][id]<br />

### Overview
A macOS daemon that performs actions based on your network location (work/non-work).

### Requirements

- macOS 10.14 or later
- Bash 4.0 or later
- GNU sed (`gsed`)
- `jq`
- `curl`
- `terminal-notifier` (optional, for toaster notifications)

### Features

- Automatically detects work vs non-work location based on:
  - Domain suffix pattern
  - Network SSID (if on Wi-Fi)

- Performs the following actions when location changes:
  - Sets default printer
  - Updates NTP server
  - Toggles notification center alerts
  - Manages background services and apps

### Installation

1. Install all dependencies via Homebrew (including optional `terminal-notifier`):
   ```bash
   brew install bash jq curl gnu-sed terminal-notifier
   ```
2. Run the setup script:
   ```bash
   ./setup
   ```

### Configuration

Setup will prompt for:
  1. Work printer name (or "none")
  2. Default printer name for non-work networks (or "none")
  3. Whether or not to enable Notification Center alerts
  4. Domain suffix pattern for work networks
  5. SSID for work networks
  6. Work NTP server
  7. Default NTP server for non-work networks (time.apple.com)
  8. Proxy server host:port for work networks (or leave blank)
  9. Extra DNS search domains for work networks (or leave blank)
  10. Automatically configure passwordless sudo access for required commands

**Proxy server (item 8):**
The specified proxy host:port will be written to `~/.curlrc` when switching to work mode and removed when switching to non-work mode. If a WPAD file (`wpad.dat`) is available and contains a proxy URL, that proxy takes priority over your manual setting.

**Extra DNS search domains (item 9):**
These domains are ones that your work network does not automatically push into `/var/run/resolv.conf`. They will be appended to your DNS search list when on work mode and ignored otherwise.

Configuration is stored in `~/.netwatcher/config`; you can re-run setup anytime with `./setup -r`.

### Options

- `-u` Uninstall
- `-r` Reinstall (re-run setup)
- `-t secs` Throttle interval between network changes
- `-p secs` Pause interval before evaluating a change

### How It Works

netwatcher runs as a LaunchAgent and watches `/var/run/resolv.conf` for changes. When a change is detected, it:

1. Waits for the configured throttle (`-t`) and pause (`-p`) intervals
2. Determines the active network interface and its IP address
3. Reads the DNS search domains from `/var/run/resolv.conf`
4. Classifies the location as “work” or “non-work” based on:
   - Matching any search domain against the configured work domain pattern
   - Matching the current SSID against the configured work SSID (if on Wi-Fi)
5. Executes the configured actions for the detected location:
   - Enables or disables proxy settings (WPAD) and updates `~/.curlrc`
   - Sets the NTP server and syncs the time
   - Sets the default printer
   - Quits VPN client when switching to non-work
   - Sends a notification summarizing the change

### Troubleshooting

- Check `~/Library/Logs/netwatcher.log` for errors
- Use `sudo touch /var/run/resolv.conf` to trigger a check
- Run `defaults read ~/Library/LaunchAgents/local.netwatcher.plist` to verify settings
- Ensure `/etc/sudoers.d/<your_user>` exists with the required `NOPASSWD` entries (setup can create it)
- To enable verbose debug logging, add `DEBUG="true"` to `~/.netwatcher/config`