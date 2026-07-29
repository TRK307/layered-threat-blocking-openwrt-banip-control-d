# Control D — Encrypted DNS Filtering

Setting up DNS over TLS (DoT) with [Control D](https://controld.com/) on an OpenWrt Router, plus a couple of endpoint devices so filtering follows them off the home network.

## Why Bother

Regular DNS is plaintext — anyone watching the network path can see every domain your devices resolve, even if the actual web traffic itself is encrypted. Moving DNS to DoT closes that gap, and using Control D on top means you also get category/threat filtering at the resolver, before a connection is even attempted.

## Router Setup (Generic OpenWrt via CLI)

If your OpenWrt router doesn't have a built-in "pick a DNS provider" option, the usual route is a small DoT/DoH forwarding package:

```sh
opkg update
opkg install https-dns-proxy
```

Then:
1. Grab your DoT connection details and resolver ID from your Control D dashboard's router setup page.
2. Configure `https-dns-proxy` (via LuCI → Services, or by editing `/etc/config/https-dns-proxy` directly) to point at Control D using those details.
3. Point dnsmasq at the proxy's local listening port so LAN DNS queries actually go through it.
4. Restart the service:

```sh
service https-dns-proxy restart
```

5. Confirm it's working — from the router:

```sh
nslookup example.com 127.0.0.1
```

> **My own setup:** I'm running a GL.iNet travel router, which already has Control D built in as a selectable option under `Network → DNS` — no manual proxy package needed. I just chose "Encrypted DNS" → "DNS over TLS" → "Control D" from the dropdown and pasted in my resolver ID. If your router has something similar, it'll save you the CLI steps above.

## Endpoint Devices (Brief, Non-Technical)

A few of my devices run their own Control D setup directly, so filtering keeps working even when they're off the home network — mobile data, coffee shop Wi-Fi, wherever.

- **Windows:** installed the Control D desktop app, signed in, picked a profile.
- **Android:** installed the Control D app from the Play Store, granted the permission it asks for (it uses a local VPN-style connection to enforce DNS), picked a profile.
- **iOS:** installed the Control D app from the App Store (there's also a manual DNS profile option in Settings if you'd rather skip the app), picked a profile.

Nothing more technical than that — the app handles the rest.

## Keeping an Eye on It

The Control D dashboard (**Statistics → All Profiles**) shows:
- **Blocked vs. Bypassed** — how much traffic is actually being filtered
- **Encrypted DNS %** — should read 100% once everything's routed through DoT
- **Home Country Traffic %** — a rough anomaly check; a sudden dip can mean a device is resolving somewhere unexpected
- **Endpoints list** — per-device breakdown, including devices connecting from outside the home network
