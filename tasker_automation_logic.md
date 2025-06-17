
# 🎯 LDAC Done Right — Universal Override Auto-Switch Logic
This section documents the full override defeat engine architecture, combining BCC Auto Switch, Intermediate Profiles, AutoNotification healing, and Tasker correction logic across all Samsung LDAC override defeat classes.---



## 🔧 1️⃣ 96 kHz 32-bit ANY Bitrate Override Auto Switch Logic  
*Class 3 — Injected Override Preemption*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `96 kHz / 32-bit / ANY Bitrate`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- Samsung injects override (`96kHz / 32-bit / Adaptive`).
- Intermediate Profile forces immediate codec renegotiation.
- Auto Switch applies target profile (`44.1 / 24 / 909`).
- AutoNotification monitors GUI desync and triggers healing if necessary.

---

## 🔧 2️⃣ AV Override Auto Switch Logic  
*Class 1 — AV OFF Stack Suppression*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `ANY Sample Rate / ANY Bit Depth / ANY Bitrate`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- AV OFF disables Samsung override injection at stack capability layer.
- Auto Switch applies target profile directly.
- AutoNotification healing remains active for full GUI desync protection.

---

## 🔧 3️⃣ SBC Auto Switch Logic  
*Class 2.1 — Non-LDAC Handshake Entry: SBC*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `SBC 44.1 kHz / 16-bit`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- SBC handshake fully blocks Samsung override injection.
- BCC transitions to target LDAC profile.
- Self-healing remains active for any desync correction.

---

## 🔧 4️⃣ AAC Auto Switch Logic  
*Class 2.2 — Non-LDAC Handshake Entry: AAC*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `AAC 44.1 kHz / 16-bit`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- AAC handshake prevents Samsung override injection.
- BCC transitions to LDAC `44.1 / 24 / 909`.
- AutoNotification monitors for correction as needed.

---

## 🔧 5️⃣ aptX Auto Switch Logic  
*Class 2.3 — Non-LDAC Handshake Entry: aptX*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `aptX 44.1 kHz / 16-bit`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- aptX handshake prevents override injection.
- BCC transitions to LDAC `44.1 / 24 / 909`.
- Self-healing engine corrects any GUI desync automatically.

---





## 🔧 Universal Self-Healing Logic (AutoNotification + Tasker Logic)

| Trigger Condition | Healing Action |
|--------------------|----------------|
| Sample Rate ≠ 44.1 kHz | Apply `44.1 kHz / 24-bit / 909` |
| Bit Depth ≠ 24-bit | Apply `44.1 kHz / 24-bit / 909` |
| Bitrate = Default (303/606/etc) **AND Codec = LDAC** | GUI Sync Only |
| Bitrate ≠ 909 (when LDAC active) | Apply Bit Depth Regeneration |
| Codec Change Event (non-LDAC → LDAC or intra-LDAC renegotiation) | Samsung Override Fully Displaced |
| Otherwise | No Action |





# LDAC Auto-Correction Logic — Final Model

## 🔧 Setup Context

- **Target Profile:** `44.1 kHz / 24-bit / 909 kbps`
- **Components:**
  - BCC Auto Switch (`0ms` execution delay)
  - Intermediate Profile: `96 kHz / 32-bit / 990 kbps`
  - Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
  - AutoNotification profile triggers
  - 2-Step: Off

---

## 🗒️ Bluetooth Codec Changer (BCC) Internal Logs

- BCC maintains **internal logs** that record profile switching events, codec negotiation attempts, and any applied self-healing triggers.
- These logs are extremely useful for:
  - Verifying Tasker automation correctness.
  - Debugging profile application order.
  - Detecting unexpected renegotiation events.
- Logs can be accessed directly from within BCC under the `Logs` or `Debug` section (depending on version).
- Ideal for verifying whether intermediate profile chaining or override defeats executed as intended.


# 🎯 Tasker Profile → Task Mapping (Samsung_Override.prj.xml)

This mapping details each Tasker task by ID and name, its intended purpose, and the list of profiles that trigger it.

---

### 🔧 Task ID: 102 — `Always Works Unless Quality Mismatch`
**Purpose:** Main fallback handler for successful BCC codec switches; used when quality is acceptable.

**Linked Profiles:**
- AAC_44100_16
- SBC_44100_16
- LDAC_44100_16_909
- LDAC_44100_16_Default
- LDAC_44100_24_303
- LDAC_44100_24_606
- LDAC_44100_24_909
- LDAC_44100_24_Default
- LDAC_44100_32_303
- LDAC_44100_32_606
- LDAC_44100_32_909
- LDAC_48000_16_330
- LDAC_48000_16_660
- LDAC_48000_16_990
- LDAC_48000_16_Default
- LDAC_48000_24_330
- LDAC_48000_24_660
- LDAC_48000_24_990
- LDAC_48000_24_Default
- LDAC_48000_32_330
- LDAC_48000_32_660
- LDAC_48000_32_990
- LDAC_48000_32_Default
- LDAC_88200_16_909
- LDAC_88200_24_303
- LDAC_88200_24_606
- LDAC_88200_24_909
- LDAC_88200_32_303
- LDAC_88200_32_606
- LDAC_88200_32_909
- LDAC_88200_32_Default
- LDAC_96000_16_330
- LDAC_96000_16_660
- LDAC_96000_16_990
- LDAC_96000_24_330
- LDAC_96000_24_660
- LDAC_96000_24_990
- LDAC_96000_24_Default
- LDAC_96000_32_330
- LDAC_96000_32_660
- LDAC_96000_32_Default

---

### ⚠️ Task ID: 23 — `Bit Depth Regeneration`
**Purpose:** Adaptive codec detection and override mismatch correction.

**Linked Profiles:**
- LDAC_44100_16_303
- LDAC_44100_16_606
- LDAC_44100_16_ADAPTIVE
- LDAC_44100_24_ADAPTIVE
- LDAC_44100_32_ADAPTIVE
- LDAC_48000_16_ADAPTIVE
- LDAC_48000_24_ADAPTIVE
- LDAC_48000_32_ADAPTIVE
- LDAC_88200_16_303
- LDAC_88200_16_606
- LDAC_88200_16_ADAPTIVE
- LDAC_88200_24_ADAPTIVE
- LDAC_88200_32_ADAPTIVE
- LDAC_96000_16_ADAPTIVE
- LDAC_96000_24_ADAPTIVE
- LDAC_96000_32_ADAPTIVE

---

### 🧪 Task ID: 17 — `Sbc Override`
**Purpose:** Force SBC handshake to reset Samsung override authority.

**Linked Profile:**
- LDAC_96000_32_990

---

### 🔍 Task ID: 15 — `Ldac 660`
**Purpose:** for apps that need bleutooth and wifi scanning which is unstable at 990 or 909 so switched back t0 606 or 909

**Linked Profile:**
- Bbc Maps

---

### 🎧 Task ID: 3 — `Location Off`
**Purpose:** Likely toggles environment prep or scanning off after connection.

**Linked Profiles:**
- Xm3 Connected
- Xm5 Connected

---

### 🔌 Task ID: 5 — `Location On`
**Purpose:** Environment or scanning restore after disconnect.

**Linked Profiles:**
- Xm3 Disconnect
- Xm5 Disconnect

---

### 🎶 Task ID: 74 — `Enable Profiles After High Res`
**Purpose:** Resume BCC switching after closing Qobuz.

**Linked Profile:**
- Qobuz Enable Bcc Profiles

---

### 🎛 Task ID: 73 — `Disable Profiles For High Res`
**Purpose:** Prevent profile switching during UAPP playback.

**Linked Profile:**
- UAPP Disable Bcc Profiles

## 📲 AutoNotification Integration

This Tasker project contains fully integrated **AutoNotification Intercept profiles** that:
- Monitor Bluetooth codec notifications (LDAC, SBC, APTX, AAC)
- Automatically trigger codec override tasks like:
  - `Sbc Override`
  - `Bit Depth Regeneration`
  - `Ldac 660`

These are embedded inside the Tasker `.prj.xml` using:
- `<plugintypeid>com.joaomgcd.autonotification.intent.IntentInterceptNotification</plugintypeid>`
- And are activated as soon as AutoNotification is installed and allowed notification access.


# 🎯 LDAC Done Right — Universal Override Auto-Switch Logic
This section documents the full override defeat engine architecture, combining BCC Auto Switch, Intermediate Profiles, AutoNotification healing, and Tasker correction logic across all Samsung LDAC override defeat classes.---



## 🔧 1️⃣ 96 kHz 32-bit ANY Bitrate Override Auto Switch Logic  
*Class 3 — Injected Override Preemption*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `96 kHz / 32-bit / ANY Bitrate`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- Samsung injects override (`96kHz / 32-bit / Adaptive`).
- Intermediate Profile forces immediate codec renegotiation.
- Auto Switch applies target profile (`44.1 / 24 / 909`).
- AutoNotification monitors GUI desync and triggers healing if necessary.

---

## 🔧 2️⃣ AV Override Auto Switch Logic  
*Class 1 — AV OFF Stack Suppression*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `ANY Sample Rate / ANY Bit Depth / ANY Bitrate`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- AV OFF disables Samsung override injection at stack capability layer.
- Auto Switch applies target profile directly.
- AutoNotification healing remains active for full GUI desync protection.

---

## 🔧 3️⃣ SBC Auto Switch Logic  
*Class 2.1 — Non-LDAC Handshake Entry: SBC*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `SBC 44.1 kHz / 16-bit`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- SBC handshake fully blocks Samsung override injection.
- BCC transitions to target LDAC profile.
- Self-healing remains active for any desync correction.

---

## 🔧 4️⃣ AAC Auto Switch Logic  
*Class 2.2 — Non-LDAC Handshake Entry: AAC*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `AAC 44.1 kHz / 16-bit`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- AAC handshake prevents Samsung override injection.
- BCC transitions to LDAC `44.1 / 24 / 909`.
- AutoNotification monitors for correction as needed.

---

## 🔧 5️⃣ aptX Auto Switch Logic  
*Class 2.3 — Non-LDAC Handshake Entry: aptX*

**Setup:**
- BCC Auto Switch: Enabled
- Intermediate Profile: `aptX 44.1 kHz / 16-bit`
- Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
- Execution Delay: `0ms` (both Intermediate and Auto Switch)
- 2-Step: Disabled
- AutoNotification: Monitors `"Default"` and `"Adaptive"`
- Tasker: Regenerates to `44.1 / 24 / 909` if needed

**Behavior:**
- aptX handshake prevents override injection.
- BCC transitions to LDAC `44.1 / 24 / 909`.
- Self-healing engine corrects any GUI desync automatically.

---





## 🔧 Universal Self-Healing Logic (AutoNotification + Tasker Logic)

| Trigger Condition | Healing Action |
|--------------------|----------------|
| Sample Rate ≠ 44.1 kHz | Apply `44.1 kHz / 24-bit / 909` |
| Bit Depth ≠ 24-bit | Apply `44.1 kHz / 24-bit / 909` |
| Bitrate = Default (303/606/etc) **AND Codec = LDAC** | GUI Sync Only |
| Bitrate ≠ 909 (when LDAC active) | Apply Bit Depth Regeneration |
| Codec Change Event (non-LDAC → LDAC or intra-LDAC renegotiation) | Samsung Override Fully Displaced |
| Otherwise | No Action |





# LDAC Auto-Correction Logic — Final Model

## 🔧 Setup Context

- **Target Profile:** `44.1 kHz / 24-bit / 909 kbps`
- **Components:**
  - BCC Auto Switch (`0ms` execution delay)
  - Intermediate Profile: `96 kHz / 32-bit / 990 kbps`
  - Auto Switch Profile: `44.1 kHz / 24-bit / 909 kbps`
  - AutoNotification profile triggers
  - 2-Step: Off

---

## 🗒️ Bluetooth Codec Changer (BCC) Internal Logs

- BCC maintains **internal logs** that record profile switching events, codec negotiation attempts, and any applied self-healing triggers.
- These logs are extremely useful for:
  - Verifying Tasker automation correctness.
  - Debugging profile application order.
  - Detecting unexpected renegotiation events.
- Logs can be accessed directly from within BCC under the `Logs` or `Debug` section (depending on version).
- Ideal for verifying whether intermediate profile chaining or override defeats executed as intended.


# 🎯 Tasker Profile → Task Mapping (Samsung_Override.prj.xml)

This mapping details each Tasker task by ID and name, its intended purpose, and the list of profiles that trigger it.

---

### 🔧 Task ID: 102 — `Always Works Unless Quality Mismatch`
**Purpose:** Main fallback handler for successful BCC codec switches; used when quality is acceptable.

**Linked Profiles:**
- AAC_44100_16
- SBC_44100_16
- LDAC_44100_16_909
- LDAC_44100_16_Default
- LDAC_44100_24_303
- LDAC_44100_24_606
- LDAC_44100_24_909
- LDAC_44100_24_Default
- LDAC_44100_32_303
- LDAC_44100_32_606
- LDAC_44100_32_909
- LDAC_48000_16_330
- LDAC_48000_16_660
- LDAC_48000_16_990
- LDAC_48000_16_Default
- LDAC_48000_24_330
- LDAC_48000_24_660
- LDAC_48000_24_990
- LDAC_48000_24_Default
- LDAC_48000_32_330
- LDAC_48000_32_660
- LDAC_48000_32_990
- LDAC_48000_32_Default
- LDAC_88200_16_909
- LDAC_88200_24_303
- LDAC_88200_24_606
- LDAC_88200_24_909
- LDAC_88200_32_303
- LDAC_88200_32_606
- LDAC_88200_32_909
- LDAC_88200_32_Default
- LDAC_96000_16_330
- LDAC_96000_16_660
- LDAC_96000_16_990
- LDAC_96000_24_330
- LDAC_96000_24_660
- LDAC_96000_24_990
- LDAC_96000_24_Default
- LDAC_96000_32_330
- LDAC_96000_32_660
- LDAC_96000_32_Default

---

### ⚠️ Task ID: 23 — `Bit Depth Regeneration`
**Purpose:** Adaptive codec detection and override mismatch correction.

**Linked Profiles:**
- LDAC_44100_16_303
- LDAC_44100_16_606
- LDAC_44100_16_ADAPTIVE
- LDAC_44100_24_ADAPTIVE
- LDAC_44100_32_ADAPTIVE
- LDAC_48000_16_ADAPTIVE
- LDAC_48000_24_ADAPTIVE
- LDAC_48000_32_ADAPTIVE
- LDAC_88200_16_303
- LDAC_88200_16_606
- LDAC_88200_16_ADAPTIVE
- LDAC_88200_24_ADAPTIVE
- LDAC_88200_32_ADAPTIVE
- LDAC_96000_16_ADAPTIVE
- LDAC_96000_24_ADAPTIVE
- LDAC_96000_32_ADAPTIVE

---

### 🧪 Task ID: 17 — `Sbc Override`
**Purpose:** Force SBC handshake to reset Samsung override authority.

**Linked Profile:**
- LDAC_96000_32_990

---

### 🔍 Task ID: 15 — `Ldac 660`
**Purpose:** for apps that need bleutooth and wifi scanning which is unstable at 990 or 909 so switched back t0 606 or 909

**Linked Profile:**
- Bbc Maps

---

### 🎧 Task ID: 3 — `Location Off`
**Purpose:** Likely toggles environment prep or scanning off after connection.

**Linked Profiles:**
- Xm3 Connected
- Xm5 Connected

---

### 🔌 Task ID: 5 — `Location On`
**Purpose:** Environment or scanning restore after disconnect.

**Linked Profiles:**
- Xm3 Disconnect
- Xm5 Disconnect

---

### 🎶 Task ID: 74 — `Enable Profiles After High Res`
**Purpose:** Resume BCC switching after closing Qobuz.

**Linked Profile:**
- Qobuz Enable Bcc Profiles

---

### 🎛 Task ID: 73 — `Disable Profiles For High Res`
**Purpose:** Prevent profile switching during UAPP playback.

**Linked Profile:**
- UAPP Disable Bcc Profiles

## 📲 AutoNotification Integration

This Tasker project contains fully integrated **AutoNotification Intercept profiles** that:
- Monitor Bluetooth codec notifications (LDAC, SBC, APTX, AAC)
- Automatically trigger codec override tasks like:
  - `Sbc Override`
  - `Bit Depth Regeneration`
  - `Ldac 660`

These are embedded inside the Tasker `.prj.xml` using:
- `<plugintypeid>com.joaomgcd.autonotification.intent.IntentInterceptNotification</plugintypeid>`
- And are activated as soon as AutoNotification is installed and allowed notification access.




# 🧬 Tasker ↔ BCC Integration Contract

This section defines how **Tasker and Bluetooth Codec Changer (BCC)** work together to control LDAC codec switching.  
All codec control in this system is done by **Tasker broadcasting BCC profile names**, and **every name must exist inside BCC's `.json` profile files** for the override logic to work.

---

## ✅ Profile Naming Convention

Each BCC profile used by Tasker follows this format:

```
LDAC_<SAMPLE_RATE>_<BIT_DEPTH>_<MODE>
```

- **SAMPLE_RATE**: `44100`, `48000`, `88200`, `96000`
- **BIT_DEPTH**: `16`, `24`, `32`
- **MODE**:
  - `303`, `606`, `909`, `990` — Fixed bitrate
  - `ADAPTIVE` — Adaptive bitrate
  - `DEFAULT` — System-decided fallback
  - `SBC_DEFAULT` — Special fallback used to trigger SBC handshake

---

## 📁 BCC Profile File Location

All profiles must be defined inside files like:

```
Bluetooth Codec Changer/profiles/tested/WH-1000XM5_AV_OFF.json
Bluetooth Codec Changer/profiles/tested/WH-1000XM5_AV_ON.json
Bluetooth Codec Changer/profiles/tested/WH-1000XM3_AV_OFF.json
...
```

These files must contain a `profiles` array with matching `"name"` fields.

---

## 📡 Tasker-to-BCC Broadcast Logic

Every codec switch in Tasker uses:

```
Intent:
  com.amrg.bluetooth_codec_converter.REQUEST_PROFILE_SWITCH

Extra:
  com.amrg.bluetooth_codec_converter.extra.PROFILE_NAME:<EXACT_NAME>
```

Tasker tasks like `Sbc Override`, `Bit Depth Regeneration`, and `Always Works Unless Quality Mismatch` all broadcast specific `PROFILE_NAME` values to BCC.

---

## ✅ Profile Usage Matrix

| Tasker Task                          | BCC Profile Name            | Must Exist in BCC? | Purpose                            |
|--------------------------------------|------------------------------|---------------------|-------------------------------------|
| `Sbc Override`                       | `SBC_DEFAULT`                | ✅                  | Triggers SBC handshake reset        |
| `Bit Depth Regeneration`            | `LDAC_44100_16_606`          | ✅                  | Forces low-bit LDAC profile to retrain firmware |
| `Always Works Unless Mismatch`      | `LDAC_44100_24_909`          | ✅                  | Applies high-quality LDAC profile   |
| `Ldac 660`                           | `LDAC_44100_24_660` (optional) | ✅                | Handles fallback from AAC/APTX      |
| `AV_OFF Recovery`                   | `LDAC_44100_16_ADAPTIVE`     | ✅                  | Ensures override bypass with AV OFF |
| `Default Detection Recovery`        | `LDAC_44100_24_DEFAULT`      | ✅                  | Handles fallback if system returns to Default |

---

## ⚠️ Enforcement Rule

> **If a Tasker task references a profile that does not exist in BCC, the codec switch fails silently.**

To ensure proper operation:
- Every profile name used in Tasker **must exist** in BCC
- Tasker assumes BCC has full authority and correct codec access
- This allows complete override of Developer Options, Samsung behavior, and Fast Pair profiles

---

## 🔁 Practical Tip

To generate or sync profiles:
- Create them manually in BCC using its UI
- Or export from a working config and import on new devices
- Match Tasker broadcast values 1:1 with `"name"` fields in `.json`


## 🔍 AutoNotification-Based Codec Validation — Canonical Method

This Tasker system validates codec switching **only via BCC’s own notification**, not guessed system state.

- BCC's notification reflects the **actual active codec profile**.
- AutoNotification intercepts this to confirm:
  - Codec type (LDAC)
  - Sample rate (e.g., 44.1 kHz)
  - Bit depth (e.g., 24-bit)
  - Bitrate/quality (e.g., 909 kbps)
- This is **the authoritative state** after any renegotiation.

### Why This Is Superior:
- ✅ No ADB or dumpsys required
- ✅ Reflects final applied state, not just intent
- ✅ Works post-handshake and after fallback recovery
- ⚠ Only inaccurate if **2-Step is visually enabled but functionally broken**, which is handled by fallback logic (Class 3.x)

### Logic Summary:
- If profile matches `LDAC_44100_24_909` → Success
- Else → Trigger fallback logic or retry (e.g., Class 2.2 → 3.2)

This is the **only trusted runtime verification layer** and forms the core of Tasker’s self-healing override strategy.





## 🔍 AutoNotification-Based Codec Validation — Canonical Method

This Tasker system validates codec switching **only via BCC’s own notification**, not guessed system state.

- BCC's notification reflects the **actual active codec profile**.
- AutoNotification intercepts this to confirm:
  - Codec type (LDAC)
  - Sample rate (e.g., 44.1 kHz)
  - Bit depth (e.g., 24-bit)
  - Bitrate/quality (e.g., 909 kbps)
- This is **the authoritative state** after any renegotiation.

### Why This Is Superior:
- ✅ No ADB or dumpsys required
- ✅ Reflects final applied state, not just intent
- ✅ Works post-handshake and after fallback recovery
- ⚠ Only inaccurate if **2-Step is visually enabled but functionally broken**, which is handled by fallback logic (Class 3.x)

### Logic Summary:
- If profile matches `LDAC_44100_24_909` → Success
- Else → Trigger fallback logic or retry (e.g., Class 2.2 → 3.2)

This is the **only trusted runtime verification layer** and forms the core of Tasker’s self-healing override strategy.
