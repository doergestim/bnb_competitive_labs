# Lab 08h — WireGuard Handshake, Keepalive & Rekey Capture

## Overview

This lab captures and annotates the full WireGuard packet lifecycle on a local two-peer setup:
initial handshake, transport data, PersistentKeepalive probes, and automatic session rekeying.
All traffic is exchanged between two WireGuard peers connected by a veth pair, so every frame
is visible to `tcpdump -i any` without any physical NIC involved.

---

## Topology
```
  Peer 1 (Initiator)          Peer 2 (Responder)
  wg0  10.99.0.2/30           wg0  10.99.0.1/30
  172.16.42.10:51820  ──────  172.16.42.20:51820
        veth2c92fce              vethb1dbf52
```

Both peers share UDP port **51820** and use a `/30` tunnel subnet (`10.99.0.0/30`).

---

## Packet Types Captured

| Length (bytes) | Type               | Count   |
|----------------|--------------------|---------|
| 148            | Handshake Init     | 6       |
| 92             | Handshake Response | 6       |
| 32             | Keepalive          | 16      |
| 128            | Transport (data)   | 592     |
| **Total**      |                    | **620** |

> Counts include two full rekey cycles, each producing a new Init + Response pair.

---

## Step-by-Step Walkthrough

### Step 1 — Tear Down & Reset

Any existing `wg0` interfaces are destroyed on both peers for a clean state. No stale keys,
handshake state, or routing rules carry over between runs.

### Step 2 — Write Configs (no PersistentKeepalive)

`PersistentKeepalive` is intentionally **omitted** so the handshake is triggered only by real
traffic, making it easy to isolate the Init and Response packets.
```
Peer 1: Address = 10.99.0.2/30,  Endpoint = 172.16.42.20:51820
Peer 2: Address = 10.99.0.1/30   (listen-only until pinged)
```

### Step 3 — Start `tcpdump`
```bash
sudo tcpdump -i any -w capture.pcap udp port 51820
```

Capturing on `any` catches both the **P** (received) and **Out** (sent) direction of every
frame. `editcap` deduplication collapses these pairs in post-processing.

### Step 4 — Bring Up `wg0`
```
ip link add wg0 type wireguard
wg setconf wg0 /dev/fd/63
ip -4 address add 10.99.0.2/30 dev wg0
ip link set mtu 1420 up dev wg0
```

MTU **1420** = 1500 (Ethernet) − 20 (IP) − 8 (UDP) − 32 (WireGuard header) − 16 (Poly1305 tag) − 4 (padding).

No handshake occurs yet — WireGuard is silent until traffic is sent.

### Step 5 — Trigger Handshake (6 pings)

The first ICMP packet causes Peer 1 to initiate a Noise_IKpsk2 handshake.
```
15:20:29.287958  172.16.42.10 → 172.16.42.20  UDP len=148  ← Handshake Initiation
15:20:29.289152  172.16.42.20 → 172.16.42.10  UDP len=92   ← Handshake Response
15:20:29.289444  172.16.42.10 → 172.16.42.20  UDP len=128  ← First transport packet
```

**Message sizes:**
- **148 bytes (Init):** type(4) + reserved(4) + sender-index(4) + ephemeral pubkey(32) +
  encrypted static pubkey(48) + encrypted timestamp(28) + MAC1(16) + MAC2(16)
- **92 bytes (Response):** type(4) + reserved(4) + sender-index(4) + receiver-index(4) +
  ephemeral pubkey(32) + empty encrypted payload + MAC1(16) + MAC2(16)

After the response, both sides derive identical session keys via X25519 + HKDF.
First-ping RTT (~2 ms) includes the handshake overhead; subsequent pings drop to sub-1 ms.

### Step 6 — Enable PersistentKeepalive=25
```bash
wg set wg0 peer <pubkey> persistent-keepalive 25
```

Peer 1 now sends a **32-byte keepalive** every 25 seconds of silence — an empty encrypted
transport message, indistinguishable from real data at the UDP layer.

### Step 7 — 30 Pings
```
30 packets transmitted, 30 received, 0% packet loss
rtt min/avg/max/mdev = 0.462/1.489/2.518/0.512 ms
```

Generates the bulk of the 592 transport packets. The keepalive timer resets on each sent packet.

### Step 8 — First Keepalive Window (55 s silence)

Two keepalives fire within the 55-second window — one at ~25 s and one at ~50 s.
```
Keepalives so far: 2   ✓
```

### Step 9 — Rekey Observation Window (200 s)

WireGuard rekeys after **180 seconds** of session use.

| Elapsed | Latest Handshake Age | Event       |
|---------|----------------------|-------------|
| 20 s    | 1 min 51 s ago       |             |
| 40 s    | 10 s ago             | ← Rekey #1 |
| 60 s    | 30 s ago             |             |
| 160 s   | 10 s ago             | ← Rekey #2 |

Two rekeys occur, each producing a fresh Init + Response pair — consistent with the
6 total Inits and 6 Responses in the capture.

### Step 10 — Post-Rekey Traffic + Second Keepalive Window

Ten fresh pings confirm the new session keys work correctly, followed by another 55 s
silence producing the remaining keepalives.

### Step 11 — Stop & Deduplicate
```
Raw: 620 packets
620 packets seen, 0 packets skipped with duplicate window of 5 packets.
```

The veth P/Out timestamps differ by a few microseconds, placing them just outside the
5-packet dedup window — both directions were retained in the final 620 count.

---

## Verification Checklist

| Check                              | Result      |
|------------------------------------|-------------|
| Handshake Init captured (≥ 2)      | 6        |
| Handshake Response captured (≥ 2)  | 6        |
| Keepalive packets captured (> 0)   | 16       |
| Zero ping packet loss              | 0%       |
| Rekey observed (~180 s cycle)      | 2 rekeys |
| MTU set to 1420                    | confirmed |

---

## Key Concepts Demonstrated

**Silence is golden.** WireGuard generates zero traffic until data needs to be sent or
`PersistentKeepalive` is active — no beacons, no announcements.

**Cookie-less fast path.** Under no load, MAC2 fields are zeroed, keeping handshake
messages at their minimum sizes of 148 and 92 bytes.

**Transparent rekeying.** Sessions rekey automatically every ~180 s with no application
interruption. A brief overlap window keeps both old and new session keys valid for in-flight packets.

**Keepalive ≠ heartbeat.** The 32-byte keepalive is a fully encrypted transport message.
A passive observer cannot distinguish it from any other short UDP datagram on port 51820.

---

## Files

| File                    | Description                      |
|-------------------------|----------------------------------|
| `08h-capture-final.sh`  | Capture automation script        |
| `capture.pcap`          | Raw tcpdump output (620 packets) |
| `capture-dedup.pcap`    | Deduplicated via `editcap`       |

---

*Lab environment: Ubuntu 24.04 · Linux kernel 6.x · WireGuard in-kernel · wg-tools · tcpdump · editcap*
