# Understanding IP Addresses: A Complete Beginner's Guide

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/understanding-ip-addresses-beginners-guide-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/understanding-ip-addresses-beginners-guide-en?utm_source=github&utm_medium=referral)**

Right now, as you read this, your device is identified on the internet by a number. Every request you make — loading this page, sending an email, streaming a video — is tagged with that number so that responses know where to come back to. That number is your IP address, and you've been using it every second you've been online.

Most people treat IP addresses like the postal system: they know it works, they're glad it works, and they'd rather not think about it. But understanding IP addresses at a basic level has real practical value — for verifying your VPN is working, diagnosing connectivity problems, understanding what websites can infer about you, and knowing how to protect your privacy online.

This guide explains everything you need to know, without assuming any networking background.

## What Is an IP Address?

An IP address (Internet Protocol address) is a numerical label assigned to every device connected to a network. Think of it as a postal address for your device on the internet.

When you type `google.com` into your browser:
1. Your computer asks a DNS server to look up what IP address corresponds to `google.com`
2. The DNS server returns an IP address (like `142.250.80.46`)
3. Your computer sends a request to that address
4. Google's server sees your IP address and sends the response back to you

Without IP addresses, devices on the internet would have no way to know where to send information — or where responses should go. It's the fundamental addressing system that makes all networked communication possible.

## IPv4: The Original System

**IPv4** (Internet Protocol version 4) is the format most people picture when they think of an IP address: four numbers separated by dots, each between 0 and 255.

Example: `192.168.1.1` or `203.0.113.42`

Each of those four numbers is called an **octet** (8 bits). Four octets × 8 bits = 32 bits total, which gives IPv4 a theoretical maximum of **4,294,967,296 addresses** (about 4.3 billion).

In the early days of the internet, 4 billion addresses seemed like far more than humanity would ever need. That estimate turned out to be catastrophically wrong. By the 1990s, it became clear that IPv4 would eventually run out. By 2011, the global pool of unallocated IPv4 addresses was officially exhausted. Regional registries have been managing a shortage ever since, using techniques like NAT (more on this shortly) to stretch the remaining supply.

## IPv6: The Long-Term Solution

**IPv6** (Internet Protocol version 6) was designed to replace IPv4 and its impending address shortage. Instead of 32 bits, IPv6 uses **128 bits** — which yields approximately **340 undecillion addresses** (340 followed by 36 zeros). For all practical purposes, this is unlimited.

...

---

**[👉 Continue reading: Understanding IP Addresses: A Complete Beginner's Guide](https://tools.cooconsbit.com/en/articles/understanding-ip-addresses-beginners-guide-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
