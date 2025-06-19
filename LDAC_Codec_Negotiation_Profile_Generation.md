# LDAC Codec Negotiation & Profile Generation

> Everything that determines which codec (SBC, LDAC 330/660/990) gets selected during Bluetooth connection.  
> This list is 100% focused on **connection-time behaviors** — not post-connection bitrate changes or audio stability.

---

### 🎧 Headphone & Device Factors

- **Power cycling headphones**  
  → Clears stored codec profile in the headphone’s memory.  
  → Allows a new profile (e.g., LDAC 990) to be stored on next clean connection.

- **Multipoint pairing active**

- **AVRCP version mismatch**  
  → May block proper Absolute Volume detection.  
  → Can disrupt handshake logic or GUI sync.

- **Absolute Volume ON vs OFF**  
  - **AV ON**: Android controls headphone volume directly. 
  - **AV OFF**:
    → Disables Android volume sync interference, enabling clean codec negotiation.

---

### 📱 Phone Settings That Affect Codec Negotiation

- **LDAC toggle in Developer Options**  
  → Activates Samsung’s LDAC override stack.  
  → Must be followed by SBC reset and Developer Options OFF to stop override.

- **Developer Options open during connection**  
  → Re-applies override logic immediately if LDAC is selected.  
  → Avoid opening Dev Options during or right before pairing.

- **HD Audio toggle in Bluetooth device settings**  
  → Triggers full codec renegotiation.  
  → May allow or re-trigger override stack.  
  → Controls whether LDAC is allowed at all in A2DP profile exchange (independent of Developer Options LDAC toggle).

- **Nearby Devices permission (Music Center, GMS)**  
  → Enables silent override via GATT.  
  → Reapplies stored codec profiles without user interaction.  
  → Must be revoked or app force-stopped to disable.

- **Connection method: Quick Settings vs Power-On**  
  → Reconnecting via **Quick Settings** toggle: more likely to honor stored (trained) profile.  
  → Reconnecting via **powering on headphones**: often re-triggers Samsung override.

- **Disabling Developer Options while disconnected**  
  → Leaves override state intact — no reset occurs.

- **Disabling Developer Options while connected**  
  → Clears override state immediately, allowing your codec profile to apply.

---

### ⚙ System Stack Behavior & Profile Storage

- **Samsung LDAC override stack**  
  → Automatically activates if LDAC is used in Developer Options.  
  → Always forces Samsung’s preferred LDAC mode unless bypassed.  
  → On Samsung, override stack can independently toggle HD Audio ON after pairing.  
  → However, override stack **cannot toggle LDAC codec toggle directly** — only HD Audio layer.

- **AAC default fallback post-pairing (Samsung stack)**  
  → After first pairing, AAC is no longer neutral.  
  → Becomes part of Samsung’s override chain if LDAC isn't fully locked.  
  → Even with AV OFF, Samsung may initially negotiate AAC, then force LDAC override within seconds unless SBC profile chaining blocks it.

- **Absolute Volume status**  
  - **AV ON**: Volume sync events can re-trigger override or block codec switching.  
  - **AV OFF**: Prevents Android-side volume control from interfering with profile logic.

- **Fast Pair timing**  
  → Determines which profile wins: Samsung override or user-defined profile.  
  → Override usually applies within 1–2 seconds unless interrupted by SBC chaining.

- **Intermediate profile chaining**  
  → Example: SBC → LDAC 16-bit → LDAC 24-bit 990  
  → Bypasses override stack when done early and with AV OFF.  
  → Essential to force LDAC 990 without triggering Samsung override.

- **Writing to firmware post-handshake (after override) with Music Center**  
  → Locks negotiated profile into headset firmware (WH-1000XM5/XM3).  
  → will apply after samsung override

- **GUI desync between Developer Options and BCC**  
  → Happens if override or stack race condition occurs.  
  → Solved by double-applying the BCC profile and using AV OFF.

- **Codec override persists across reboots**  
  → Only cleared via SBC handshake followed by Developer Options OFF during active connection.

---

### 📲 App Behavior That Influences Codec Negotiation

- **Sony | Music Center**  
  → With Nearby Devices permission: silently re-applies LDAC profile at connection.  
  → Override happens even if you only changed volume.  
  → Must be force-stopped or stripped of permission to prevent interference.

- **Bluetooth Codec Changer (BCC)**  
  → Defeats Samsung override using profile chaining:  
    - SBC → LDAC 16-bit → LDAC 24-bit 990  
  → Must apply within 1–2 seconds of connection.  
  → Double-apply profile to fix GUI mismatch.

- **USB Audio Player PRO (UAPP)**  
  → May re-trigger codec negotiation at playback start.  
  → Can override or conflict with BCC if launched too early.  
  → Best practice: allow BCC to finish first, then launch UAPP.

- **Google Play Services (GMS)**  
  → With Nearby Devices permission: silently applies stored override.  
  → Often triggered during Fast Pair.  
  → Disable permission to stop this.

- **Tasker (Bluetooth connect triggers)**  
  → Can switch to SBC or intermediate LDAC profiles instantly at connect.  
  → Must run before override logic executes (within ~1–2s).  
  → Used to automate profile chaining for override bypass.

- **“Automatically save devices” in Fast Pair**  
  → If enabled, GMS syncs override profiles to the cloud.  
  → Reapplies LDAC override silently after reset or on new device.  
  → Must be turned OFF to prevent Samsung override returning.

---

## 🧬 Samsung Codec Behavior — Override Negotiation Logic

Samsung's codec handling is **never neutral** after pairing — all codec transitions participate in its override system.

---

### ✅ AAC Override Logic (When LDAC Is Disabled)

- If **LDAC is disabled globally** (HD Audio toggle OFF):
  - Samsung will forcibly activate **AAC** whenever possible.
  - This occurs regardless of **Absolute Volume ON or OFF**.
- AAC becomes the "preferred high-quality codec" under Samsung's stack if LDAC is not allowed.
- AAC selection here is not user-driven — it is part of Samsung’s override logic asserting itself even in absence of LDAC.
- Disabling LDAC does not fall back to SBC by default; AAC is treated as the next-in-line override.
- **Absolute Volume state has no effect on AAC override behavior**:  
  → AV OFF does not prevent AAC default activation when LDAC is disabled.

---

### ✅ SBC Is Never Neutral — Samsung Will Never Default to SBC

- Samsung’s pairing and reconnect behavior **never defaults to SBC** after initial pairing.
- SBC only appears under very specific conditions:
  - Explicit SBC selection in Developer Options.
  - Manual override defeat workflows (SBC handshake exploit).
  - Complete device capability fallback (rare edge case if all high-quality codecs fail).
- Even during poor RF conditions or reconnections, Samsung prefers Adaptive LDAC degradation over SBC fallback.
- SBC is therefore not a "graceful fallback" in Samsung's override stack — it is an explicit override defeat tool.

---

### ✅ AAC ≠ Neutral on Reconnect — It’s Part of Override Negotiation

- AAC is not a fallback or neutral reconnect state.
- On reconnect:
  - Samsung may briefly show **AAC** for several seconds.
  - The override system then auto-transitions to **LDAC Adaptive** unless override defeat logic is actively blocking it.
  - AAC is simply a transient override phase in Samsung's handshake logic.
- This is why AAC sometimes appears momentarily before BCC or SBC handshake defeats fully assert.

---

### 🔧 Samsung Override Control Priority

| Priority | Control Layer |
|----------|----------------|
| 1️⃣ | HD Audio Toggle (global switch Samsung controls) |
| 2️⃣ | Override stack injects AAC or LDAC depending on allowed codec set |
| 3️⃣ | Developer Options LDAC settings mostly ignored unless override is fully defeated |

---

### 🔬 Summary Table

| Scenario | Codec | Override Behavior |
|----------|-------|--------------------|
| LDAC Disabled | AAC | Forced by Samsung (independent of AV state) |
| LDAC Enabled (Override Active) | AAC → LDAC Adaptive | Sequential override progression |
| Override Defeated (BCC/Tasker) | Anything | Full manual handshake control |


# 🎛 LDAC Codec Negotiation — Activation, Memory Effects & Authority Model (Final Engineering Companion Edition)

> This module documents the complete verified behavior behind LDAC activation, delayed switching, Developer Options memory effects, codec renegotiation triggers, and exact codec authority layers across all apps and subsystems.

---

## 🎯 LDAC Toggle Behavior — Activation vs Actual Codec Switch

- The LDAC toggle in Android Bluetooth settings **only updates the allowed codec list**.
- It does **not** guarantee immediate LDAC activation during live playback.
- Actual codec switching depends on:
  - A2DP session state
  - Playback state
  - Prior Developer Options usage
  - Codec memory state inside Android’s Bluetooth stack.

---

### 🔧 LDAC Toggle — Immediate vs Delayed Activation (Fully Validated)

| Condition | LDAC Switches Automatically? | Notes |
|-----------|----------------------------|-------|
| ✅ Disconnect → Reconnect | ✅ Yes | Always renegotiates codec. |
| ✅ Changing LDAC settings (Developer Options) | ✅ Yes | Developer Options directly injects codec pre-handshake. |
| ⚠ Changing LDAC settings (BCC, Music Center) | ❌ No (needs renegotiation) | Requires active A2DP session and renegotiation. |
| ⚠ Changing sample rate via UAPP | ✅ Yes (indirect) | HAL session reset triggers renegotiation. |
| ⚠ Pause + Resume (media apps) | ❌ no | Session often stays alive. |
| 🔴 Active playback ongoing during toggle | ❌ No | Fully locked session. |
| 🔴 Short pause (few sec) | ❌ No | A2DP stack remains alive. |
| 🔴 No external renegotiation triggers | ❌ No | Toggle alone insufficient. |

---

## 🧬 Developer Options Memory Carryover Behavior

> Prior use of Developer Options codec selection can cause LDAC to activate automatically even without renegotiation.

| Developer Options History | Behavior on Reconnect |
|---------------------------|-----------------------|
| LDAC previously selected in Developer Options | LDAC always activates automatically upon reconnect. |
| Developer Options disabled, but SBC reset not performed | LDAC memory still applied automatically. |
| Developer Options disabled **and** SBC handshake performed (override reset) | LDAC activation requires full renegotiation. |

- This codec memory state lives inside Android’s Bluetooth stack (not Sony firmware).
- Samsung override still executes after this unless fully defeated.

---

## 🔑 Absolute Authority Rule — Post-Toggle Codec Injection

> ✅ **Once LDAC toggle is enabled, Developer Options is the only mechanism capable of directly injecting LDAC profile selection without renegotiation.**

| Layer | Can Inject LDAC Profile After Toggle? | Why? |
|-------|-----------------------------------|-------|
| **Developer Options (Codec Memory)** | ✅ YES | Injects codec directly into A2DP negotiation stack. |
| **Samsung Override Stack** | ✅ Yes (at handshake only) | Injects Adaptive LDAC default profile. |
| **Fast Pair (Google Play Services)** | ❌ NO | Only injects pairing metadata during initial pairing. |
| **Bluetooth Codec Changer (BCC)** | ❌ NO | Requires renegotiation after A2DP is live. |
| **Sony Music Center** | ❌ NO | Issues renegotiation request only. |
| **UAPP** | ❌ NO (Bluetooth) | Can trigger renegotiation via session recreation. |
| **Tasker + AutoNotification** | ❌ NO | Monitors and triggers renegotiation workflows only. |

---

## 🎯 Master Authority Table — Who Can Control Codecs?

| Layer / App | Authority Type | Can Force Codec Without Live Renegotiation? | Notes |
|--------------|----------------|----------------------------------|-------|
| **Developer Options (Android Bluetooth Settings)** | Codec Memory Authority | ✅ YES | Injects codec directly into A2DP stack before/during connection. Retains memory across sessions even after disabled. |
| **Samsung Override Stack** | Vendor Override Stack | ✅ YES (own injected profile only) | Injects Adaptive LDAC profile at handshake unless blocked by reset. |
| **Fast Pair (Google Play Services)** | Pairing Metadata Authority | ✅ YES (during pairing) | Injects codec preferences during initial pairing handshake only. |
| **Bluetooth Codec Changer (BCC)** | Runtime Profile Switcher | ❌ NO | Triggers codec renegotiation after A2DP session is live. |
| **Sony Music Center** | App-Level Codec Negotiator | ❌ NO | Requests renegotiation but requires active A2DP session. |
| **UAPP (USB/Hi-Res mode)** | Direct Driver Control | ✅ YES (USB mode) | Can directly manage codec/sample rate for USB audio; Bluetooth still subject to renegotiation logic. |
| **UAPP (Bluetooth mode)** | Session Manager | ❌ NO | Can only trigger renegotiation by restarting playback sessions. |
| **Tasker + AutoNotification** | Automation Healing Layer | ❌ NO | Monitors codec state and issues corrective triggers via BCC or app relaunches. |

---

## 🔧 Codec Control Timing Map

| Timing | Who Can Control? | Method |
|--------|------------------|--------|
| 🔌 **Before Connection (Handshake)** | Developer Options, Samsung Override, Fast Pair | Direct codec injection authority. |
| 🎶 **After Connection (Active Session)** | BCC, Music Center, UAPP (session restart) | Renegotiation triggers only. |

---

## 🔬 System Authority Layer Hierarchy

| Layer | Control Scope |
|-------|----------------|
| **Developer Options Codec Memory** | Full injection pre-handshake. |
| **Samsung Override Stack** | Injects override profile at handshake. |
| **Fast Pair Subsystem** | Injects pairing metadata codec preferences. |
| **Bluetooth Codec Changer (BCC)** | Runtime renegotiation. |
| **Sony Music Center** | Requests renegotiation after session live. |
| **UAPP** | Session management (Bluetooth) / Direct control (USB). |
| **Tasker + AutoNotification** | Codec state monitoring and healing automation. |

---

## 🚀 Relation to LDAC Override Defeat Workflow

- The full override defeat chain works by:
  - Fully resetting Developer Options codec memory state.
  - Forcing Samsung override reset via SBC → LDAC16 → LDAC990 handshake chaining.
  - Utilizing BCC + Tasker to maintain target profile via controlled renegotiation.
  - Suppressing Fast Pair reinjections via permission revocation timing.

---


### ✅ aptX Override Logic (Unified with AAC Behavior)

Samsung’s override stack applies identical injection logic for aptX and AAC when LDAC is unavailable or disabled.

- If **LDAC is disabled globally** (HD Audio toggle OFF or LDAC toggle OFF):
  - Samsung will forcibly activate **aptX** if supported by both device and headphones.
  - If aptX is not supported, Samsung falls back to **AAC** as the next override layer.
  - Only if both aptX and AAC are unavailable will SBC be negotiated.

- The same Samsung authority behavior applies for both aptX and AAC:
  - HD Audio toggle is automatically enabled at connection when aptX- or AAC-capable headphones are paired.
  - Override injection happens immediately at handshake.
  - Absolute Volume ON or OFF has no effect on Samsung’s decision to inject aptX or AAC.
  - Developer Options can override codec selection manually if codec whitelist manipulation is performed.

- aptX and AAC are never neutral fallback codecs inside Samsung's override stack:
  - Both are active override pathways.
  - Both may briefly appear during initial connection before Samsung may transition to LDAC (if later allowed).
  - SBC is not negotiated unless all high-quality codecs are unavailable or manual override defeat methods (e.g. SBC handshake exploit) are used.

---

### ✅ HD Audio Toggle Auto-Activation Logic (Full Behavior)

- Samsung automatically enables the HD Audio toggle at:
  - First pairing with any LDAC-, aptX-, or AAC-capable device.
  - Every reconnection of any previously paired LDAC-, aptX-, or AAC-capable device.
  - Reconnection after reboot, power cycle, or Bluetooth toggle cycling.
  - Even if user manually disabled HD Audio previously, Samsung re-enables it at handshake.

- This occurs independently of:
  - Developer Options state.
  - Absolute Volume ON or OFF.
  - Google Fast Pair metadata.
  - Manual user interaction.

- HD Audio toggle acts as Samsung's override authority gate:
  - Required for Samsung to inject LDAC, aptX or AAC.
  - Toggle state is vendor-managed at handshake level.

| Event | HD Audio Toggle Behavior |
|-------|--------------------------|
| First pairing (LDAC/aptX/AAC capable device) | ✅ Auto-enabled |
| Reconnection after reboot / power cycle | ✅ Auto-enabled |
| Reconnection after manual HD Audio disable | ✅ Auto-reenabled |
| SBC-only device | 🔴 No change |
| Device unpaired and fully removed | 🔴 Reset to default OFF |

> **Note:** Samsung never toggles the LDAC codec toggle directly. Only HD Audio toggle is manipulated automatically as part of Samsung’s vendor override activation logic.

---

### 🔧 Samsung Codec Override Injection Priority

Samsung's override stack injects codecs in strict priority order based on available capabilities:

| Priority | Codec Injected | Trigger Condition |
|----------|-----------------|--------------------|
| 1️⃣ | LDAC | If LDAC allowed and override active |
| 2️⃣ | aptX | If LDAC unavailable but aptX supported |
| 3️⃣ | AAC | If LDAC and aptX unavailable but AAC supported |
| 4️⃣ | SBC | Only if all others unavailable or override defeat logic applies |

- The HD Audio toggle must be ON for any of these override injections to execute.
- Absolute Volume state has no effect on this priority ordering.
- Developer Options codec selections override this priority chain only if explicitly forced.


## 🎯 LDAC Toggle — Playback State Independence

- Enabling the LDAC toggle in Bluetooth settings **only updates the allowed codec whitelist.**
- This whitelist update occurs immediately, regardless of whether music playback is active or paused.
- Playback state has no authority over codec permission.
- However, playback state can affect *when* renegotiation occurs:
  - If playback is running during whitelist change, the A2DP session will not renegotiate live.
  - If playback is paused or restarted, the next A2DP handshake can negotiate LDAC automatically using the updated whitelist.
- Developer Options remains the only authority that can immediately force renegotiation during active playback.
- BCC (Bluetooth Codec Changer) can trigger renegotiation indirectly by manipulating handshake profiles, even while playback is active.


## 🧬 Samsung Override — Full Authority Chain Behavior (Finalized Edition)

> This module documents the fully validated override behavior inside Samsung’s Bluetooth stack, covering profile inheritance, filtering, session isolation, and parameter authority boundaries.

---

### 1️⃣ A2DP Profile Inheritance Authority

Samsung’s override system dynamically inherits LDAC profiles from the active A2DP session whenever the LDAC toggle (HD Audio ON/OFF) is flipped while the headphones remain connected.

- Inheritance applies to any codec profile negotiated through:
  - Bluetooth Codec Changer (BCC)
  - Sony Music Center
  - Developer Options
  - Android system codec negotiation
- Direct-driver apps like UAPP do not contribute to inheritance; they bypass the A2DP stack entirely.
- Override authority operates strictly at the A2DP session layer.
- Override logic reads and copies the currently active A2DP LDAC profile into its injection logic.

---

### 2️⃣ Selective Parameter Filtering Logic

Samsung override does not fully copy every parameter from the inherited A2DP profile. Internal filtering applies:

| Codec Parameter | Inheritance Behavior |
|------------------|----------------------|
| Bitrate (330/660/990/909 kbps) | Fully inherited |
| Bit depth (16/24/32-bit) | Fully inherited |
| Sample rate (44.1 / 48 / 88.2 / 96 kHz) | Selective: non-standard rates often normalized to 96 kHz |
| Adaptive vs Fixed | Often forced into Fixed 990 kbps |

- Exotic profiles (such as 44.1 kHz / 24-bit / 909 kbps) often revert back to 96 kHz / 24-bit / 990 kbps after override injection.
- Samsung validates and adjusts parameters based on internal profile limits before final injection occurs.

---

### 3️⃣ A2DP Session Isolation Protection

Samsung override only operates within system-managed A2DP sessions. It does not activate inside direct-driver sessions.

| Playback Session Type | Samsung Override Active? |
|------------------------|--------------------------|
| System A2DP session (standard apps, BCC, Music Center) | Yes — override triggers |
| Direct Hi-Res driver session (UAPP, exclusive drivers) | No — override fully bypassed |

- Direct-driver apps bypass the system’s A2DP stack entirely.
- Samsung’s override logic has no injection access during direct-driver sessions.
- Direct-driver playback creates a fully protected override-safe zone, even when toggling LDAC system settings.

---

### 4️⃣ LDAC Control Authority — Parameter Limitations by Control Source

| Control Source | Sample Rate Control | Bit Depth Control | Bitrate Control | Full LDAC Frame Control? |
|----------------|---------------------|--------------------|------------------|-------------------------|
| Bluetooth Codec Changer (BCC) | Limited — 44.1/88.2 kHz often rejected | Partial | Full (330/660/990/909 kbps) | No |
| Sony Music Center | Limited — sample rate usually restricted to standard rates | Partial | Full | No |
| Developer Options | Limited — controlled by internal stack defaults | Partial | Full | No |
| Android Native System | Fully controlled by Samsung's internal negotiation rules | Partial | Full | No |
| UAPP (Hi-Res Direct Driver) | Full — all sample rates allowed (44.1/88.2/96 kHz) | Full | Full | Yes |

---

### 🧬 Master Override Rule

Samsung override inherits the active A2DP codec profile during LDAC toggle events, but applies internal filtering rules before injecting the profile. Override authority is fully bypassed for direct-driver playback sessions. Only direct-driver apps (such as UAPP) have full LDAC parameter control.



## 🔬 Developer Options — Session-Based Authority Model (Fully Validated)

> Developer Options LDAC control operates strictly at the A2DP session level.

| Session Phase | Developer Options Behavior | Injection Authority |
|----------------|-----------------------------|---------------------|
| **Handshake Phase (New Connection)** | Injects LDAC codec profile directly into A2DP negotiation. | ✅ Full injection authority. |
| **Active Session (Already Connected)** | Updates codec whitelist only; no codec renegotiation occurs automatically. | ❌ No live injection authority. |

### 🧬 Memory Behavior

- If Developer Options was active during handshake → codec profile injection persists in system memory across reconnects.
- Disabling Developer Options while disconnected does not clear this memory state.
- Only performing a **SBC handshake reset + Developer Options OFF while connected** fully clears the Developer Options memory carryover.

### 🔑 Absolute Summary Rule

> **Developer Options = Session-based codec injection authority.**  
> - Active only at handshake phase.  
> - Passive (whitelist only) during live A2DP sessions.
dev options is session based


# 🧬 LDAC Override Authority Model — Unified Engineering Addendum (v3.5 Final Master Model)

This addendum documents the full internal injection logic, authority timing, renegotiation models, firmware storage layers, and reinjection subsystems behind Samsung's LDAC override behavior — fully reverse-engineered from real-world system behavior.

---

## 1️⃣ Unified Override Authority Chain

| Authority | Injection Timing | Function |
|------------|-------------------|----------|
| **Developer Options** | Pre-Handshake | Modifies Android’s Bluetooth codec whitelist before A2DP negotiation. Controls which codecs, sample rates, bit depths, and bitrate modes are allowed. Does **not** inject full codec profiles directly. |
| **Fast Pair (Google Play Services)** | Pre-Handshake (Reinjection Layer) | Injects previously learned profiles before A2DP handshake completes if cloud profile syncing is enabled. Can fully bypass Samsung override injection. |
| **Samsung Override** | During Handshake | Injects LDAC codec parameters (Adaptive 96 kHz / 32-bit / 990 kbps) into A2DP session if Fast Pair or Developer Options did not preempt injection. |
| **Bluetooth Codec Changer (BCC)** | Post-Handshake (Active Renegotiation) | Actively forces codec renegotiation after Samsung injection. Fully overrides active A2DP profile before firmware memory is updated. |
| **Tasker (via BCC control)** | Post-Handshake Automation | Indirect control via BCC API. Triggers BCC renegotiation actions but holds no native injection authority itself. |
| **UAPP Direct Driver (HAL)** | Post-Handshake (Active Renegotiation via HAL driver reinit) | Triggers A2DP renegotiation indirectly via playback parameter change (sample rate, bit depth). Fully valid renegotiation authority. |
| **Sony Headphone Firmware Profile Memory** | Post-Renegotiation (Passive Learning) | Learns and stores LDAC profiles *after* valid renegotiation has stabilized. Applies stored profiles during future reconnects. |
| **Music Center App** | Post-Handshake (Reinjection Layer) | Reasserts previously stored LDAC profiles after A2DP session opens. Applies correction if session parameters mismatch firmware memory.

---

## 2️⃣ Unified Override Injection & Reinjection Flow

```text
Bluetooth Connection Event
  ├── Developer Options modifies codec whitelist (if active)
  │      ├── Restrictive whitelist → Samsung blocked
  │      └── Permissive whitelist → Samsung allowed
  ├── Fast Pair may inject full profile pre-handshake (if cloud profile syncing enabled)
  └── If Fast Pair inactive → Samsung injects Adaptive 96/32 profile during handshake

Post-Handshake Session Opens
  ├── Renegotiation Engines:
  │      ├── BCC (Manual or Tasker): Direct A2DP renegotiation
  │      └── UAPP: HAL parameter change → indirect renegotiation
  │              ├── Sample rate / bit depth change → guaranteed renegotiation
  │              └── HAL idle-resume reopening → renegotiation possible
  ├── Music Center Reinjection:
  │      └── Post-handshake profile reassertion from firmware memory
  └── Firmware Learning:
         └── Firmware stores profile after valid renegotiation

Future Reconnects:
  ├── Fast Pair may inject stored profile pre-handshake
  └── Music Center may reapply profile post-handshake if mismatch detected
```

