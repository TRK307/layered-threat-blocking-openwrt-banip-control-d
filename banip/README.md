# banIP (IP-Reputation Blocking at the Firewall)

Blocking traffic to/from known-bad IPs on an OpenWrt Router, using [banIP](https://github.com/openwrt/packages/tree/master/net/banip) and fw4/nftables.

## Why Bother

Control D handles DNS-layer filtering, but that only helps if a connection actually starts with a DNS lookup. Anything that skips DNS and goes straight for an IP sails right past that. banIP catches that gap by checking every packet at the firewall against known-bad IP lists, regardless of how the connection got started.

## A Quick Note on fw4/nftables

OpenWrt's firewall4 (fw4) manages rules in `table inet fw4`, using the `nft` command. There's also an older `table ip filter`, managed behind the scenes by an iptables-nft compatibility layer.

## Installing (CLI)

```sh
opkg update
opkg install banip
```

Enable and start it:

```sh
/etc/init.d/banip enable
/etc/init.d/banip start
```

Configuration lives in `/etc/config/banip`, or you can manage it through LuCI under **Services → banIP** if you'd rather use a UI.

## Feeds Worth Turning On

| Feed | What it does |
|---|---|
| `dshieldv4` | Blocks known attacking IPs (SANS DShield list) |
| `dropv4` | Blocks hijacked/spam netblocks (Spamhaus DROP list) |
| `blocklistv4` | General aggregated threat IPs |
| `allowlistv4` | Your manual exceptions, in case a feed ever blocks something it shouldn't |

Enable feeds either in LuCI or by editing the feed list in `/etc/config/banip`, then reload:

```sh
/etc/init.d/banip reload
```

## Checking It's Actually Doing Something

```sh
banip status
logread | grep banip
nft list ruleset | grep -A 5 "WAN-Input"
```

The main enforcement point here is the `WAN-Input` chain. That's where inbound internet traffic gets checked against the feeds before it ever reaches a LAN device.

## What "Normal" Looks Like

Worth checking your invalid-conntrack packet count once things settle (packets that don't match a known connection state). On a stable single-WAN setup, this typically sits around 0.4% of total traffic, which is just background WAN noise (scans, stray retransmits, that kind of thing) rather than a problem. Worth noting the number now, because it's the kind of thing that can look "off" later just from a topology change (like adding a second WAN link) rather than an actual issue. 
