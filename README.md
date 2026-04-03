# localhost-mobilizer

Turn a Laravel Herd local site into something your phone can reach on the same network, then switch everything back with one command.

`mobile-mode` updates Herd's `dnsmasq` and `nginx` config to listen on your Mac's LAN IP. `local-mode` restores the exact original config from backup.

## What It Does

- Detects your Mac's LAN IPv4 address
- Prefers Wi-Fi when multiple private interfaces exist
- Backs up your current Herd config before making changes
- Updates Herd `dnsmasq` so `.test` domains resolve to your Mac on the network
- Updates Herd `nginx` so it listens on all interfaces
- Restarts the required Herd services
- Restores the original config with `local-mode`

## Requirements

- macOS
- [Laravel Herd](https://herd.laravel.com/)
- Herd using the default config locations:
  - `~/Library/Application Support/Herd/config/dnsmasq/dnsmasq.conf`
  - `~/Library/Application Support/Herd/config/nginx/herd.conf`
- `~/.local/bin` on your `PATH`
- `sudo` access to restart `dnsmasq`

## Installation

Clone the repo somewhere permanent:

```bash
git clone git@github.com:glennraya/localhost-mobilizer.git
cd localhost-mobilizer
```

Install the commands into `~/.local/bin`:

```bash
mkdir -p ~/.local/bin
cp mobile-mode local-mode .mobile-mode-common ~/.local/bin/
chmod +x ~/.local/bin/mobile-mode ~/.local/bin/local-mode ~/.local/bin/.mobile-mode-common
```

Confirm they are available:

```bash
command -v mobile-mode
command -v local-mode
```

If `~/.local/bin` is not on your `PATH`, add this to your shell profile:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

Enable mobile testing mode:

```bash
mobile-mode
```

Start your frontend so it binds to the network:

```bash
npm run dev -- --host
```

Then on your phone:

1. Make sure your phone and Mac are on the same Wi-Fi network.
2. Set your phone's DNS server to the IP shown by `mobile-mode`.
3. Open your Herd site over `http`, not `https`.

When you are done:

```bash
local-mode
```

## Example Flow

```bash
mobile-mode
# Password:
# ...
# LAN IP: 192.168.68.56 (en0)

npm run dev -- --host

# Test on phone, then restore:
local-mode
```

## What Gets Changed

`mobile-mode` updates these Herd values:

`dnsmasq.conf`

```conf
address=/.test/<LAN_IP>
listen-address=0.0.0.0
```

`herd.conf`

```nginx
listen 0.0.0.0:80 default_server;
```

`local-mode` restores the exact original files from:

```text
~/.local/state/mobile-mode
```

## Safety Behavior

- The original Herd config is backed up only once per mobile session.
- Re-running `mobile-mode` refreshes the detected IP without overwriting the original backup.
- `local-mode` restores the exact backed-up files, then removes the saved state.
- The scripts refuse to patch config files if the expected Herd lines are missing or ambiguous.

## Environment Overrides

You usually do not need these, but they are useful for edge cases and testing.

| Variable | Purpose |
| --- | --- |
| `MOBILE_MODE_LAN_IP` | Force a specific IPv4 address instead of auto-detecting |
| `MOBILE_MODE_INTERFACE` | Force a specific interface, like `en0` |
| `MOBILE_MODE_HERD_HOME` | Override the Herd config root |
| `MOBILE_MODE_STATE_DIR` | Override the backup directory |
| `MOBILE_MODE_SKIP_DNSMASQ_KILL=1` | Skip `sudo killall dnsmasq` |
| `MOBILE_MODE_SKIP_RESTART=1` | Skip `herd restart` |

Examples:

```bash
MOBILE_MODE_INTERFACE=en0 mobile-mode
MOBILE_MODE_LAN_IP=192.168.68.56 mobile-mode
```

## Troubleshooting

### It picked the wrong IP

If your Mac has multiple private interfaces, force the right one:

```bash
MOBILE_MODE_INTERFACE=en0 mobile-mode
```

Or force the address directly:

```bash
MOBILE_MODE_LAN_IP=192.168.68.56 mobile-mode
```

### My phone cannot load the site

- Confirm the phone and Mac are on the same network
- Confirm the phone DNS is set to the IP printed by `mobile-mode`
- Use `http`, not `https`
- Make sure your app dev server is running with `--host`
- Check macOS firewall or VPN software if traffic is still blocked

### `local-mode` says no backup was found

You need to run `mobile-mode` first so it can snapshot the original Herd config.

## Notes

- This tool is designed for Laravel Herd's default `dnsmasq` and `nginx` config layout.
- It does not change phone DNS settings for you.
- It does not start your frontend dev server for you.
- It currently targets `.test` domains served by Herd.

## License

MIT
