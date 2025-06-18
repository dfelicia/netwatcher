# README
***

## netwatcher
[id]: don@effinthing.com "Don Feliciano"
Author: [Don Feliciano][id]<br />

### Overview
A macOS daemon that performs actions based on your network location (work/home).

### Requirements

- macOS 10.14 or later
- Bash 4.0 or later (installed via Homebrew)
- GNU sed (`gsed`)
- `jq`
- `curl`
- `terminal-notifier` (optional, for notifications)

### Features

- Automatically detects work vs home location based on:
  - Network SSID
  - IP address pattern
  - Domain suffix pattern

- Performs the following actions when location changes:
  - Sets default printer
  - Updates NTP server
  - Toggles notification center alerts
  - Manages background services and apps

### Installation

1. Install dependencies via Homebrew:
   ```bash
   brew install bash jq curl gnu-sed
   ```
2. (Optional) Install `terminal-notifier` for macOS notifications:
   ```bash
   brew install terminal-notifier
   ```
3. Run the setup script:
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

Configuration is stored in `~/.netwatcher/config`; you can re-run setup anytime with `./setup -r`.

### Options

- `-u` Uninstall
- `-r` Reinstall (re-run setup)
- `-t secs` Throttle interval between network changes
- `-p secs` Pause interval before evaluating a change

### How It Works

Netwatcher monitors network changes by watching:
- /var/run/resolv.conf
- Network interface status
- SSID changes
- Domain suffix changes

When changes occur, it evaluates the current network state against your configured work/home patterns to determine your location and performs the appropriate actions.

- Watches /var/run/resolv.conf for changes to DNS search domains
- Debounces rapid events using the -t throttle interval and allows interfaces to settle with -p
- Prioritizes VPN interfaces (utun*)
- Then wired links (non-Wi-Fi), preferring port names containing "Ethernet"
- Falls back to Wi-Fi ports
- Finally uses the system default route as a last resort
- Verifies the interface is active by checking for an IPv4 or global-scope IPv6 address
- Uses the get_ip_address() helper
- Attempts ipconfig getifaddr <iface>
- Falls back to parsing ifconfig <iface> output with awk
- Reads DNS search domains from /var/run/resolv.conf
- Compares against the configured WORK_DOMAIN_PATTERN to decide "work" vs "home"
- In work mode:
  - Enables automatic proxy (WPAD) and writes proxy = … to ~/.curlrc
  - Sets NTP server to WORK_NTP_SERVER
  - Configures default printer to WORK_PRINTER
  - Applies any VPN DNS overrides and manages background services
- In home mode:
  - Disables the proxy and removes WPAD entries from ~/.curlrc
  - Resets NTP server to HOME_NTP_SERVER
  - Sets default printer to HOME_PRINTER
  - Quits known VPN clients
- Sends a macOS notification (via terminal-notifier) summarizing
  - Active interface and local IP address
  - Detected location mode (work/home)
  - ISP, region, and city (if available)

### Troubleshooting

- Check `~/Library/Logs/netwatcher.log` for errors
- Use `sudo touch /var/run/resolv.conf` to trigger a check
- Run `defaults read ~/Library/LaunchAgents/local.netwatcher.plist` to verify settings
- Ensure `/etc/sudoers.d/<your_user>` exists with the required `NOPASSWD` entries (setup can create it)