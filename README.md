# localhost-mobilizer

Simple macOS commands for testing a Laravel Herd local site on a real phone.

`mobile-mode` switches Herd into device-testing mode.
`local-mode` restores your original Herd config.

## macOS Only

This is built for:

- macOS
- Laravel Herd
- Herd's default `dnsmasq` and `nginx` config files

## Install

```bash
git clone git@github.com:glennraya/localhost-mobilizer.git
cd localhost-mobilizer

mkdir -p ~/.local/bin
cp mobile-mode local-mode .mobile-mode-common ~/.local/bin/
chmod +x ~/.local/bin/mobile-mode ~/.local/bin/local-mode ~/.local/bin/.mobile-mode-common
```

If needed, add `~/.local/bin` to your shell path:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Use

1. Enable mobile mode:

```bash
mobile-mode
```

2. Start your frontend so it listens on the network:

```bash
npm run dev -- --host
```

3. On your phone:

- connect to the same Wi-Fi as your Mac
- set the phone DNS to the IP shown by `mobile-mode`
- open your site over `http`, not `https`

4. Restore normal local setup when finished:

```bash
local-mode
```

## What It Changes

`mobile-mode` updates Herd so:

- `.test` domains resolve to your Mac's local network IP
- Herd nginx listens on all interfaces

`local-mode` restores the exact original config from:

```text
~/.local/state/mobile-mode
```

## If It Picks The Wrong IP

Force the correct interface:

```bash
MOBILE_MODE_INTERFACE=en0 mobile-mode
```

Or force the IP directly (replace <your-ip-address> with your IP address:

```bash
MOBILE_MODE_LAN_IP=<your-ip-address> mobile-mode
```

## Notes

- "LAN IP" here means your private local network IP, usually your Wi-Fi IP
- this does not expose your Mac to the public internet by itself
- avoid using it on public or untrusted Wi-Fi
