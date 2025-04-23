# nftables Personal Firewall Project

This is a basic but useful firewall setup using **nftables** for personal Linux systems. The goal is to make something simple to understand, that gives a bit of protection, and also to promote the use of `nftables` instead of `iptables`, since it is more modern and flexible.

## Why I chose nftables

Even if many systems still come with `iptables` or tools like `ufw`, I think `nftables` is a better choice today. It lets you manage both IPv4 and IPv6 in one place, the syntax is easier to read and edit, and it’s more powerful when you need to do advanced things.

Also, it helps write cleaner rules, and that makes it easier to come back to the file later and still understand what it does.

## What this firewall does

This configuration was made mainly to:
- Protect SSH from brute-force attempts (with rate limits and a denylist)
- Drop invalid or malformed packets
- Allow ping (ICMP) but in a limited way, so it doesn’t become a problem
- Accept traffic only from trusted IPs
- Log and block everything else

## What it doesn’t do

There are things this firewall cannot do, and it’s good to be clear about that:
- It doesn’t inspect what’s inside a website or HTTPS traffic
- It can’t block web attacks like XSS or SQL injection
- It doesn’t protect from unknown (zero-day) attacks
- It doesn’t act like an IDS or Web Application Firewall

So, this is mostly a **network-level firewall**, not something that checks applications or content.

## Who should use this

This project is good for:
- People who use Linux and want a basic firewall for their system
- Anyone curious to understand how `nftables` works
- Someone who is replacing `iptables` with something more modern
- Users who don’t want a complicated setup, just something clean and working

I made this as a personal project to explore what `nftables` can do, and I think it can be useful to others who want to try it without starting from scratch.

## Simple and open

This is open-source and meant to be reused or modified. The goal is to show how to make a small but working firewall, and also to help others learn the basics of using `nftables`.

---

**License**: See [LICENSE](./LICENSE)

---

*Protect your system. Start simple. Use open tools.*

> Work in progress - feedback welcome!

