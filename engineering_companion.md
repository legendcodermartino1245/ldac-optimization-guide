# 🛠 LDAC Engineering Companion — System Authority Map (2025 Edition)

This module provides the full technical consolidation of **all verified behaviors, override models, firmware state transitions, and protocol control layers** derived from real-world validation across the entire LDAC Done Right framework.

---

## 🔬 Governing System Model Layers

| Layer | Role | Control Scope |
|-------|------|----------------|
| **A2DP Host Stack (Android/Samsung)** | Primary override injector | Injects Adaptive 32-bit profiles during handshake |
| **BCC (Bluetooth Codec Changer)** | Override neutralizer | Forces explicit codec renegotiation (Auto Switch / Intermediate Profile) |
| **Tasker + AutoNotification** | Healing engine | Monitors codec state and performs self-healing regeneration |
| **Sony Headphone Firmware** | Persistent profile storage | Learns profile handshake sequence (SBC → LDAC 16 → LDAC 990) |
| **Fast Pair Subsystem** | Override reinjection vector | Syncs override profiles unless disabled |
| **Music Center App** | Secondary override | Can inject override upon reconnect unless AV context suppressed |
| **Google Play Services** | Passive override replicator | Stores Fast Pair device metadata unless lockdown applied |
| **AVRCP Layer (Multipoint)** | Controller role negotiator | Controls CT/TG switching, passive control sync, button mapping |
| **DSP Layer (AV ON/OFF)** | Signal integrity modifier | Alters volume scaling, DSEE behavior, and internal processing fidelity |

---

## 🎯 Master Override Defeat Authority Model

| Action Stage | Trigger Mechanism | Stable Defeat Path |
|---------------|-------------------|---------------------|
| Connection Initialization | Samsung injects Adaptive override | BCC Intermediate Profile triggers forced renegotiation |
| Override Collapse | BCC applies target profile | Override displacement applied |
| Persistent Profile Storage | Firmware retains last valid renegotiation | Profile chaining stabilized |
| Reinfection Risk | Music Center or Google Fast Pair resyncs override | Prevent via AV OFF + Fast Pair lockdown |
| Self-Healing | Post-connect desync detected (AutoNotification) | Tasker regenerates profile via healing chain |

---

## 🔬 Adaptive Sample Rate — Stability Tier (Verified)

| Sample Rate | Stability | Remarks |
|--------------|-----------|---------|
| 96 kHz | ✅ Fully Stable | Optimal for Adaptive stack |
| 48 kHz | ✅ Fully Stable | Fully tolerant |
| 88.2 kHz | ⚠ Partially Stable | Tolerant under mirrored multipoint profile |
| 44.1 kHz | ❌ Unstable | Trigger bin shift jitter under load |

---

## 🧠 Firmware Negotiation Window — Session Training Logic

- SBC → LDAC 16-bit → LDAC 990 creates stable profile training chain.
- Training window closes after ~10s idle while fully connected.
- Profile storage survives power cycle if stored cleanly via BCC Auto Switch method.
- Profile loss occurs if Bluetooth stack shuts down while link still active.

---

## 🔧 Multipoint Passive Role Rule (Validated 2025)

- Android acts as passive controller during Windows-driven multipoint.
- Mirrored AV/AVRCP versions across hosts minimize CT/TG swap renegotiation.
- Full two-way control possible under AVRCP 1.6 with synchronized codec profiles.

---

## 🚩 Canonical Debugging Tools

| Tool | Function |
|------|----------|
| ADB dumpsys | Live codec state |
| BCC Internal Log | Negotiation window capture |
| Tasker State Vars | Post-connection healing |
| AutoNotification | GUI desync detection |
| Alternative A2DP Driver (Windows) | Real-time bitrate feedback |

---

## 🏁 Summary Statement

> The entire LDAC Done Right system model operates as a **multi-layered handshake arbitration engine**, chaining Samsung stack suppression, BCC-based renegotiation control, firmware training optimization, AV DSP preservation, and cross-platform multipoint coordination.

---

*This engineering companion is intended for advanced debugging, reverse-engineering alignment, and state machine tracking across full LDAC 990kbps override suppression environments.*
