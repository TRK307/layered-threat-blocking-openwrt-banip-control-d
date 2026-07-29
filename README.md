# layered-threat-blocking-openwrt-banip-control-d

IP-reputation blocking at the firewall (banIP) plus Encrypted DNS filtering (Control D), running together on an OpenWrt Router, fw4/nftables.

## Hey 👋

This is a look at something I set up on my home network — not a polished product, just me documenting what I did in case it's useful to someone else poking around their own OpenWrt router. Feel free to have a look, borrow whatever's useful, or just enjoy the read.

The short version: DNS filtering catches bad stuff before a connection is even attempted, and firewall-level IP blocking catches whatever slips past that. Running both together covers more ground than either one alone.

Related bits from the same homelab, if you're curious:
- 🔗 SQM/CAKE Bufferbloat Fix — *(link here)*
- 🔗 Real-Time Network Monitoring Dashboard — *(link here)*

## What's Actually Happening Here

```
                     Client devices (phone / laptop / etc.)
                              │
              ┌───────────────┴───────────────┐
              │                               │
    Router-wide encrypted DNS       Per-device Control D app
    (OpenWrt Router → Control D)   (works on/off home network)
              │                               │
              └───────────────┬───────────────┘
                              ▼
                Control D — encrypted DNS + filtering
                (bad domains stopped before connection)
                              │
                              ▼
                banIP @ WAN-Input (table inet fw4)
                (bad IPs stopped at the firewall)
                              │
                              ▼
                        LAN devices
```

## The Wins

✅ DNS queries used to be plaintext — anything watching the network could see every domain resolved. Fixed with DNS over TLS (DoT) via Control D, both on the router and on individual devices.

✅ Router-only filtering stopped working the second a device left the house. Fixed by giving key devices their own Control D setup, so filtering follows them wherever they connect.

✅ DNS filtering alone still misses anything that skips DNS and hits an IP directly. That's exactly what banIP covers, using reputation feeds at the firewall (`dshieldv4`, `dropv4`, `blocklistv4`, `allowlistv4`).

✅ Used to have no idea if "weird" traffic numbers were actually weird. Now there's a baseline (~0.4% invalid-conntrack traffic) to compare against later, which matters once a dual-WAN setup enters the picture.

Honestly, the DNS side was the more satisfying win — Control D's dashboard shows blocked vs. bypassed traffic in real time, so the payoff is immediate. The firewall side is quieter but catches a different kind of problem entirely, which is why both are worth running side by side.

## Guides

- [`control-d/`](./control-d/README.md) — encrypted DNS setup, router + endpoint devices
- [`banip/`](./banip/README.md) — IP-reputation blocking setup on the firewall

## A Note on My Setup

Screenshots in here come from a GL.iNet travel router, which happens to have Control D built in as a selectable DNS provider — no manual config needed on that side, just a dropdown. Everything else (banIP, fw4/nftables) is standard OpenWrt and applies regardless of which router you're running. Where the GL.iNet UI differs from stock OpenWrt/LuCI, I've called it out.

## Nothing Sensitive In Here

Screenshots and examples use generic device names (`phone-1`, `laptop-1`, etc.) and placeholder IDs — no real device names, resolver IDs, IPs, or keys from my actual setup.

## What's Next

- [ ] Cross-reference Control D block logs with banIP block logs for one combined view
- [ ] Add Suricata into the mix and see how it plays with the existing setup
- [ ] Re-check the invalid-conntrack baseline once a dual-WAN setup is in place
