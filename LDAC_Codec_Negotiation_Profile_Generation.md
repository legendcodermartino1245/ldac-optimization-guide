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
    → Disables Android volume sync interference

---

### 📱 Phone Settings That Affect Codec Negotiation

- **LDAC toggle in Developer Options**  
  → Often causes Samsung devices to renegotiate toward preferred LDAC behavior.  
  → Typically requires an SBC reset and Developer Options OFF during active connection to clear observed renegotiation carryover behavior.

- **Developer Options open during connection**  
  → May retrigger Samsung codec renegotiation behavior if LDAC is selected.  
  → Avoid opening Dev Options during or right before pairing.

- **HD Audio toggle in Bluetooth device settings**  
  → Triggers full codec renegotiation.  
  → May retrigger Samsung codec preference behavior.  
  → Controls whether LDAC is allowed at all in A2DP profile exchange (independent of Developer Options LDAC toggle).

- **Nearby Devices permission (Music Center, GMS)**  
  → May silently influence reconnect negotiation behavior through companion-app communication.  
  → Reapplies stored codec profiles without user interaction.  
  → Must be revoked or app force-stopped to disable.

- **Connection method: Quick Settings vs Power-On**  
  → Reconnecting via **Quick Settings** toggle: more likely to honor stored (trained) profile.  
  → Reconnecting via **powering on headphones**: often re-triggers Samsung renegotiation behavior.

- **Disabling Developer Options while disconnected**  
  → Leaves renegotiation state intact — no reset occurs.

- **Disabling Developer Options while connected**  
  → Clears renegotiation state immediately, allowing your codec profile to apply.

---

### ⚙ Observed System Negotiation Behavior & Profile Persistence

- **Samsung LDAC observed negotiation behavior**  
  → Automatically activates if LDAC is used in Developer Options.  
  → Often renegotiates toward Samsung’s preferred LDAC behavior unless alternative renegotiation workflows intervene.  
  → On Samsung, observed negotiation behavior can independently toggle HD Audio ON after pairing.  
  → However, observed negotiation behavior **cannot toggle LDAC codec toggle directly** — only HD Audio layer.

- **AAC default fallback post-pairing (Samsung stack)**  
  → After first pairing, AAC is no longer neutral.  
  → Becomes part of Samsung’s observed codec negotiation behavior if LDAC isn't fully locked.  

- **Absolute Volume status**  
  - **AV ON**: Volume sync events can re-trigger override or block codec switching.  
  - **AV OFF**: Prevents Android-side volume control.

- **Fast Pair timing**  
  → Samsung negotiation behavior usually takes precedence
  → if Automatically save devices is on it will get stored in the cloud

- **Writing to firmware post-handshake (after override) with Music Center**  
  → Behavior suggests negotiated profiles may persist across reconnects (WH-1000XM5/XM3).  
  → will apply after Samsung renegotiation behavior

- **GUI desync between Developer Options and BCC**  
  → Happens if renegotiation timing conflict occurs.  
  → Solved by double-applying the BCC profile

- **Codec override persists across reboots**  
  → Only cleared via SBC handshake followed by Developer Options OFF during active connection.

---

### 📲 App Behavior That Influences Codec Negotiation

- **Sony | Music Center**  
  → With Nearby Devices permission: silently re-applies LDAC profile at connection.  
  → Override happens even if you only changed volume.  
  → Must be force-stopped or stripped of permission to prevent interference.

- **Bluetooth Codec Changer (BCC)**  
  → Defeats Samsung renegotiation behavior using profile chaining:  
    - SBC → LDAC 16-bit → LDAC 24-bit 990  
  → Must apply within 1–2 seconds of connection.  
  → Double-apply profile to fix GUI mismatch.

- **USB Audio Player PRO (UAPP)**  
  → May re-trigger codec negotiation at playback start.  
  → Can override or conflict with BCC if launched too early.  
  → Best practice: allow BCC to finish first, then launch UAPP.

- **Google Play Services (GMS)**  
  → With Nearby Devices permission: may silently reapply previously negotiated codec behavior.  
  → Often triggered during Fast Pair.  
  → Disable permission to stop this.

- **Tasker (Bluetooth connect triggers)**  
  → Can switch to SBC or intermediate LDAC profiles instantly at connect.  
  → Must run before renegotiation behavior executes (within ~1–2s).  
  → Used to automate profile chaining for override bypass.

- **“Automatically save devices” in Fast Pair**  
  → If enabled, GMS syncs previously negotiated codec profiles to the cloud.  
  → Reapplies LDAC override silently after reset or on new device.  
  → Must be turned OFF to reduce Samsung renegotiation behavior on reconnect.

---

## 🧬 Samsung Codec Behavior — Override Negotiation Logic

Samsung reconnect behavior often appears to prioritize higher-quality codecs after pairing — all codec transitions participate in its override system.

---

### ✅ AAC renegotiation behavior (When LDAC Is Disabled)

- If **LDAC is disabled globally** (HD Audio toggle OFF):
  Samsung devices often negotiate AAC automatically when LDAC is unavailable.
  - This occurs regardless of **Absolute Volume ON or OFF**.
- AAC becomes the "preferred high-quality codec" under Samsung's stack if LDAC is not allowed.
- AAC selection here is not user-driven — it is part of Samsung’s renegotiation behavior asserting itself even in absence of LDAC.
- Disabling LDAC does not fall back to SBC by default; AAC is treated as the next-in-line override.
- **Absolute Volume state has no effect on AAC override behavior**:  

---

### ✅ SBC Is Never Neutral — Samsung Will Never Default to SBC

- Samsung devices rarely appear to default to SBC after initial pairing.
- SBC only appears under very specific conditions:
  - Explicit SBC selection in Developer Options.
  - Manual renegotiation defeat workflow workflows (SBC handshake exploit).
  - Complete device capability fallback (rare edge case if all high-quality codecs fail).
- Even during poor RF conditions or reconnections, Samsung devices often appear to favor Adaptive LDAC degradation before falling back to SBC.
- SBC is therefore not a "graceful fallback" in Samsung's observed negotiation behavior — it is an explicit renegotiation defeat workflow tool.

---

### ✅ AAC ≠ Neutral on Reconnect — It’s Part of Override Negotiation

- AAC is not a fallback or neutral reconnect state.
- On reconnect:
  - Samsung may briefly show **AAC** for several seconds.
  - reconnect behavior may later transition toward Adaptive LDAC unless renegotiation defeat workflow logic is actively blocking it.
  - AAC is simply a transient override phase in Samsung's handshake logic.
- This is why AAC sometimes appears momentarily before BCC or SBC handshake defeats fully assert.

---

### 🔧 Samsung renegotiation behavior Control Priority

| Priority | Control Layer |
|----------|----------------|
| 1️⃣ | HD Audio Toggle (global switch Samsung controls) |
| 2️⃣ | Samsung negotiation behavior often prioritizes AAC or LDAC depending on allowed codec set |
| 3️⃣ | Developer Options LDAC settings mostly ignored unless override is fully defeated |

---

### 🔬 Summary Table

| Scenario | Codec | Override Behavior |
|----------|-------|--------------------|
| LDAC Disabled | AAC | Frequently negotiated automatically on Samsung devices (independent of AV state) |
| LDAC Enabled (Override Active) | AAC → LDAC Adaptive | Sequential override progression |
| renegotiation defeat workflowed (BCC/Tasker) | Anything | Greater manual influence over renegotiation behavior |


# 🎛 LDAC Codec Negotiation — Observed Activation & Renegotiation Behavior

> This module documents observed codec negotiation behavior based on empirical testing behind LDAC activation, delayed switching, Developer Options memory effects, codec renegotiation triggers, and exact codec authority layers across all apps and subsystems.

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

### 🔧 LDAC Toggle — Immediate vs Delayed Activation (Observed consistently in testing)

| Condition | LDAC Switches Automatically? | Notes |
|-----------|----------------------------|-------|
| ✅ Disconnect → Reconnect | ✅ Yes | Always renegotiates codec. |
| ✅ Changing LDAC settings (Developer Options) | ✅ Yes | Developer Options appears to influence codec negotiation during handshake. |
| ⚠ Changing LDAC settings (BCC) | ❌ No (needs renegotiation) | Requires active A2DP session and renegotiation. |
| ⚠ Changing LDAC settings (Music Center) | ✅ Yes | Requires active A2DP session and renegotiation. |
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
- Samsung renegotiation behavior still executes after this unless fully defeated.

---

## 🔑 Absolute Authority Rule — Post-Toggle Codec Injection

> ✅ **Once LDAC toggle is enabled, Developer Options is the only mechanism capable of directly injecting LDAC profile selection without renegotiation.**

| Layer | Can Inject LDAC Profile After Toggle? | Why? |
|-------|-----------------------------------|-------|
| **Developer Options (Codec Memory)** | ✅ YES | Appears to influence A2DP negotiation behavior during connection setup. |
| **Samsung codec preference behavior** | ✅ Yes (at handshake only) | Often negotiates toward Adaptive LDAC profiles during reconnect behavior. |
| **Fast Pair (Google Play Services)** | ❌ NO | Only injects pairing metadata during initial pairing. |
| **Bluetooth Codec Changer (BCC)** | ❌ NO | Requires renegotiation after A2DP is live. |
| **Sony Music Center** | ❌ NO | Issues renegotiation request only. |
| **UAPP** | ❌ NO (Bluetooth) | Can trigger renegotiation via session recreation. |
| **Tasker + AutoNotification** | ❌ NO | Monitors and triggers renegotiation workflows only. |

---

## 🎯 Observed Control Behavior Table — Who Can Control Codecs?

| Layer / App | Authority Type | Can Force Codec Without Live Renegotiation? | Notes |
|--------------|----------------|----------------------------------|-------|
| **Developer Options (Android Bluetooth Settings)** | Observed codec memory behavior | ✅ YES | Appears to strongly influence codec negotiation before or during connection setup. Retains memory across sessions even after disabled. |
| **Samsung codec preference behavior** | Observed Samsung negotiation behavior | ✅ YES (own injected profile only) | Often renegotiates toward Adaptive LDAC profiles during reconnect behavior. |
| **Fast Pair (Google Play Services)** | Pairing Metadata Authority | ✅ YES (during pairing) | May influence codec preferences during initial pairing behavior. |
| **Bluetooth Codec Changer (BCC)** | Runtime Profile Switcher | ❌ NO | Triggers codec renegotiation after A2DP session is live. |
| **Sony Music Center** | App-Level Codec Negotiator | ❌ NO | Requests renegotiation but requires active A2DP session. |
| **UAPP (USB/Hi-Res mode)** | Direct-driver playback behavior | ✅ YES (USB mode) | Can directly manage codec/sample rate for USB audio; Bluetooth still subject to renegotiation logic. |
| **UAPP (Bluetooth mode)** | Session Manager | ❌ NO | Can only trigger renegotiation by restarting playback sessions. |
| **Tasker + AutoNotification** | Automation Healing Layer | ❌ NO | Monitors codec state and issues corrective triggers via BCC or app relaunches. |

---

## 🔧 Codec Control Timing Map

| Timing | Who Can Control? | Method |
|--------|------------------|--------|
| 🔌 **Before Connection (Handshake)** | Developer Options, Samsung renegotiation behavior, Fast Pair | Pre-handshake negotiation influence. |
| 🎶 **After Connection (Active Session)** | BCC, Music Center, UAPP (session restart) | Renegotiation triggers only. |

---

## 🔬 Observed Negotiation Priority Behavior

| Layer | Control Scope |
|-------|----------------|
| **Developer Options Codec Memory** | Strong pre-handshake negotiation influence. |
| **Samsung codec preference behavior** | Often renegotiates toward preferred codec behavior during handshake. |
| **Fast Pair Subsystem** | May influence reconnect negotiation behavior through stored pairing metadata. |
| **Bluetooth Codec Changer (BCC)** | Runtime renegotiation. |
| **Sony Music Center** | Requests renegotiation after session live. |
| **UAPP** | Session management (Bluetooth) / Direct control (USB). |
| **Tasker + AutoNotification** | Codec state monitoring and healing automation. |

---

## 🚀 Relation to LDAC renegotiation defeat workflow Workflow

- The full renegotiation defeat workflow chain works by:
  - Fully resetting Developer Options codec memory state.
  - Forcing Samsung renegotiation behavior reset via SBC → LDAC16 → LDAC990 handshake chaining.
  - Utilizing BCC + Tasker to maintain target profile via controlled renegotiation.
  - Suppressing Fast Pair reinjections via permission revocation timing.

---


### ✅ aptX renegotiation behavior (Unified with AAC Behavior)

Observed Samsung reconnect behavior for aptX and AAC appears similar to its LDAC negotiation behavior when LDAC is unavailable or disabled.

- If **LDAC is disabled globally** (HD Audio toggle OFF or LDAC toggle OFF):
  - Samsung devices often negotiate aptX automatically if supported by both device and headphones.
  - If aptX is not supported, Samsung falls back to **AAC** as the next override layer.
  - Only if both aptX and AAC are unavailable will SBC be negotiated.

- The same Samsung authority behavior applies for both aptX and AAC:
  - HD Audio toggle is often automatically re-enabled at connection when aptX- or AAC-capable headphones are paired.
  - Codec renegotiation behavior often occurs during handshake.
  - Absolute Volume ON or OFF has no effect on Samsung’s decision to inject aptX or AAC.
  - Developer Options can override codec selection manually if codec whitelist manipulation is performed.

- aptX and AAC are never neutral fallback codecs inside Samsung's observed negotiation behavior:
  - Both are active override pathways.
  - Both may briefly appear during initial connection before Samsung may transition to LDAC (if later allowed).
  - SBC is not negotiated unless all high-quality codecs are unavailable or manual renegotiation defeat workflow methods (e.g. SBC handshake exploit) are used.

---

### ✅ HD Audio Toggle Auto-Activation Logic (Full Behavior)

- Samsung devices often automatically re-enable the HD Audio toggle at:
  - First pairing with any LDAC-, aptX-, or AAC-capable device.
  - Every reconnection of any previously paired LDAC-, aptX-, or AAC-capable device.
  - Reconnection after reboot, power cycle, or Bluetooth toggle cycling.
  - Even if user manually disabled HD Audio previously, Samsung re-enables it at handshake.

- This occurs independently of:
  - Developer Options state.
  - Absolute Volume ON or OFF.
  - Google Fast Pair metadata.
  - Manual user interaction.

- HD Audio toggle appears to strongly influence codec negotiation behavior:
  - Strongly associated with whether Samsung devices negotiate LDAC, aptX, or AAC.
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

Observed Samsung negotiation behavior appears to prioritize codecs in the following order based on available capabilities:

| Priority | Codec Injected | Trigger Condition |
|----------|-----------------|--------------------|
| 1️⃣ | LDAC | If LDAC allowed and override active |
| 2️⃣ | aptX | If LDAC unavailable but aptX supported |
| 3️⃣ | AAC | If LDAC and aptX unavailable but AAC supported |
| 4️⃣ | SBC | Only if all others unavailable or renegotiation defeat workflow logic applies |

- The HD Audio toggle appears strongly associated with whether these codec renegotiation behaviors occur.
- Absolute Volume state has no effect on this priority ordering.
- Developer Options codec selections override this priority chain only if explicitly forced.


## 🎯 LDAC Toggle — Playback State Independence

- Enabling the LDAC toggle in Bluetooth settings **only updates the allowed codec whitelist.**
- This whitelist update occurs immediately, regardless of whether music playback is active or paused.
- Playback state does not appear to directly determine codec permission.
- However, playback state can affect *when* renegotiation occurs:
  - If playback is running during whitelist change, the A2DP session will not renegotiate live.
  - If playback is paused or restarted, the next A2DP handshake can negotiate LDAC automatically using the updated whitelist.
- Developer Options appears uniquely capable of influencing renegotiation behavior during active playback.
- BCC (Bluetooth Codec Changer) can trigger renegotiation indirectly by manipulating handshake profiles, even while playback is active.


## 🧬 Observed Samsung Renegotiation & Profile Persistence Behavior

> This module documents the Observed consistently in testing override behavior inside Samsung’s Bluetooth stack, covering profile inheritance, filtering, session isolation, and parameter authority boundaries.

---

### 1️⃣ A2DP Profile Inheritance Authority

Observed behavior suggests Samsung devices may reuse previously negotiated LDAC parameters from the active A2DP session whenever the LDAC toggle (HD Audio ON/OFF) is flipped while the headphones remain connected.

- Inheritance applies to any codec profile negotiated through:
  - Bluetooth Codec Changer (BCC)
  - Sony Music Center
  - Developer Options
  - Android system codec negotiation
- Direct-driver apps like UAPP do not contribute to inheritance; they bypass the A2DP stack entirely.
- Override authority operates strictly at the A2DP session layer.
- Behavior suggests previously negotiated parameters may influence how Samsung devices renegotiate active LDAC profiles during reconnect events.

---

### 2️⃣ Selective Parameter Filtering Logic

Samsung renegotiation behavior does not fully copy every parameter from the inherited A2DP profile. Observed renegotiation behavior suggests some parameter normalization may occur:

| Codec Parameter | Inheritance Behavior |
|------------------|----------------------|
| Bitrate (330/660/990/909 kbps) | Often preserved |
| Bit depth (16/24/32-bit) | Often preserved |
| Sample rate (44.1 / 48 / 88.2 / 96 kHz) | Selective: non-standard rates often normalized to 96 kHz |
| Adaptive vs Fixed | Often renegotiated toward Fixed 990 kbps |

- Exotic profiles (such as 44.1 kHz / 24-bit / 909 kbps) often revert back to 96 kHz / 24-bit / 990 kbps after override injection.
- Observed renegotiation behavior suggests some parameters may be normalized during reconnects before final injection occurs.

---

### 3️⃣ A2DP Session Isolation Protection

Samsung renegotiation behavior only operates within system-managed A2DP sessions. It does not activate inside direct-driver sessions.

| Playback Session Type | Samsung renegotiation behavior Active? |
|------------------------|--------------------------|
| System A2DP session (standard apps, BCC, Music Center) | Yes — override triggers |
| Direct Hi-Res driver session (UAPP, exclusive drivers) | Appears significantly less affected |

- Direct-driver apps bypass the system’s A2DP stack entirely.
- Samsung renegotiation behavior appears less active during direct-driver sessions.
- Direct-driver playback appears significantly less affected by normal Android renegotiation behavior, even when toggling LDAC system settings.

---

### 4️⃣ LDAC Control Authority — Parameter Limitations by Control Source

| Control Source | Sample Rate Control | Bit Depth Control | Bitrate Control | Full LDAC Frame Control? |
|----------------|---------------------|--------------------|------------------|-------------------------|
| Bluetooth Codec Changer (BCC) | Limited — 44.1/88.2 kHz often rejected | Partial | Full (330/660/990/909 kbps) | No |
| Sony Music Center | Limited — sample rate usually restricted to standard rates | Partial | Full | No |
| Developer Options | Limited — controlled by internal stack defaults | Partial | Full | No |
| Android Native System | Strongly influenced by device-specific negotiation behavior | Partial | Full | No |
| UAPP (Hi-Res Direct Driver) | Full — all sample rates allowed (44.1/88.2/96 kHz) | Full | Full | Yes |

---

### 🧬 Master Override Rule

Observed behavior suggests Samsung devices may reuse previously negotiated A2DP codec parameters during LDAC toggle events, but applies internal filtering rules before injecting the profile. Observed renegotiation effects appear significantly reduced for direct-driver playback sessions. Direct-driver apps such as UAPP appear less constrained by normal Android A2DP renegotiation behavior.



## 🔬 Developer Options — Session-Based Authority Model (Observed consistently in testing)

> Developer Options LDAC control operates strictly at the A2DP session level.

| Session Phase | Developer Options Behavior | Injection Authority |
|----------------|-----------------------------|---------------------|
| **Handshake Phase (New Connection)** | Appears to strongly influence LDAC negotiation during connection setup. | ✅ Strong handshake influence. |
| **Active Session (Already Connected)** | Updates codec whitelist only; no codec renegotiation occurs automatically. | ❌ No live injection authority. |

### 🧬 Memory Behavior

- If Developer Options was active during handshake → observed codec preferences may persist across reconnects.
- Disabling Developer Options while disconnected does not clear this memory state.
- Only performing a **SBC handshake reset + Developer Options OFF while connected** fully clears the Developer Options memory carryover.

### 🔑 Absolute Summary Rule

> **Developer Options behavior appears primarily handshake-scoped.**  
> - Active only at handshake phase.  
> - Passive (whitelist only) during live A2DP sessions.
dev options is session based


# 🧬 LDAC Negotiation Behavior — Empirical Behavioral Addendum

This addendum documents an observed behavioral model derived from empirical testing, authority timing, renegotiation models, firmware storage layers, and reinjection subsystems behind Samsung's LDAC override behavior — derived from extensive real-world behavioral testing.

---

## 1️⃣ Unified Override Authority Chain

| Authority | Injection Timing | Function |
|------------|-------------------|----------|
| **Developer Options** | Pre-Handshake | Modifies Android’s Bluetooth codec whitelist before A2DP negotiation. Controls which codecs, sample rates, bit depths, and bitrate modes are allowed. Does **not** inject full codec profiles directly. |
| **Fast Pair (Google Play Services)** | Pre-Handshake (Reinjection Layer) | May influence reconnect negotiation behavior before A2DP negotiation completes if cloud profile syncing is enabled. Participates in Samsung's renegotiation behavior by reusing previously negotiated codec preferences, but cannot bypass Samsung’s override. Samsung devices often renegotiate toward their preferred LDAC profile behavior |
| **Samsung renegotiation behavior** | During Handshake | Often renegotiates toward LDAC parameter sets (Adaptive 96 kHz / 32-bit / 990 kbps) into A2DP session if Fast Pair or Developer Options did not preempt injection. |
| **Bluetooth Codec Changer (BCC)** | Post-Handshake (Active Renegotiation) | Actively forces codec renegotiation after Samsung injection. Can renegotiate the active A2DP profile before firmware memory is updated. |
| **UAPP Direct Driver (HAL)** | Post-Handshake (Active Renegotiation via HAL driver reinit) | Triggers A2DP renegotiation indirectly via playback parameter change (sample rate, bit depth). Reliably observed renegotiation trigger. |
| **Sony Headphone Firmware Profile Memory** | Post-Renegotiation (Passive Learning) | Behavior suggests some negotiated profiles may persist across reconnects *after* valid renegotiation has stabilized. May reapply previously negotiated profiles during future reconnects. |
| **Music Center App** | Post-Handshake (Reinjection Layer) | Reasserts previously stored LDAC profiles after A2DP session opens. Applies correction if session parameters mismatch firmware memory.

---

## 2️⃣ Unified Override Injection & Reinjection Flow

```text
Bluetooth Connection Event
  ├── Developer Options modifies codec whitelist (if active)
  │      ├── Restrictive whitelist → Samsung renegotiation constrained
  │      └── Permissive whitelist → Samsung renegotiation unrestricted
  ├── Fast Pair may influence reconnect negotiation behavior before handshake completion (if cloud profile syncing enabled)
  └── If Fast Pair inactive → Samsung devices often renegotiate toward Adaptive 96 kHz / high-bitrate LDAC behavior during handshake

Post-Handshake Session Opens
  ├── Renegotiation Engines:
  │      ├── BCC (Manual or Tasker): Direct A2DP renegotiation
  │      └── UAPP: HAL parameter change → indirect renegotiation
  │              ├── Sample rate / bit depth change → reliably observed renegotiation
  │              └── HAL idle-resume reopening → renegotiation possible
  ├── Music Center Reinjection:
  │      └── post-handshake profile reapplication based on previously negotiated state
  └── Firmware Learning:
         └── Behavior suggests some negotiated parameters may persist after successful renegotiation

Future Reconnects:
  ├── Fast Pair may influence reconnect negotiation behavior before handshake completion
  └── Music Center may reapply profile post-handshake if mismatch detected
```

