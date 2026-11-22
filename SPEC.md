# Ghost Network Protocol Specification (GPN-Core / SPEC-001)

- **Status:** Draft / Active (WIP)
- **Author:** Masahiro Sugaya (Section-9.sh)
- **Target Architecture:** Linux (Kernel ≥ 5.15), Go / eBPF / nftables

---

## 1. Architectural Philosophy: The Figure-Ground Reversal

Traditional network privacy designs treat the public Internet as the authoritative foreground and attempt to patch surveillance vulnerabilities through proxies, VPN exit nodes, or obfuscated tunnels. Ghost Network inverts this model:

1. **Substrate Demotion:** The clearnet (IPv4/IPv6, ISP routing, BGP, cellular, commercial optical transit) is demoted to **untrusted, hostile physical plumbing**.
2. **Sovereign Foreground:** The cryptographic overlay—operating in the global Unique Local Address (ULA) space (`fd00::/8`)—is elevated to the primary topology.
3. **Statistically Unmanaged Space:** With $2^{120}$ addressable bits in `fd00::/8`, collisions are mathematically negligible without central registries (ICANN, RIRs, DNS). Central governance is rendered obsolete by raw mathematical scale.

---

## 2. Core Protocol Stack & Layer Mapping

Ghost Network re-maps classic networking primitives onto a flat, zero-server planetary substrate:

| Layer | Traditional Stack | Ghost Network Implementation |
| :--- | :--- | :--- |
| **L7 (Apps)** | Standard Sockets (Web, SSH, Stream) | Unmodified Linux userland applications |
| **L3 (Logical)** | Public IP / RFC 1918 Private IP | Dynamic Cryptographic IPv6 ULA (`fd00::/8`) |
| **L2 (Resolution)**| ARP / IPv6 NDP (Ethernet Broadcast) | **P2P-ARP Signaling**: Key-space query ($O(\log N)$) |
| **L1/Transit** | Physical Ethernet Cable / Switch | Ephemeral UDP Hole-Punched 1-Hop Tunnel (Port 443) |

---

## 3. Subsystem Architecture

### 3.1. Zero-Server Signaling & P2P-ARP (Decoupled Discovery)
- **Anti-Explosion Topology ($O(\log N)$):** Nodes do NOT maintain a full mesh ($O(N^2)$). Each node keeps active signaling links with only $k$ (8–20) neighbor peers in the cryptographic metric space (Winny-inspired neighbor table).
- **Matchmaker Role:** Intermediate nodes only route signaling metadata to resolve destination coordinates. They never carry payload transit traffic.
- **P2P-ARP Resolution:**
  1. Application sends traffic to `fd00:XXXX:...`.
  2. Local daemon intercepts packet via `tun0` and triggers an internal P2P-ARP lookup.
  3. Neighbor graph resolves target node's current external `(Public IP, UDP Port)`.
  4. Both endpoints initiate bidirectional UDP hole-punching.
  5. Direct 1-hop ephemeral encrypted tunnel (WireGuard/Noise-based) is forged.

### 3.2. Unstoppable Transit (UDP Port 443 / QUIC Mimicry)
- All payload encapsulation uses **UDP port 443**.
- Emulates QUIC/HTTP3 packet structure and entropy distribution.
- Deep Packet Inspection (DPI) and state-level firewalls cannot block or throttle Ghost Network traffic without causing widespread collateral outage to legitimate global e-commerce and web traffic.

### 3.3. Temporal Coordinate Hopping (Invisible Handshakes & Listen-Free Sockets)
- **Zero Shodan/Scanner Footprint:** Nodes bind to random ephemeral outbound ports; they never open a fixed listening socket.
- **Deterministic Coordinate Drift:** Nodes derive their active IPv6 ULA address and rendezvous port from a pre-shared master seed / epoch entropy:
  $$\text{Addr}_{\text{epoch}} = \text{KDF}(\text{Seed}, \lfloor t / \Delta t \rfloor)$$
- **Black Hole Immunity:** Unauthorized probing against unauthenticated addresses hits a mathematical vacuum with zero RST, ICMP, or error packet return.

### 3.4. Seamless Handover: Dual-Address Make-Before-Break
To prevent session drops (TCP resets, stream interruptions) during epoch transitions:
1. **Transition Window ($t_{\text{hop}} \pm \delta$):** The local daemon binds **both** the expiring address ($\text{IPv6}_{\text{old}}$) and the incoming address ($\text{IPv6}_{\text{new}}$) simultaneously onto `tun0`.
2. **State Transition:**
   - $\text{IPv6}_{\text{new}}$ is set to **Preferred** (receives all new outbound connections).
   - $\text{IPv6}_{\text{old}}$ is set to **Deprecated** (sustains in-flight connections until socket closure/TTL expiry).
3. **Graceful Teardown:** Once old socket flows reach idle timeout, $\text{IPv6}_{\text{old}}$ is detached from `tun0`. Packet loss during hopping is strictly zero ($0\%$).

### 3.5. "Private-in-Private" Enforcement: Kernel nftables Hook
Isolation of sub-communities within the globally shared `fd00::/8` ocean is guaranteed at the Linux kernel packet filtering hook:
- Dynamic `nftables` sets track active valid cryptographic peers.
- Any incoming packet originating from an unauthenticated ULA coordinate or mismatched cryptographic signature is dropped silently (`counter drop`) at the `netfilter` / `ingress` hook before traversing the host network stack.
- Zero telemetry leaks, zero state exposure.

### 3.6. Rogue Peer Revocation: Mathematical Void
Revocation is serverless and cryptographic:
1. **Asynchronous Dead-Drop:** Epoch entropy is committed to an immutable public substrate (e.g., L1 blockchain or threshold-signed ledger).
2. **Multi-Recipient HPKE:** Seeds are encrypted into discrete key slots per authorized node public key.
3. **Eviction:** To revoke an adversary node, its public key slot is omitted in the next epoch. The evicted node drifts into mathematical vacuum, computing hops on stale entropy. Zero error packets, zero "403 Forbidden" leakage.

---

## 4. Implementation Directives (Phase 1)

1. **`internal/crypto/hop.go`**: Epoch-quantized HKDF generator converting `(Seed, UnixTime, WindowSize)` into `fd00::/8` addresses.
2. **`internal/tun/device.go`**: Linux `ioctl` (`/dev/net/tun`) driver with concurrent dual-IP binding for Make-Before-Break handover.
3. **`internal/p2p/transport.go`**: UDP 443 frame encapsulation with Noise Protocol handshake.
4. **`internal/nftables/firewall.go`**: Netlink dynamic set synchronizer for hardware/kernel packet discard.
