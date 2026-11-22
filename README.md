# ghost-net

> *“AI is Geas.”* — Sovereign, listen-free IPv6 overlay architecture over untrusted physical substrates.

[![Status](https://img.shields.io/badge/status-WIP%20%2F%20Experimental-orange.svg)]()
[![License: GPL v2](https://img.shields.io/badge/License-GPL_v2-blue.svg)](LICENSE)

---

## Overview (WIP)

`ghost-net` is an autonomous, censorship-resistant networking engine that flips the figure and ground of modern connectivity. It demotes the public Internet to untrusted physical plumbing and constructs a sovereign, unmanaged parallel universe inside the vast IPv6 Unique Local Address (`fd00::/8`) space.

### Core Architecture Highlights

- **Figure & Ground Reversal:** Uses the clearnet solely as a carrier substrate for fragmented, uncorrelated UDP noise.
- **Global `fd00::/8` Without Central Registries:** $2^{120}$ addressable bits eliminate address collision risk without requiring ICANN, RIRs, or DNS.
- **P2P-ARP Signaling ($O(\log N)$):** Winny-inspired key-space neighbor discovery decouples route lookup from payload transit, avoiding $O(N^2)$ full-mesh state explosions.
- **Ephemeral 1-Hop Hole-Punching:** Intermediate nodes only facilitate signaling. Actual payload data flows end-to-end via direct UDP 443 tunnels.
- **Dynamic Temporal IP Hopping:** Node coordinates drift deterministically based on cryptographic epoch seeds, presenting a zero-listening-socket black hole to external scanners.
- **Make-Before-Break Handover:** Dual-binds incoming and outgoing IPv6 addresses to `tun0` during hop boundaries to guarantee zero packet loss and seamless session persistence.
- **Private-in-Private Isolation:** Enforced at the Linux kernel boundary using dynamic `nftables` sets.

---

## Repository Structure

```text
ghost-net/
├── README.md              # Project manifesto and architecture overview
├── SPEC.md                # Complete engineering specification
├── LICENSE                # GNU General Public License v2.0
├── Makefile
├── cmd/
│   └── ghostd/            # Core daemon entry point
└── internal/
    ├── crypto/            # Seed derivation and temporal coordinate hopping
    ├── tun/               # Linux tun0 interface & Make-Before-Break dual-binding
    ├── p2p/               # Ephemeral UDP 443 hole-punching and signaling
    ├── nftables/          # Kernel netfilter dynamic set integration
    └── dns/               # Local virtual resolver
