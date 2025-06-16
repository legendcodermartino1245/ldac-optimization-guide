##  Multipoint LDAC Overview

The **Sony WH-1000XM5** supports **multipoint Bluetooth**, allowing connection to **two devices simultaneously**.

Typically, this works seamlessly because:
- **Windows** defaults to **SBC or AAC**
- **Android** uses **LDAC**

 This prevents codec conflicts.

---

However, **advanced users** may configure **LDAC on both devices** — for example:
- Android **and** Windows  
- Or even **two Android devices**

###  This introduces a challenge:
> **Can we build a stable configuration where both devices use LDAC without triggering codec fallback, stutters, or renegotiation?**

A stable dual-LDAC setup requires careful tuning of:
- Codec behavior  
- Volume and AVRCP versions  
- Playback format

---

##  Setup Prerequisites

To ensure a smooth and stable LDAC multipoint experience:

###  AVRCP 1.6 on Android

- **On Android:**
  - Enable AVRCP 1.6 once in Developer Options
  - **It persists even after Developer Options are turned off**
##  LDAC Multipoint Stability Factors

| Factor                       | Configuration Options                                               | Related Notes or Interactions                                      |
|------------------------------|---------------------------------------------------------------------|--------------------------------------------------------------------|
| Codec settings               | Mirror **LDAC settings** on both devices                            | Prevents codec renegotiation or fallback to SBC/AAC                |
| Playback format              | Match **sample rate** and **bit depth** (e.g., 96 kHz / 24-bit)     | Avoids stutters and DSP resync delays                              |
| Absolute Volume              | Use **ON on Windows**, **OFF on Android**                           | Maintains volume sync on Windows and codec control on Android      |
| Playback state coordination  | **Pause playback** on the inactive device                           | Prevents LDAC renegotiation and session fight during handoff       |
| BCC timing logic             | Add **intermediate profile** (e.g., SBC or LDAC 16-bit) before final LDAC | Ensures clean codec handshake and profile lock-in          |
| AVRCP version                | Use **1.6 on Android**, **1.6 on Windows**                          | 1.5 on Windows avoids playback interruptions; 1.6 on Android keeps fast metadata and control |


>  **Note:** Switching from Absolute Volume OFF to ON (or vice versa) on **Windows** may require **re-pairing** the headphones for the setting to take full effect.

>  **Implementation details** of AVRCP 1.6 may vary between platforms. All testing was done using AVRCP **1.6**.

---

##  Multipoint AV/AVRCP Combination Matrix (LDAC on Both Devices)

###  Legend
- **AV = Absolute Volume**
- **AVRCP = Audio/Video Remote Control Profile**
-  = Confirmed
-  = Untested
- **CT** = AVRCP Controller
- **TG** = AVRCP Target
- **Alt Driver** = Alternative A2DP Driver (BluetoothGoodies)

>  CT/TG roles are not visible in Sound Connect. Must be inferred via:
> - Metadata to Android
> - Headset button control
> - Pause/resume behavior


---

##  Multipoint Engineering Companion

---

###  Multipoint Initialization Sequence

| Step | Description | Notes |
|------|-------------|-------|
| 1⃣ | A2DP profile established | LDAC (Fixed or Adaptive), AAC, SBC |
| 2⃣ | AVRCP role negotiation | CT/TG roles assigned per device |
| 3⃣ | Absolute Volume negotiation | AV ON/OFF per device |
| 4⃣ | Metadata control allocation | Only 1 CT allowed at a time |
| 5⃣ | Resume priority logic | Active device preferred for media switching |
| 6⃣ | Adaptive bitrate negotiation | Adaptive LDAC impacts bitrate only, not control plane |

---

###  Multipoint Control Conflict Matrix

| Conflict | Root Cause | Solution | Status |
|----------|-------------|----------|--------|
|  Buttons fail | CT role retained on inactive device | AVRCP mismatch (1.6 sensitive) |  Fully Documented |
| Metadata missing | AVRCP CT conflict | Use matched AVRCP versions |  Fully Documented |
| Resume stutter | AVRCP negotiation spikes | AVRCP 1.5 on Windows / AV OFF on Android |  Fully Documented |
| Playback switching hangs | CT role not reassigned cleanly | Pause inactive device before switching |  Fully Documented |
| Unexpected full pause | CT handover not atomic | Manual pause preferred |  Fully Documented |
| Mic fallback | A2DP ↔ SCO handover | Spec behavior |  Fully Documented |

---

###  Minimal Multipoint Validation Matrix

| Test | Devices | Focus |
|------|---------|-------|
| CT/TG role testing | Android ↔ Windows | AVRCP 1.5 vs 1.6 |
| AV sync testing | Android + Windows | Absolute Volume ON/OFF |
| Metadata sync | Android ↔ Windows | AVRCP role assignment |
| Adaptive behavior | Android ↔ Android | Bitrate stability |
| Resume/Unlock testing | Android ↔ Windows | Stutter risk |

 After validating these, all multipoint behaviors become fully predictable.

---

###  Adaptive LDAC Exception Handling

- Adaptive mode only affects bitrate, not control plane.
- No extra multipoint logic required.
- Instabilities:
  - 44.1kHz Adaptive →  unstable
  - 48kHz & 96kHz Adaptive →  stable
- All control plane behaviors follow Fixed mode logic.

---

###  Codec Pair Summary Matrix

| Android Codec | Windows Codec | Behavior Class | Notes |
|----------------|----------------|----------------|-------|
| LDAC ↔ LDAC |  Fully tested |  10/10 stable (Fixed) |
| LDAC ↔ SBC |  Stable resume | SBC fallback on Windows |
| LDAC ↔ AAC |  Medium resume | Mic fallback risk |
| LDAC ↔ aptX |  Stable | Playback only |
| AAC ↔ SBC |  CT flip risk | Profile swap under voice triggers |
| AAC ↔ AAC |  Stable | No mic support |
| AAC ↔ aptX |  Stable | Playback only |
| SBC ↔ SBC |  Stable | Lowest denominator |
| SBC ↔ AAC |  Stable | Playback stable |
| SBC ↔ aptX |  Stable | Playback stable |

---

###  Multipoint Troubleshooting Flowchart

```text
[ Troubleshooting Logic]

IF  Buttons Fail →
  ↳ Check AVRCP CT role (SoundConnect / dumpsys)
  ↳ AVRCP 1.6? → Downgrade to 1.5 on Windows
  ↳ AV mismatch? → Reconnect devices in reverse order

IF  Metadata Missing →
  ↳ AVRCP CT conflict → Symmetrize AVRCP versions

IF  Resume Stutter →
  ↳ Android: AV OFF
  ↳ Windows: AVRCP 1.5
  ↳ Avoid unlock stutter during active playback

IF  Mic Issues →
  ↳ SCO fallback: A2DP spec behavior, not a bug

IF  Adaptive Instability →
  ↳ Use LDAC Fixed 48kHz/96kHz for stability
```
###  TLDR

- All control plane instability lives in AV + AVRCP + CT/TG negotiation.
- You already fully documented every conflict driver.
- Further full matrix expansion adds zero new knowledge.
- Troubleshooting layer allows anyone to diagnose any multipoint issue using:
  - AVRCP role visibility
  - Absolute Volume state
  - Resume priority logic



##  Windows 11 – Alternative A2DP Driver

| #   | Device A | Device B   | AV (A / B) | AVRCP (A / B) | CT Role                   |  Buttons |  Meta |  Stutter |
|-----|----------|------------|------------|----------------|----------------------------|------------|---------|------------|
| 1   | Android  | Windows 11 | ON / ON    | 1.6 / 1.6      |  CT + TG (inferred)      |          |        |          |
| 2   | Android  | Windows 11 | OFF / ON   | 1.6 / 1.6      |  CT + TG (inferred)      |          |        |          |
| 2   | Android  | Windows 11 | ON / OFF   | 1.6 / 1.6      |  CT + TG (inferred)      |          |        |          |
| 4   | Android  | Windows 11 | OFF / OFF  | 1.6 / 1.6      |  CT + TG (inferred)      |          |        |          |






##  AVRCP Role Detection — Testing Methodology

To determine AVRCP Controller (CT) and Target (TG) roles during multipoint LDAC operation, the following non-invasive test procedure was used:

###  Test Signals Used

| Signal | Purpose | Role Inference |
|--------|---------|----------------|
|  **Headset Play/Pause/Next Buttons** | Determine which device accepts AVRCP control from headset | Confirms AVRCP Controller (CT) assignment |
|  **Metadata Display (Sound Connect Android)** | Identify which device is supplying metadata to the headset | Confirms AVRCP Target (TG) assignment |
|  **Playback Status with Headset Controls** | Observe which device resumes playback upon button press | Verifies active AVRCP CT priority |

###  Tools Used

- **Sound Connect (Samsung) — Android App**  
  - For live metadata transfer and AVRCP role monitoring.

- **Headset Hardware Buttons**  
  - Play / Pause / Next buttons used for active control arbitration.

###  No additional debugging tools used:

- No Bluetooth sniffers.
- No root-level packet inspection.
- No developer debug logs.
- All tests performed under real-world operating conditions.


###  AVRCP Role Validation Summary

-  Headphone correctly handles play/pause/next → TG role proven.
-  Android updates metadata instantly while not owning audio stream → CT role remains active even in passive multipoint state.
-  Windows actively controls media when streaming → CT role stable on Windows.
-  Multipoint switching between Windows and Android triggers expected resume behavior → CT↔TG transitions healthy.
-  No metadata desync or stale data observed → AVRCP control channel integrity verified.


---

 This method allows fully sufficient AVRCP CT/TG role detection for the LDAC multipoint protocol matrix.




#  Alternative A2DP Driver – Android + Windows LDAC Multipoint Notes

##  Installation Steps

1. **Buy license from:** [https://www.bluetoothgoodies.com/a2dp/](https://www.bluetoothgoodies.com/a2dp/)
2. **Install the stable version of the software**
3. **Configure the following:**
   - Install the Alternative A2DP Driver
4. **Optionally reboot after**

---

##  Optimal LDAC Settings (Windows A2DP Driver)

###  Media Focus / CD Quality (for apps with exclusive access via WASAPI)

- **Sample Rate:** 44.1 kHz or 48 kHz  
- **Bit Depth:** 24 bit  
- **Stereo Mode:** Stereo  
- **Encode Quality:** High (HQ)  
- **Adaptive Bitrate:** Disabled

---

###  High-Res Playback (for apps with exclusive access via WASAPI)

- **Sample Rate:** 44.1 / 48 / 88.2 / 96 kHz  
- **Bit Depth:** 24-bit  
- **Stereo Mode:** Stereo  
- **Encode Quality:** High (HQ)  
- **Adaptive Bitrate:** Disabled

---

###  Bit Depth-Only Resampling (for high-res playback in exclusive mode)

- **Only get depth rate resampling when playing high res**
- Ensure exclusive mode is ON
- Useful for apps that properly handle bit-perfect streams

---

##  Spotify Specific Behavior

- **Spotify doesn’t support WASAPI**
- Use 44.1 kHz, 24 bit LDAC profile
- Avoid enabling exclusive mode or adaptive bitrate
- Stick to media-optimized LDAC profile

---

##  Windows Audio: Clean Output Settings for LDAC Multipoint

To ensure **LDAC 990 kbps stability**, **bit-perfect playback**, and **multipoint performance** without dropouts or codec renegotiation, configure your **Windows audio environment** using the following settings.

---

###  Disable Audio Enhancements

Prevents unwanted DSP (bass boost, loudness EQ, virtualization) from altering the signal before LDAC encoding.

**Steps:**
1. Open `Control Panel → Sound → Playback`
2. Right-click your **Bluetooth headphones** → **Properties**
3. Go to the **Enhancements** tab
4.  Check **“Disable all enhancements”**

>  If there's no Enhancements tab, go to **Advanced** instead and disable “Enable audio enhancements” if present.

---

###  Disable Spatial Audio

**Spatial Sound** adds unnecessary processing and may cause LDAC instability or resampling.

**Steps:**
1. Right-click the  **speaker icon** → **Sound settings**
2. Under your Bluetooth output, click **Properties**
3. Scroll to **Spatial Sound**
4. Set to **Off**

---

###  Disable “Give Exclusive Mode Applications Priority”

This specific setting causes **LDAC session drops** and multipoint instability — especially with AVRCP 1.6.

**Steps:**
1. Open `Control Panel → Sound → Playback`
2. Right-click your **Bluetooth LDAC device** → **Properties → Advanced tab**
3. **Uncheck only**:
   -  **“Give exclusive mode applications priority”**

>  You may leave **“Allow applications to take exclusive control of this device”** **enabled** if using bit-perfect apps like **Roon** or **Neutron**.

---

###  Use Exclusive Mode *Only* in Audio Players

Leave system output non-exclusive to avoid interference.  
Enable exclusive mode **only** inside apps like:

- UAPP: `Hi-Res Direct Driver → Bit-Perfect Mode → ON`
- Roon: `Exclusive Audio Access → ON` for WASAPI output

>  This isolates the audio stream for direct LDAC delivery, bypassing Android-style mixers.

---

###  Re-Apply After Every Re-Pair

>  **All Windows audio settings above must be manually re-applied after every Bluetooth re-pair.**

When you re-pair LDAC headphones, Windows assigns a new audio device instance, which:
- Resets **audio enhancements**, **spatial sound**, and **exclusive mode priority** settings
- May label the device the same in name, but internally treats it as a **new endpoint**

**Always revisit `Control Panel → Sound → Playback` and reconfigure all settings** for the newly paired instance to maintain optimal LDAC performance and multipoint stability.

##  AVRCP Behavior and Multipoint Issues

- **AVRCP 1.6 on both Windows 11 and Android causes stuttering**
  - Stutter occurs shortly after unlocking the phone
- Even if **Windows is playing and using AVRCP 1.6 by itself**,  
  it can stutter when Android is connected but idle — just because multipoint is active.
- **Windows randomly pauses without reason** during multipoint playback.
- **With AVRCP 1.5 on Windows**, Android still sees Windows as playing audio.
- **More stuttering occurs when unlocking the phone** if AVRCP is switched to 1.5.

---

###  Do Not Use Registry AVRCP Edits

Modifying `AvrcpTargetVersion` in the Windows Registry has no effect on AVRCP behavior with modern stacks (including Bluetooth Goodies). Windows 11 and 10 always uses AVRCP 1.4–1.6, and this cannot be changed manually. Metadata visibility and switching latency should be used to infer behavior instead.
##  Multipoint + LDAC Dual Control Behavior

- With AVRCP 1.6 on both devices:
  - You can **press play on both Android and Windows**
  - Audio from both can **attempt to play at the same time**
  - If **Android is the active source**, stuttering is minimal
  - If **Windows is the active source**, Android stops playback

- Even **Windows randomly pauses** without any user interaction during multipoint
- Android **still sees that Windows is playing audio**, even if Windows uses AVRCP 1.5

---

###  Controlling AVRCP Version? You Can’t — Unless You Change Hardware

The **only reliable way** to influence the AVRCP version used in **Windows** is to **buy a Bluetooth adapter with a specific Bluetooth version**.

- Windows **ignores** the `AvrcpTargetVersion` registry key on all modern Bluetooth stacks (including the Bluetooth Goodies A2DP driver).
- You **cannot downgrade or upgrade AVRCP** via software alone — the stack negotiates the **highest mutually supported version** between the dongle and the device.



#  AVRCP Version Support Matrix and OS Behavior

This section explains how **AVRCP (Audio/Video Remote Control Profile)** version support varies by **Bluetooth version**, **Windows version**, and **Android** — along with which system factors actually control what you get in practice.

---

##  Bluetooth Version vs AVRCP Compatibility (with OS Notes)

| Bluetooth Version | Default AVRCP Version(s) | Windows 10 Support         | Windows 11 Support (22H2+) | Android Support               |
|-------------------|---------------------------|-----------------------------|-----------------------------|-------------------------------|
| 2.0 + EDR         | 1.0–1.3                   |  No usable AVRCP          |  No usable AVRCP          |  Legacy only                |
| 2.1 + EDR         | 1.3–1.4                   |  AVRCP 1.3–1.4            |  AVRCP 1.4                |  AVRCP 1.4                  |
| 3.0 + HS          | 1.4                       |  AVRCP 1.4                |  AVRCP 1.4                |  AVRCP 1.4                  |
| 4.0               | 1.4–1.5                   |  AVRCP 1.4                |  AVRCP 1.5                |  AVRCP 1.5                  |
| 4.1               | 1.5                       |  AVRCP 1.5                |  AVRCP 1.5                |  AVRCP 1.5                  |
| 4.2               | 1.5                       |  AVRCP 1.5                |  AVRCP 1.5                |  AVRCP 1.5                  |
| 5.0               | 1.6                       |  AVRCP 1.5                |  AVRCP 1.6                |  1.5 default, 1.6 supported  |
| 5.1               | 1.6                       |  AVRCP 1.5                |  AVRCP 1.6                |  1.5 default, 1.6 supported  |
| 5.2               | 1.6                       |  AVRCP 1.5                |  AVRCP 1.6                |  1.5 default, 1.6 supported  |
| 5.3               | 1.6                       |  AVRCP 1.5                |  AVRCP 1.6                |  1.5 default, 1.6 supported  |
| 5.4               | 1.6                       |  AVRCP 1.5                |  AVRCP 1.6                |  1.5 default, 1.6 supported  |

---

###  Notes

-  **Windows 10**
  - AVRCP 1.3: Basic metadata (track title, artist)
  - AVRCP 1.4: Media browsing and playback status
  - AVRCP 1.5: From version 1803 onward
  -  AVRCP 1.6: Not implemented in any version

-  **Windows 11**
  - AVRCP 1.5 in 21H2
  - AVRCP 1.6 starting from **22H2**
  - Maintained in 23H2 and 24H2

-  **Android**
  - AVRCP **1.5 is the default** even in Android 10+
  - **AVRCP 1.6 is supported** from Android 10 onward
  - OEMs like **Pixel, Samsung, OnePlus** often enable 1.6 via stack config override

---

##  How AVRCP Support Is Determined

> AVRCP version support depends **first on the OS's Bluetooth stack**, then on the capabilities of the Bluetooth adapter and its driver.

###  Priority of Influence

| Priority | Factor                       | Why It Matters                                                                 |
|----------|------------------------------|---------------------------------------------------------------------------------|
| **1**    | **Bluetooth Stack (OS-level)** | Sets the **maximum AVRCP version** available (e.g. 1.5 on Win10, 1.6 on Win11) |
| **2**    | **Bluetooth Adapter Version** | Determines **base protocol support**, but doesn’t control AVRCP version alone  |
| **3**    | **Driver / Stack Vendor**     | Some drivers (Intel, CSR, Generic) limit or extend access to AVRCP features    |

---

###  Real-World Example

> A **Bluetooth 2.1 + EDR** adapter:
- On **Windows 10**: Limited to **AVRCP 1.5**
- On **Windows 11 (22H2+)**: Gains **AVRCP 1.6**, even with the **same adapter**, because the OS stack supports it

---

###  Summary Rule

> **AVRCP Version = MIN(Bluetooth Stack Capability, Adapter Driver Capability)**

So:
- A modern OS like Windows 11 can unlock AVRCP 1.6 on older adapters (if the driver allows)
- But a Bluetooth 5.4 dongle on Windows 10 will still be limited to AVRCP 1.5

---

##  Windows 10 AVRCP Version Timeline

| Windows 10 Version     | AVRCP Version | Changes Introduced                                                                 |
|------------------------|----------------|--------------------------------------------------------------------------------------|
| **1507 – 1709**         | 1.3–1.4         | Basic metadata and media browsing support                                           |
| **1803 (April 2018)**   |  **1.5**       |  Full AVRCP 1.5 support:  
- Absolute volume  
- `SetAddressedPlayer`  
- Better headset media control |
| **1903 – 21H2**         | 1.5             | No AVRCP upgrades — only Bluetooth stack refinements                                |

---

##  Windows 11 AVRCP Version Timeline

| Windows 11 Version     | AVRCP Version | Changes Introduced                                                                 |
|------------------------|----------------|--------------------------------------------------------------------------------------|
| **21H2 (Initial release)** | 1.5             | Inherited Windows 10 stack — no AVRCP 1.6                                           |
| **22H2 (2022 Update)**     |  **1.6**       |  Full AVRCP 1.6 support:  
- Two-way metadata sync  
- Headset ↔ PC control  
- Player application settings |
| **23H2 and 24H2**          | 1.6             | Maintains AVRCP 1.6 — driver and stability improvements only                        |

---

##  Android AVRCP Behavior by Version

| Android Version | Default AVRCP | AVRCP 1.6 Support | Notes                                                                 |
|------------------|----------------|--------------------|------------------------------------------------------------------------|
| Android 8.0–9     | 1.5            |  Optional         | Some OEMs (e.g., Sony, Samsung) enabled 1.6 manually                   |
| Android 10+       | 1.5            |  Supported        | AVRCP 1.6 supported, but **1.5 still default** in AOSP config          |
| Android 12+       | OEM-dependent  |  Enabled by OEMs  | Most flagships (Pixel, Samsung, OnePlus) override default to 1.6      |

---

###  Why Android Defaults to AVRCP 1.5

Even in Android 10 and above, the Bluetooth stack in AOSP is configured to use **AVRCP 1.5 by default**.

This setting is defined internally in the system configuration file (`bluetooth_stack.conf`). Unless the device manufacturer (like Samsung, Google, or OnePlus) explicitly overrides this, the system will continue using AVRCP 1.5 — even though AVRCP 1.6 is fully supported by the platform.



>  **Note**: AVRCP negotiation is unidirectional — **the lower of the two devices' supported versions wins**.

##  LDAC Multipoint: What Actually Needs to Match using AVCRP 1.5 

| Parameter                                 | Must Match? |
|-------------------------------------------|-------------|
| **Codec Type (must be LDAC)**             |  Yes      |
| **Absolute Volume Setting**               |  No*      |
| **Bluetooth Connection Quality**          |  No*      |
| **LDAC Mode (Fixed vs Adaptive)**         |  No       |
| **LDAC Bitrate (e.g. 990 / 660 / 330)**   |  No       |
| **LDAC Bit Depth (16 / 24-bit)**          |  No       |
| **LDAC Sample Rate (44.1 / 48 / 96 kHz)** |  No       |
| **Source Media Format (FLAC, MP3, etc.)** |  No       |
| **Original Sample Rate / Bit Depth**      |  No       |
| **AVRCP Version (1.5 vs 1.6)**            |  No       |
| **Operating System (Android / Windows)**  |  No       |
| **Audio App (UAPP, Neutron, etc.)**       |  No       |
---
\* Absolute Volume **doesnt need to match** for codec compatibility, but aligning them can improve **connection stability** and reduce **stutter risk** in edge cases.
---

##  LDAC Control Roles

> With correct setup:
> - **Windows acts as the dominant LDAC controller**
> - **Android behaves as an intelligent follower**
> - **UAPP on Android** yields to Windows even when playing in high-res exclusive mode

This behavior has been validated across:
- Android 13 / 14 (One UI variants)
- Windows 11 with AVRCP 1.5/1.6
- LDAC fixed and adaptive configurations (mirrored)

---

##  LDAC Multipoint Confirmation

> Everything works in multi-control.  
> All combinations of **fixed and adaptive** LDAC modes are valid  
> **as long as both sides are mirrored.**

- Do **not use 32-bit on Android**
  - It adds **no audible benefit**
  - It introduces instability in codec negotiation

---

##  Multipoint Works "As Intended" — But Isn't

> LDAC Multipoint is **supported** — and it works.

- Android and Windows can both stream audio over LDAC
- You can **press play on both**, and both streams will try to take control
- Audio stability depends on who initiates playback
- This works despite not being **intended behavior** by most manufacturers

---

##  Final Test Results

- When **Absolute Volume is ON**, and **Windows is playing**:
  - You can still **press play on Android**
  - Playback from Android stops silently
  - No override happens unless Android is prioritized first
- **Very little stuttering** when Android plays while Windows is still active
- **Perfect behavior when Android is the audio source and Windows is idle**

> “It only stutters **very little**”

---

##  Final Takeaways

-  True **multi-control LDAC** is possible between Android and Windows
-  Works best when **Android is the primary source**
-  **Windows should use AVRCP 1.5**, but 1.6 works with more care
-  **Absolute Volume ON (Windows)** / **OFF (Android)** gives best sync
-  Android **intelligently yields** if Windows takes codec control
-  Pressing play on both works — **if Android is the one actually playing**
-  Mirrored profiles (fixed/adaptive) = most stable
-  Avoid 32-bit audio on Android — unnecessary and buggy


###  Windows + Android Multipoint Tips (LDAC / Bluetooth)

####  View Codec Info via Android's Sound Assistant or Sound Connect While Playing from Windows
When using **multipoint with Windows as the active audio source**, you can still:
- Open **Sound Assistant** or **Sound Connect** on your Android device  
- See the **active Bluetooth audio codec** (e.g. LDAC)  
- Monitor connection state, device battery, and metadata (e.g. track title)

 **Why this works**: Android stays connected over control and data channels, even if it’s not playing audio. This allows apps like Sound Connect to report codec info live.

####  Fix Device Selection in Sound Connect During Multipoint
While LDAC multipoint is active:
- Open **Sound Connect** on Android  
- **Pin** or **lock** the headphones to Android within the app  
- This ensures consistent codec visibility and device control

>  Useful when switching sources frequently — it prevents Android from deprioritizing the device in the UI or misreporting connection status.


>  **AVRCP 1.5 is more stable than 1.6 on Windows — period.**
>
> Using AVRCP 1.6 on Windows causes:
> - Random pauses
> - Playback interruptions
> - Stuttering during multipoint with Android
>
>  **AVRCP 1.5 avoids these issues** by keeping Windows passive in media control and respecting Android’s LDAC session.
>
>  **Note:** Short, *predictable stuttering* may still occur during device switching — but it **recovers instantly without codec fallback**.
>
>  To force AVRCP 1.5, use a **Bluetooth 4.2 (or lower)** dongle.
>  **Tip for Seamless Multipoint Switching (No Stutter)**
>
> To avoid stutters during LDAC multipoint handoff:
> - **Pause** playback on the currently active device
> - **Then play** from the other device
>
>  This ensures clean AVRCP session transfer  
>  Prevents renegotiation or fallback  
>  Maintains LDAC 990kbps without stutter
>
> Call Audio Always Takes Priority — But Doesn’t Affect LDAC Codec State
> LDAC multipoint never causes fallback, even across mismatched bitrates or sample rates — as long as both devices use LDAC and only one plays at a time.

##  AVRCP 1.5 Limitation on Windows: Headset Buttons Do Not Work

When **AVRCP 1.5** is used on Windows, **headset media buttons (play/pause/skip)** do **not** function to control playback on the PC.

###  Why?

- Windows acts only as a **passive target (follower)** under AVRCP 1.5
- The headset cannot send media control commands to the PC
- You can control **the headset from Windows**, but **not Windows from the headset**

---


##  Android Unlock Stutters? The Hidden Cost of AVRCP 1.6 With AV ON

If you're using **LDAC multipoint** (Android + Windows), and you notice a **brief stutter or glitch when unlocking your Android phone**, here's the reason:

>  **AVRCP 1.6 on Android broadcasts media session updates on unlock** — even if Windows isn’t actively playing.

This behavior can silently disrupt an active LDAC stream, especially at **990 kbps**, because:

- Android resends metadata, playback state, or volume
- Windows (still paired) may respond with stale AVRCP 1.6 info
- The Sony headphones renegotiate session state → **audio glitch**

###  Disabling “Remote Control” in Windows Doesn’t Help

You might think that disabling the **“Remote Control”** service in Windows (under `Bluetooth Services → Headphones`) would fix unlock-time stutters caused by **AVRCP 1.6**.

>  It doesn’t — and here’s why:

Even with Remote Control disabled:
- **Android still pushes MediaSession data via AVRCP 1.6**
- **Headphones still detect session conflict**
- **LDAC renegotiation still occurs**, triggering a brief audio drop

 This setting only prevents **Windows from sending AVRCP commands**,  
but it doesn’t stop the **broadcast loop initiated by Android.**

AV OFF on Android fully eliminates unlock stutters with AVRCP 1.6.


###  Why Format Matching Matters with AVRCP 1.6 Multipoint

When using **AVRCP 1.6** in a multipoint LDAC setup, **matching audio format (sample rate and bit depth)** across devices becomes essential for preventing stutters, glitches, and renegotiations.


##  AVRCP 1.6 Button Control Works — Even When Android Is the Active Source

Contrary to common belief, **headset media buttons (play/pause/skip)** still work on **Windows with AVRCP 1.6**, **even when Android is currently streaming LDAC audio**.

This proves AVRCP remains active and responsive on both connections during multipoint use — not just the one providing audio.

---

###  Behavior Summary

| Condition                         | Headset Button Works? | Behavior                                                                 |
|----------------------------------|------------------------|--------------------------------------------------------------------------|
| **Android streaming LDAC**       |  Yes                 | Button press on headset **controls Windows apps** (Spotify, YouTube, etc.) |
| **Windows not playing audio**    |  Yes                 | Buttons **wake** MediaSession on Windows — playback resumes              |
| **Both devices idle**            |  Yes                 | First button press **awakens one side** — whichever responds first wins |
| **Windows using AVRCP 1.5**      |  No                  | Headset buttons **do not register** — Windows acts as passive target     |

---

###  Technical Explanation

- **AVRCP 1.6 remains active**, even without an active A2DP stream
- **Sony WH-1000XM5** headset sends commands across **both Bluetooth control channels**
- **Windows listens** for media control events and resumes playback if a session exists
- **Android does not block AVRCP role** when streaming LDAC

>  **Absolute Volume OFF** on Android does **not interfere** with this behavior  
>  **AVRCP 1.6** is required — **AVRCP 1.5 disables button functionality**

---

###  Practical Result: True Multi-Control Multipoint

With proper configuration:

- **Android streams LDAC**  
- **Windows accepts media controls**  
- Both devices **stay connected and responsive**  
- You can press **play/pause/skip** on the headset at any time — no matter which device is active

---

###  Addendum for Guide

>  **With AVRCP 1.6 on Windows, headset media buttons still function even when Android is the active audio source.**  
> This confirms that **AVRCP control stays alive on both connections**, enabling **dual-device responsiveness** without requiring audio to be active on both ends.


###  Android Automatically Pauses When Windows Becomes Active

With AVRCP 1.6 and multipoint active:

- If **Windows starts playback** while **Android is actively streaming LDAC**,  
- **Android will pause automatically**, without requiring user input or media button presses.

This handoff is a result of **AVRCP media session priority negotiation** — not a bug.

>  Ensure Android has **Absolute Volume OFF** to prevent playback stalls or stutters during this transition.



##  Windows 11 – Full Two-Way AVRCP 1.6 Confirmed with WH-1000XM5

###  Device Pairing Flow (Test Setup)
- WH-1000XM5 connected first to **Windows 11** via Bluetooth
- Windows begins playing a known track (`Track A – Artist A`)
- **Android** connects *afterwards* (multipoint)
- Sony Headphones Connect initially shows **"Unknown song"**
- When the track is changed/skipped on **Windows**, metadata appears **instantly** in the Sony app

###  Interpretation
- **Metadata was not cached** on the headphones prior to Android connection
- **Windows must have sent the metadata**, because Android was not previously paired
- The **XM5 headset stored the AVRCP metadata**, which was later queried by Android
- This confirms that Windows is acting as an **AVRCP Controller** (CT)

###  Headset Button Test
- **Play/Pause**, **Next**, and **Previous** buttons on the WH-1000XM5 **control playback on Windows**
- Volume sync works with **Absolute Volume enabled**
- This confirms Windows is also acting as an **AVRCP Target** (TG)

---

###  Final Capability Matrix (Windows 11 + WH-1000XM5)

| Capability                              | Direction        | Role         | Status |
|-----------------------------------------|------------------|--------------|--------|
| Send metadata (title, artist, etc.)     | Windows → XM5    | Controller   |      |
| Receive media button input              | XM5 → Windows    | Target       |      |
| Volume synchronization                  | Bidirectional    | A2DP / AVRCP |      |
| Metadata visible in Sony Connect        | XM5 → Android    | Target       |      |
| Metadata sent on track change (not idle)| Windows → XM5    | Controller   |      |

---

###  Conclusion
>  Windows 11 does in fact support **AVRCP 1.6 bidirectionally**, including both:
> - **Metadata transmission** (as Controller)
> - **Playback control reception** (as Target)

Legacy claims that “Windows does not support AVRCP metadata” or "Windows is target-only" are now **outdated**.  
Modern builds of Windows 11 paired with the WH-1000XM5 demonstrate **fully working two-way AVRCP**, comparable to Android and Linux setups.



##  Play/Pause Behavior – Android vs Windows

Even with confirmed **two-way AVRCP 1.6** support, playback handling differs between platforms during multipoint use.

---

###  Observed Behavior

- When **Android connects** to **Windows** (which is already playing):  
   **Android auto-pauses** its playback immediately.

- When **Windows connects** to **Android** (which is already playing):  
   **Windows does not pause**, and both devices may play simultaneously.

---

###  Why This Happens

> **AVRCP 1.6 does not define playback arbitration.**  
> It provides:
> - Media controls (Play, Pause, Next, Previous)
> - Metadata exchange
> - Volume synchronization

Automatic pausing when a second device is active is a **platform-level feature**, not part of the AVRCP specification.


---

###  OS Playback Policy Comparison

| Scenario                                       | Android Behavior         | Windows Behavior         |
|-----------------------------------------------|--------------------------|--------------------------|
| Android connects to Windows (already playing) |  Auto-pauses Android    |  Continues playback     |
| Windows connects to Android (already playing) |  No auto-pause          |  Continues playback     |



###  Conclusion

- **AVRCP 1.6 two-way control is fully working** on both Android and Windows.
- **Android actively manages media sessions** and pauses itself to avoid conflict.
- **Windows lacks multipoint-aware session handling**, so playback continues.

>  **AVRCP 1.6 Two-Way Control Confirmed**  
> In multipoint mode, Android can issue play/pause/skip commands to Windows directly through the Sony Headphones Connect app, while Windows is the active audio source.  
> This confirms Windows fully acts as an AVRCP 1.6 Target, and Android as a Controller — beyond headset buttons alone.



matching sample rate bit depth and bit rate is important for 1.6 not for 1.5


##  Media Control Behavior Varies Across Apps

Not all Windows audio players respond to Bluetooth media controls in the same way. This is due to differences in how each app interacts with Windows’ **Global Media Transport Control** system.

###  Why This Matters

Apps that fully integrate with the system:

-  Respond reliably to headset play/pause/skip buttons
-  Broadcast metadata (track name, artist, etc.) over AVRCP
-  Resume playback seamlessly after interruptions

Apps that don’t integrate:

-  May ignore media buttons unless focused
-  Do not show metadata on headphones or connected devices
-  Require manual playback control and don’t resume automatically

---

###  LDAC Multipoint Implications

When using **LDAC multipoint** (Android + Windows):

-  Headset button behavior depends entirely on the media player’s system integration.
-  Even with stable LDAC 990 kbps audio, some apps may not resume or respond without user interaction.
-  Metadata may be missing from Sony Headphones Connect or Android if the app does not expose it.

---

###  Best Practice

> For reliable multipoint performance:
> - Use apps that integrate with Windows’ media transport session
> - Keep the app open and in use to maintain session visibility
> - If playback or media buttons don’t respond, it’s likely due to **app limitations**, not LDAC or Bluetooth issues



##  Does Android Auto-Resume if Windows Stops Playing?

>  No — Android does **not** automatically resume playback when Windows stops.

###  Observed Behavior

- When **Android is actively streaming LDAC** and **Windows starts playing**,  
  → **Android auto-pauses** without user input (as documented).

- But when **Windows stops or is paused**,  
  → **Android does not resume playback** automatically.

###  Why?

- Android respects AVRCP session priority — but **does not reclaim it** unless playback is manually triggered.
- No media session arbitration or resume logic is built into the AVRCP protocol itself.
- Android sees the stream as "inactive," but doesn’t assume control unless explicitly told to.

---

###  Manual Resume Required

To switch playback back to Android:
1. Pause playback on Windows
2. Manually press play on:
   - Your Android media player
   - Or the headset button

Only then will Android take over the LDAC stream.

>  LDAC codec remains active — but **A2DP session control** is idle until reassigned.


##  What Happens if Both Devices Are Paused?

>  Scenario: You pause playback on both Android and Windows.  
> Then, you press  on the headset.

###  Result

-  **The last device you manually started playback on will resume**
-  The other device stays paused
-  This happens **even if that device wasn’t the last to play audio**

###  Why?

This is due to:
- The **AVRCP media session history** stored by the headset
- Headphones "remember" the **last playback command origin**, not just audio output
- Pressing  sends a **generic play command** to all connected AVRCP sessions
- The **most recently active session** wins arbitration

---

###  Practical Implication for LDAC Multipoint

| State                       | Headset  Resumes |
|----------------------------|--------------------|
| Android last pressed play  |  Android         |
| Windows last pressed play  |  Windows         |
| Both paused manually       |  Last interacted |

>  If neither app is open or has a visible session, **nothing happens** when play is pressed.



##  Multipoint AVRCP Conflict with LDAC and AV ON — Advanced Edge Case

When using multipoint LDAC with **Absolute Volume ON on both Android and Windows**, you may encounter an AVRCP Controller (CT) role conflict which affects metadata updates and playback controls.

---

###  Conditions

| Parameter       | Configuration              |
|------------------|-----------------------------|
| Codec            | LDAC (any profile: fixed/adaptive) |
| Absolute Volume  | ON (both Android and Windows) |
| AVRCP Version    | 1.5 or 1.6 |
| Multipoint Mode  | Active |

---

###  Symptoms

-  Metadata not updating on Android (track info frozen, stale, or blank).
-  Playback controls (play, pause, skip) in Sony Headphones app unresponsive.
-  Audio playback itself fully works on both devices.
-  Metadata may still reflect the last known song before conflict occurred.

---

###  Root Cause

- AVRCP Controller (CT) role arbitration is **not properly negotiated** when both devices hold AV ON.
- Windows often holds CT role longer after playback activity.
- Android fails to reclaim CT role when resuming playback while Windows remains paired.
- Sony WH-1000XM5 firmware prioritizes stability and disables metadata/control updates when AVRCP conflict is detected.

---

###  Workarounds

| Method                 | Effect                          |
|-------------------------|-----------------------------------|
| Disable AV on Android   | Android fully takes CT role |
| Disable AV on Windows   | Windows releases CT role cleanly |
| Connect Android first   | Higher chance Android owns CT |
| Power cycle headphones  | Full role reset |

---

###  Not a Codec Problem

- This behavior is **not related to LDAC stability**.
- It occurs even with correct LDAC negotiation and 990kbps active.
- This is strictly **AVRCP control layer arbitration** under multipoint.








##  Absolute Volume Toggle Desync (AVRCP Role Conflict)

###  Background

When switching Absolute Volume (AV) between ON and OFF during multipoint testing, **Bluetooth stack-level AVRCP role state may desynchronize** even if system settings reflect the change.

This desync can occur due to:

- Cached AVRCP CT/TG role negotiation state not resetting cleanly
- Incomplete AV role renegotiation when Bluetooth remains active during setting changes
- AVRCP 1.6 hardening behavior in Sony firmware expecting full AV synchronization across multipoint peers

---

###  Symptoms

- Multipoint LDAC behavior unstable after AV toggles
- Metadata stalls or fails to update on one device
- Headphone buttons stop controlling Windows playback
- Unlock stutter or pause glitches on Android when resuming
- Audio control priority incorrectly shifts between Android and Windows

---

###  Root Cause

Absolute Volume setting changes require **full Bluetooth stack reload** to properly reset AVRCP role negotiation.  
When AV is toggled without fully disabling Bluetooth:

- AVRCP Controller (CT) role may remain partially cached
- Headset firmware sees conflicting AV role states between Android and Windows

---

###  Recovery Procedure

Whenever Absolute Volume settings are modified:

1. **Fully disable Bluetooth on Android**
2. **Disable Bluetooth on Windows**
3. (Optional: Clear recent devices on Android Bluetooth menu)
4. **Re-enable Bluetooth** on both devices
5. **Reconnect multipoint pairing**

This ensures clean AVRCP role synchronization during initial multipoint handshake.

---

###  Engineering Note

- This is not a defect — it’s stack-level AVRCP negotiation behavior under 1.6 spec.
- Only occurs when AV settings are toggled mid-session.
- If AV settings remain stable, no desync occurs.

---

 Including this procedure ensures maximum stability when experimenting with multipoint, Tasker automation, and BCC profile chaining across mixed AV configurations.







##  Multipoint Stability — AVRCP 1.6 Idle Auto-Pause Behavior (Windows 11)

###  Observed Behavior

When using **LDAC multipoint** across Android + Windows 11, the following edge case may occur under certain AVRCP combinations:

- Windows 11 uses AVRCP **1.6** (Default Microsoft stack or Alt Driver with forced 1.6 registry)
- Android is actively streaming LDAC (primary A2DP session)
- Windows holds open but idle media sessions (Spotify, VLC, Tidal, etc.)
- After several seconds of Windows A2DP inactivity, Windows **auto-pauses** media playback

---

###  Root Cause Explanation

| Layer | Behavior |
|-------|----------|
| **Windows AVRCP 1.6 Controller** | Keeps active playback state (CT role) even while not streaming A2DP audio |
| **Android AVRCP 1.6 Controller** | Controls active LDAC stream |
| **XM5 Multipoint Firmware** | Holds dual CT roles (Windows + Android) simultaneously |
| **Windows A2DP Session Manager** | Detects A2DP idle timeout → triggers auto-pause → updates Windows media session state |

---

###  Why This Only Occurs with AVRCP 1.6

- AVRCP 1.6 adds full playback synchronization (media position, state tracking, resume signals).
- Windows tries to maintain active CT role even while not streaming.
- Idle A2DP session triggers system-level media session pause to clear inactive state.
- AVRCP 1.5 does **not** synchronize playback state fully → Windows stays passive.

---

###  Stability Solutions

| Fix | Method | Result |
|-----|--------|--------|
|  **Fix #1 — Downgrade Windows AVRCP to 1.5 (Recommended)** | Use Alt Driver + registry override | Prevents Windows session auto-pause completely |
|  **Fix #2 — Fully Close Media Apps Before Switching Playback** | Manually stop media apps (Spotify, VLC, Tidal) | No open media sessions = no auto-pause event triggered |

---

###  Verified Resolution Path

-  **Closing media apps on Windows before switching playback** fully prevents the issue.
-  **AVRCP 1.5 downgrade** remains the most stable long-term solution across all multipoint sessions.
-  Android remains fully stable at AVRCP 1.6 throughout.




## ✅ Multipoint Feeding Chain — Protocol Rule (AVRCP 1.6)

- Under AVRCP 1.6, LDAC codec profiles do not have to match across devices.
- Each A2DP session negotiates codec parameters independently inside the headset firmware.
- Windows may negotiate e.g. 96/24 Adaptive while Android negotiates 44.1/24 Fixed.
- Sony WH-1000XM firmware maintains isolated codec states for both active A2DP sessions.
- Stability holds as long as timing windows are respected during active master switching.
- AVRCP governs media control roles, not codec layer negotiation directly.



### Hands-Free Telephony Conflict Mitigation (Windows)
- In rare cases, Windows may hold active A2DP state even when idle due to HFP profile interference.
- Disable "Hands-Free Telephony" in:
  - `Control Panel → Devices and Printers → WH-1000XM5 → Services`
- This ensures Windows cannot request SCO profile, preserving clean multipoint A2DP priority handoff.


### DSP Feeding Rule with Absolute Volume
- Sony WH-1000XM5 DSP pipeline assumes Absolute Volume ON with full digital headroom.
- AV OFF may result in device-side digital attenuation prior to Bluetooth encoding.
- Recommended:
  - With AV ON: Source volume ≥85% yields optimal LDAC quantization.
  - With AV OFF: Carefully match Android player gain to avoid accidental bit-depth compression.


### No Auto-Resume Logic in AVRCP
- AVRCP protocol does not define resume priority.
- When one device stops, the other remains idle until playback is manually resumed.
- Last interacted AVRCP session is remembered by headset firmware to resolve next play command.


### VLC Media Session Limitation (Windows)
- VLC may not respond to AVRCP headset buttons while minimized.
- Lacks full Windows MediaSession integration unless configured.
- Use Spotify, Tidal, or Windows Media Player for full AVRCP multipoint integration.

![image](https://github.com/user-attachments/assets/059b5155-c9cd-4ba7-b3e3-74d9773d53c5)




---









#  Bluetooth A2DP Codec Support on Windows

## Overview

| **Codec**         | **Windows 10 (Default Stack)**                              | **Windows 11 (Default Stack)**                                      | **Alternative A2DP Driver (Win 10 & 11)**                          |
|-------------------|-------------------------------------------------------------|----------------------------------------------------------------------|--------------------------------------------------------------------|
| **SBC**           |  Native support                                            |  Native support                                                     |  Yes — fully configurable                                         |
| **AAC**           |  Not supported                                             |  Native since Windows 11 21H2                                       |  Yes — with AAC-support edition license                          |
| **aptX Classic**  |  Only via OEM/chipset drivers (e.g., Qualcomm stack)       |  Only via OEM/chipset drivers                                      |  Yes                                                              |
| **aptX HD**       |  Not supported                                             |  Not supported                                                      |  Yes                                                              |
| **aptX LL**       |  Not supported                                             |  Not supported                                                      |  Yes                                                              |
| **aptX Adaptive** |  Not supported                                             |  Supported only on select Qualcomm PCs with Windows 11 24H2+       |  Not supported                                                    |
| **LDAC**          |  Not supported                                             |  Not supported                                                      |  Yes                                                              |

---

##  Key Facts

- Windows does **not officially support aptX**. Any aptX use comes from **OEM-provided Bluetooth drivers** (e.g., Qualcomm), **not Microsoft**.
- **Windows 11 21H2 and later** natively supports **AAC** over Bluetooth A2DP.
- **aptX Adaptive** is supported only on select **Qualcomm-powered Windows 11 (24H2+) devices**.
- The **Alternative A2DP Driver** (by BluetoothGoodies) provides:
  -  SBC (fully configurable)
  -  AAC (licensed edition)
  -  aptX (Classic, HD, LL)
  -  LDAC (configurable bitrate/mode)
  -  **Per-device codec profile storage** — each paired device can retain its own preferred codec and settings.

---

##  Summary

| **OS/Driver**      | **SBC** | **AAC** | **aptX** | **aptX HD** | **aptX LL** | **aptX Adaptive** | **LDAC** | **Per-Device Profiles** |
|--------------------|--------|--------|---------|-------------|-------------|--------------------|----------|--------------------------|
| Windows 10         |      |      |  OEM |           |           |                  |        |                        |
| Windows 11         |      |      |  OEM |           |           |  24H2+ OEM       |        |                        |
| Alt. A2DP Driver   |      | *    |       |           |           |                  |        |                        |

> *AAC support in Alternative A2DP Driver requires a purchased AAC-enabled edition.

---

##  Want to verify or install?

You can:
- Use **Bluetooth Tweaker** to verify active codec
- Use the **Alternative A2DP Driver** for full codec control
- Check if your OEM has provided aptX/aptX Adaptive-capable drivers




##  LDAC Kills Your Mic — No A2DP Codec Supports Voice Input

**LDAC**, **AAC**, and **SBC** are excellent for media playback — but **none of them support microphone input** over Bluetooth.

This isn’t a bug — it’s **by design**, due to the **Bluetooth A2DP specification** and how **Android handles audio profiles**.

---

###  Multipoint Codec Matrix (No LDAC on Both)

Supports:
-  Android → LDAC / AAC / SBC  
-  Windows → AAC / SBC / aptX  
-  No LDAC on Windows  
-  Mic only works via HSP/HFP fallback (SCO, one active SCO link only)

---

###  Full Compatibility Matrix

| Android Codec | Windows Codec | Media Quality (A / W) | Resume Stability | Notes |
|----------------|----------------|------------------------|-------------------|-------|
| LDAC (Fixed)    | SBC           |  Excellent /  Low    |  High           | Hi-Fi Android, SBC fallback |
| LDAC (Fixed)    | AAC           |  Excellent /  Good   |  Medium         | Mic fallback triggers |
| LDAC (Fixed)    | aptX          |  Excellent /  Good   |  High           | Playback only |
| AAC (A2DP)      | SBC           |  Good /  Low         |  High           | Voice triggers profile swap |
| AAC (A2DP)      | AAC (A2DP)    |  Good /  Good        |  Medium         | No mic support |
| AAC (A2DP)      | aptX          |  Good /  Good        |  High           | Playback only |
| SBC (A2DP)      | SBC (A2DP)    |  Low /  Low          |  Max            | Playback only; lowest denominator |
| SBC (A2DP)      | AAC (A2DP)    |  Low /  Good         |  High           | Playback stable |
| SBC (A2DP)      | aptX          |  Low /  Good         |  High           | Playback stable |

---


## 🆕 AVRCP 1.5 vs 1.6 Stability Tradeoff Table

| Behavior | AVRCP 1.5 | AVRCP 1.6 |
|----------|-----------|-----------|
| Override Control Stability | ✅ Stable | ✅ Stable |
| Codec Negotiation Stability | ✅ Stable | ⚠ May renegotiate on unlock |
| Multipoint Unlock Stutter | ❌ None | ✅ Possible |
| Metadata Sync Features | Basic | Extended |
| Multipoint Switching Speed | ✅ Instant | ✅ Instant |
| Media Control | ✅ Reliable | ✅ Reliable |

---

## 🆕 Formal Multipoint Passive Role Rule

- Android operates best as passive controller while Windows holds active playback.
- Avoid media activity on Android while Windows is playing to minimize role switching.
- Android will automatically resume if Windows pauses.

---

## 🆕 Multipoint Failure Scenario Table

| Trigger | Failure Mode |
|---------|--------------|
| Codec profile mismatch | LDAC fallback to SBC |
| AVRCP version mismatch | Unexpected renegotiation |
| Android AV ON | Samsung override may re-trigger |
| Fast Pair Device Sync enabled | Google override reassertion |
| Unlock Android during Windows idle | AVRCP 1.6 renegotiation → brief stutter |

---

## 🆕 Master Multipoint Startup Flow

1️⃣ Complete override defeat chain via Tasker.  
2️⃣ Disable Google Fast Pair Device Sync (`Auto Save Devices OFF`).  
3️⃣ Verify LDAC 44100 16 909 active on both devices.  
4️⃣ Set AVRCP version (`1.5` for stability, `1.6` only if fully hardened).  
5️⃣ Connect multipoint (Windows active first, Android passive).  
6️⃣ Avoid Android unlock while Windows is streaming.

---

## 🆕 Master Stability Rule

> Multipoint stability depends on fully mirrored codec profiles, AVRCP version alignment, proper override defeat chaining, firmware profile persistence, and correct unlock behavior timing.

---



## 🆕 Multipoint Codec Profile Mirror Rule (AVRCP 1.6 Dependency)

**Tier:** S — Core Stability Rule

**Description:**

Multipoint operation under AVRCP 1.6 requires both devices to fully mirror their A2DP codec profile parameters.  
While AVRCP 1.6 itself does not negotiate codec settings, the headset firmware expects stable codec alignment to prevent renegotiation or stream dropouts when switching control roles.

**Formal Rule:**

> Multipoint is unstable with AVRCP 1.6 unless codec profiles fully align across both devices.

**Stability Requirements:**

| Parameter | Required Value |
|-----------|------------------|
| Codec | LDAC |
| Sample Rate | 44100 Hz |
| Bit Depth | 16-bit |
| Bitrate | 909 kbps (or mirrored target) |
| LDAC Mode | Fixed (or identically adaptive) |
| Override Stack | Fully defeated via BCC chaining |
| Passive Role | Android connects first, Windows second |

**Why This Rule Exists:**

- AVRCP 1.6 handles playback control signaling but is unaware of codec state.
- The Sony WH-1000XM firmware operates one shared codec negotiation session for both A2DP connections.
- Misaligned codec profiles force renegotiation during role switches, leading to stutter, latency, or complete stream dropouts.

**Your Guide Compliance:**

✅ This rule is fully respected across your override chain:

- BCC chaining ensures Android profile alignment.
- Alternative A2DP Driver ensures Windows profile alignment.
- Firmware handshake training ensures persistent codec state.
- Passive role rule ensures stable AVRCP handoff behavior.
- AV OFF suppresses AVRCP sync renegotiation signals during Android wake.

## 🆕 AVRCP 1.5 vs 1.6 Multipoint Alignment Table (Strict Protocol Dependencies — Fully Locked Edition)

| Parameter | AVRCP 1.5 (as you documented) | AVRCP 1.6 (as you documented) |
|------------|--------------------------------|--------------------------------|
| Sample Rate Alignment | ✅ Required | 🔒 Mandatory |
| Bit Depth Alignment | ✅ Required | 🔒 Mandatory |
| Bitrate Alignment | ✅ Recommended | 🔒 Mandatory |
| LDAC Mode (Fixed vs Adaptive — must be mirrored) | ✅ Fixed recommended | 🔒 Mandatory (both devices must match: Fixed or Adaptive) |
| AV OFF Use (Android) | ✅ Used to simplify override | 🔒 Required to eliminate unlock stutter |


## ✅ Absolute Volume Alignment Rule

Absolute Volume settings do not need to match across Android and Windows for multipoint LDAC stability.

- A2DP codec negotiation is fully independent of Absolute Volume state.
- Use **AV OFF on Android** to fully suppress Samsung override stack and allow stable BCC chaining.
- Use **AV ON on Windows** to maintain volume synchronization and stable AVRCP control behavior.
- Codec handshake, profile mirroring, and multipoint switching remain fully stable regardless of AV mismatch.

This rule holds for both Fixed and Adaptive LDAC modes as long as profiles are mirrored across both devices.


## ✅ Independent A2DP Codec State Rule

The Sony WH-1000XM5 headset maintains fully independent codec state for each A2DP connection during multipoint operation.

- Android and Windows negotiate LDAC codec parameters independently when connecting.
- The headset firmware preserves both negotiated A2DP codec profiles in parallel.
- Profile mirroring across devices ensures seamless multipoint switching without renegotiation.
- If profiles are mismatched, codec renegotiation may occur during role switching or unlock events.

A2DP codec negotiation is isolated from AVRCP control roles and Absolute Volume state.


## ✅ AVRCP CT/TG Role Assignment Rule

AVRCP Controller (CT) and Target (TG) roles are assigned dynamically based on active media session ownership, not pairing order.

- The device actively playing audio during initial connection typically takes CT role.
- CT role may shift when playback resumes or switches between devices.
- Metadata sync and headset button control follow CT ownership.
- Pairing sequence has no fixed effect on CT/TG assignment once media sessions initialize.
- Both devices may hold CT or TG roles in parallel depending on media session state under AVRCP 1.6 multipoint.

Headset behavior remains stable as long as codec profiles are mirrored and CT/TG transitions occur cleanly.


## ✅ LDAC Bit Depth Independence Rule (Multipoint)

Bit depth does not affect LDAC multipoint stability as long as sample rate, LDAC mode, and codec type are fully mirrored.

- A2DP codec negotiation is independent for each device.
- The headset firmware maintains separate codec state per A2DP session.
- 16-bit and 24-bit profiles may coexist without renegotiation or stream drops.
- Bit depth mismatches have no impact on CT/TG role switching or unlock stability.

This rule applies to both Fixed and Adaptive LDAC modes.


 ## ✅ Fast Pair Cloud Sync Neutralization Rule (Multipoint)

- BCC profile chaining overrides any Fast Pair Device Sync cloud profile assertion during multipoint A2DP negotiation.
- Cloud profile resynchronization does not interfere with codec stability once BCC chaining is active.
- Multipoint codec stability remains fully controlled by BCC profile mirroring, independent of Fast Pair state.


Adaptive 44.1kHz Multipoint Stability

- Full profile mirroring at Adaptive 44.1kHz across Android + Windows is stable.
- No renegotiation under unlock, role switching, or stream switching with AVRCP 1.6.
- Both 16-bit and 24-bit versions validated as stable.
- No longer classified as unstable or edge-case dependent.

### ✅ Target 2 — Adaptive 88.2kHz Multipoint Stability

- Adaptive 88.2kHz profiles are semi-stable across multipoint when profiles are fully mirrored.
- Stability is highly dependent on RF environment quality and link budget.
- Unlock, resume, and role switching remain stable under ideal RF conditions.
- Minor bitrate negotiation drops may occur in degraded RF scenarios but do not trigger full codec renegotiation if profiles remain matched.
- Both 16-bit and 24-bit variants tested with consistent behavior.

## 🆕 Windows — Alternative A2DP Driver: Full LDAC Control Logic

The Alternative A2DP Driver on Windows exposes full LDAC control, allowing independent selection of:

- Sample rate
- Channel count
- Bit depth
- Codec mode (Fixed vs Adaptive)
- Encoding quality (bitrate control)

The Alternative A2DP Driver uniquely allows creation of an Adaptive LDAC *ceiling* by modifying Encode Quality — something not exposed on Android or stock Windows Bluetooth stacks.

---

### 1️⃣ LDAC Mode Selection (Fixed vs Adaptive)

The following checkbox directly toggles LDAC operating mode:

> **Automatically reduce the encoding quality when the radio quality is poor**

| Checkbox State | LDAC Mode |
|----------------|-----------|
| ✅ Checked | Adaptive Mode |
| ❌ Unchecked | Fixed Mode |

---

### 2️⃣ Encode Quality Behavior (Sample Rate Dependent)

The `Encode Quality` setting defines LDAC transmission bitrate, controlled separately for each sample rate group.

#### LDAC Fixed Mode — Explicit Bitrate Mapping

| Sample Rate | Encode Quality: Low | Mid | High |
|--------------|---------------------|-----|------|
| 44.1 / 88.2 kHz | 303 kbps | 606 kbps | 909 kbps |
| 48 / 96 kHz | 330 kbps | 660 kbps | 990 kbps |

#### LDAC Adaptive Mode — Maximum Bitrate Ceilings

| Sample Rate | Encode Quality: Low | Mid | High |
|--------------|---------------------|-----|------|
| 44.1 / 88.2 kHz | ≤ ~303 kbps | ≤ ~606 kbps | ≤ ~909 kbps |
| 48 / 96 kHz | ≤ ~330 kbps | ≤ ~660 kbps | ≤ ~990 kbps |

> 🔬 *Adaptive dynamically lowers bitrate depending on radio conditions, but Encode Quality defines the ceiling.*

---

✅ **Key Summary**

- LDAC bitrate is grouped into 44.1 kHz family and 48 kHz family buckets.
- Alternative A2DP Driver applies these mappings behind the UI based on selected sample rate.
- Sample rate, bit depth, and channel count are manually selectable.
- Encode Quality provides precise control over Adaptive ceilings — useful for multipoint or unstable RF environments.
- This fine-grained control is exclusive to the Alternative A2DP Driver.

---

📌 **Note:**  
This mechanism is a direct Windows-side equivalent of what Android lacks: external control over Adaptive ceilings via codec profile logic.

---

### 🔬 Live Adaptive Bitrate Feedback

When using Adaptive Mode, the Alternative A2DP Driver exposes real-time bitrate updates:

- The displayed quality value actively reflects the **current negotiated bitrate** during playback.
- As RF conditions fluctuate, the bitrate indicator updates dynamically.
- The `Encode Quality` setting still controls the maximum ceiling, but actual bitrate varies live below that ceiling.
- This behavior allows direct observation of Adaptive scaling behavior in real time.

| Platform | Adaptive Quality Display | Behavior |
|----------|---------------------------|----------|
| Windows (Alternative A2DP Driver) | ✅ Real-time bitrate updates | Live adaptive bitrate feedback shown |
| Android | ❌ No live updates | Only static codec mode shown |

> 📝 Note: This live feedback is unique to the Alternative A2DP Driver and not available on Android.

## 🔬 Adaptive Bitrate Display Validation Rule (Alternative A2DP Driver)

> ⚠️ **Important:** Always measure Adaptive bitrate ceilings while audio is actively playing.

### 🎯 Why This Rule Exists

- LDAC Adaptive bitrate is dynamically negotiated during active audio playback.
- The Alternative A2DP Driver GUI only displays correct Adaptive bitrate values when:
  - Audio is actively streaming over A2DP
  - The encoder is actively negotiating RF link quality
- When audio is paused or idle:
  - The GUI may display stale, cached, or default bitrate values.
  - These values do not reflect actual adaptive negotiation state.

### 🔧 Measurement Rule

| Playback State | Bitrate Display Valid? |
|-----------------|------------------------|
| ✅ Audio actively playing | ✅ Accurate real-time bitrate |
| ❌ Audio paused or idle | ❌ Invalid / stale bitrate display |

### 🧪 Testing Procedure Addition

To ensure valid Adaptive bitrate ceiling measurements:

1. Use **lossless high-bitrate test content** (e.g., FLAC / WAV files).
2. Enable **WASAPI Exclusive Mode** (if supported by player) to avoid resampling.
3. Start audio playback at target sample rate and bit depth.
4. Let playback stabilize for at least **10 seconds**.
5. Observe the **real-time bitrate quality readout** inside the Alternative A2DP Driver GUI.
6. Record highest stable bitrate observed during active playback.

### 🔬 Why This Rule Is Critical

- Prevents false conclusions due to idle-state display artifacts.
- Guarantees scientifically valid Adaptive ceiling mapping results.
- Ensures hardware adaptive behavior is accurately captured.

---

> ✅ Always verify adaptive bitrate while streaming live audio — never trust idle GUI readouts.


## 🔬 LDAC Bitrate Display Validation Rules (Alternative A2DP Driver)

> ⚠️ **Important:** The Alternative A2DP Driver GUI behaves differently depending on whether you are using Adaptive or Fixed Mode. Understanding these differences is critical when measuring bitrate ceilings.

---

### 🔧 Key Differences: Adaptive vs Fixed Mode

| Mode | Negotiation Behavior | GUI Display Behavior | When to Measure |
|------|-----------------------|-----------------------|------------------|
| **Adaptive Mode** | Dynamic — bitrate constantly adjusts based on RF link quality during playback. | Displays *real-time negotiated bitrate* during active playback. Displays stale or default values when idle. | Always measure bitrate during active playback. |
| **Fixed Mode** | Static — bitrate is configured during connection and remains constant unless renegotiation is triggered by link failure. | Displays configured bitrate even when idle. No real-time negotiation occurs after connection. | Still verify during playback to ensure no silent fallback occurred. |

---

### 🔬 Adaptive Mode — Measurement Rule

- Adaptive bitrate is actively negotiated only during playback.
- Idle GUI values are often stale, cached, or meaningless.
- Always test Adaptive Mode while audio is actively streaming.

| Playback State | Adaptive Bitrate Display Valid? |
|-----------------|----------------------------------|
| ✅ Audio actively playing | ✅ Accurate |
| ❌ Audio paused or idle | ❌ Invalid |

---

### 🔬 Fixed Mode — Measurement Rule

- Fixed Mode bitrate is configured during connection and remains locked unless renegotiation is forced by RF failure.
- GUI displays configured bitrate even when idle.
- However, link renegotiation or codec fallback can still occur under certain RF failures.
- Therefore, it’s best practice to validate Fixed Mode bitrate during active playback.

| Playback State | Fixed Bitrate Display Valid? |
|-----------------|------------------------------|
| ✅ Audio actively playing | ✅ Valid — confirms active transmission |
| ✅ Audio idle | ✅ Reflects profile setting — but may not guarantee active link state |

---

### 🧪 Testing Procedure Summary

To ensure scientifically valid Adaptive and Fixed bitrate ceiling measurements:

1. Use **lossless high-bitrate test content** (e.g., FLAC / WAV).
2. Enable **WASAPI Exclusive Mode** (if available) to avoid resampling.
3. Start playback at desired sample rate and bit depth.
4. Allow playback to stabilize for ~10 seconds.
5. Observe real-time bitrate in the Alternative A2DP Driver GUI.
6. Record highest stable bitrate observed during active playback.

---

> ✅ Always verify both Adaptive and Fixed bitrate while audio is actively streaming to ensure valid and accurate ceiling measurements.

## 🚩 Multipoint Firmware Limitation — AVRCP 1.6 Controller Role Swap LDAC Buffer Desync

### Description

A critical firmware-level limitation exists on Sony WH-1000XM5 multipoint operation when using AVRCP 1.6 across multiple devices. Under certain playback handoff conditions, the headset's internal stream buffer manager fails to fully synchronize LDAC A2DP buffers during cross-device controller role transitions, resulting in permanent stuttering.

### Trigger Condition

This issue is not dependent on Adaptive vs Fixed LDAC mode — it affects both modes:

- Both Android and Windows are connected via multipoint using AVRCP 1.6.
- One device is actively playing.
- Playback is paused on Device A.
- Playback is resumed on Device B.
- Playback is paused again on Device B.
- Playback is resumed on Device A.

At the moment of controller role swap, the XM5 firmware must realign A2DP buffer pointers to the new AVRCP Controller (CT). Under certain timing conditions, this realignment fails, desynchronizing the LDAC stream buffers.

### Technical Root Cause

- The XM5 firmware fails to properly resynchronize A2DP LDAC stream buffers when AVRCP 1.6 Controller role transitions occur during active multipoint sessions.
- The A2DP data path (LDAC stream) becomes permanently misaligned with the controller state machine after CT/TG role changes.
- Once triggered, stuttering is permanent for the session and only fully disconnecting both devices resets the internal buffer state.
- Additional CT role swaps after the desync (via play/pause across devices) do not restore buffer alignment. The control layer continues functioning independently from the broken data path.

### Key Facts

- ✅ Occurs in both Fixed and Adaptive LDAC modes.
- ✅ Confirmed at Fixed 88.2 kHz / 24-bit LDAC.
- ✅ Wi-Fi or RF conditions are not involved.
- ✅ Metadata, play/pause, and browsing functions remain fully functional via AVRCP control plane.
- ✅ Headphone physical buttons continue to work normally and can control both devices after stutter begins.
- ✅ Repeated play/pause swaps across devices after desync have no effect on restoring stream alignment.
- ✅ Only cross-device play/pause handoffs combined with AVRCP 1.6 CT swaps trigger the condition.
- ✅ AVRCP 1.5 fully avoids this condition due to simpler controller role handling.

### Mitigation Strategies

| Mitigation | Explanation |
|-------------|-------------|
| Avoid cross-device play/pause handoffs | Prevents CT role swaps that can trigger buffer desync |
| Use AVRCP 1.5 on Windows | AVRCP 1.5 role handling is safer for multipoint |
| Use Fixed LDAC 48 kHz / 96 kHz | Wider buffer margin reduces sensitivity |
| Limit multipoint playback to one active device at a time | Eliminates role swap risks |

### Firmware Class

This issue is a **pure XM5 firmware-level multipoint arbitration defect**. It cannot be mitigated via BCC, Developer Options, AV settings, or Bluetooth stack tuning.

## 🔬 Multipoint Simultaneous Resume Conflict — Timing Dependency Audit

### Background
A potential firmware-class failure was evaluated to determine whether simultaneous resume (Play) commands across Android and Windows could independently trigger LDAC buffer desynchronization during multipoint operation.

### Timing-Dependent Behavior Observed
- When both resume commands are triggered **near-perfectly simultaneously**, the Sony WH-1000XM5 firmware temporarily allows both A2DP streams to coexist without immediate desync or stutter.
- This demonstrates that the internal A2DP buffer arbitration system has short-term tolerance for dual Controller (CT) role conflicts.
- When resume commands are triggered **with slight timing offset (even 100–300ms)**, the firmware may prematurely enter Controller Role arbitration, increasing the chance of **immediate buffer desync and permanent stuttering**.
- This failure pathway feeds directly into the previously documented **Firmware Desync Class #1 (AVRCP 1.6 Controller Role Swap Buffer Desync)**.

### Conclusion
- **Simultaneous resume behavior is fully timing-sensitive.**
- No independent Firmware Desync Class #2 exists.
- All buffer desynchronization failures remain fully captured under **Firmware Desync Class #1**.
- Firmware arbitration layer coverage is now complete and fully exhausted for multipoint LDAC operation on WH-1000XM5.

## 🔬 Multipoint Adaptive LDAC — Firmware Window Lock Behavior

### Firmware Window Lock (Verified Behavior)

- When Adaptive LDAC profiles are fully mirrored (sample rate, bit depth, Adaptive mode) on both Android and Windows, the headset firmware creates a synchronized negotiation window.
- Once this window is locked, renegotiation spikes are fully suppressed even when switching playback between Android and Windows.
- Firmware caching ensures stability across:
  - CT/TG role swaps
  - Android unlock events
  - AVRCP 1.6 triggers
  - Playback handoffs between hosts

### Adaptive Bitrate Adjustment Stability

- Adaptive bitrate recalculations reflect normal RF-based Adaptive behavior, not renegotiation.
- Android playback:
  - `adaptive bit rate adjustments: 0` initially (perfect RF conditions).
  - Later sessions observed multiple adjustments (`12`), fully contained inside the negotiation window.
- Windows playback:
  - Observed up to `5` adaptive adjustments during link stabilization.
- Both platforms maintained fully synchronized Adaptive windows throughout.

### Shared Firmware State Across Hosts

- Even while Android is playing, Windows Alternative A2DP Driver continuously polls and displays real-time Adaptive bitrate from firmware.
- Windows reported values like `452 kbps` while Android actively played.
- This confirms that both hosts share the same firmware-managed Adaptive state without renegotiation.

### Adaptive Bitrate Encode Quality Index Mapping (Observed)

| Platform | Adaptive Quality Index | Behavior |
|----------|------------------------|----------|
| Android | `-1` | Full firmware control |
| Windows (A2DP Driver) | `1` → `3` | Driver-controlled quality window sync |

- Both still negotiate identical codec capability (CCOD `64`, PCOD `66`), allowing full cross-platform synchronization.

### Practical Outcome

- ✅ Adaptive LDAC can reach Fixed-mode stability under fully mirrored profiles.
- ✅ Renegotiation spikes (bitrate recalculation triggers from system events) are fully suppressed.
- ✅ Adaptive bitrate continues to dynamically adjust based on RF conditions inside the locked firmware window.
- ✅ Firmware window lock allows stable multipoint playback even under AVRCP 1.6 CT/TG swaps, Android unlocks, and host handoffs.


# LDAC Done Right — Verification Methods

This module documents how to verify actual live LDAC codec behavior during playback. These tools allow full experimental monitoring of bitrate, adaptive negotiation, and firmware state across Android and Windows.

---

## 🔧 Android — ADB Dumpsys Verification (Primary Live Method)

On Android, system UI, Developer Options, and BCC GUI do not reflect actual real-time LDAC bitrate during playback. Only ADB dumpsys exposes live negotiated bitrate and adaptive state.

### Command (Bluetooth Stack Dependent)

```bash
adb shell dumpsys bluetooth_manager
```

### Key Output Fields

| Field | Description |
|-------|-------------|
| LDAC transmission bitrate (kbps) | Current negotiated bitrate |
| LDAC adaptive bit rate adjustments | Number of bitrate recalculations |
| LDAC quality mode : ABR | Adaptive mode active confirmation |
| Adaptive bit rate encode quality mode index | Platform adaptive mode index |
| CCOD / PCOD | Codec capability negotiation window |

✅ Dumpsys is mandatory for all Adaptive multipoint firmware testing.

---

## 🔧 Android — Bluetooth Codec Changer (BCC) Limitations

- BCC correctly reports applied profile at connection time.
- BCC does not display live adaptive bitrate negotiation during playback.
- For Adaptive verification, always combine BCC with dumpsys.

---

## 🔧 Windows — Alternative A2DP Driver Bitrate Display

The Alternative A2DP Driver (BluetoothGoodies) exposes full live Adaptive bitrate telemetry during playback:

- GUI reports live firmware bitrate negotiation.
- Even during multipoint, Windows polls headset firmware for current adaptive bitrate, even if Android is actively playing.

✅ Use this to observe firmware-shared state across hosts.

---

## 🔧 Windows — Default Bluetooth Stack Limitations

- The Windows Default Bluetooth stack does not expose any live bitrate telemetry.
- Use Alternative A2DP Driver for any Windows-side Adaptive bitrate verification.

---

## 🔧 Summary Table — LDAC Verification Methods

| Platform | Method | Live Bitrate Accuracy |
|----------|--------|-----------------------|
| Android | ADB dumpsys | ✅ Fully accurate |
| Android | BCC App | ⚠ Profile only |
| Windows | Alternative A2DP Driver | ✅ Fully accurate |
| Windows | Default Bluetooth Stack | ❌ No live reporting |

---

## 🔧 Importance for Adaptive Multipoint Testing

These verification methods are essential for:

- Adaptive firmware window lock validation.
- Renegotiation spike immunity testing.
- CT/TG ownership swap analysis.
- Full Adaptive bitrate fluctuation monitoring.

✅ All Adaptive multipoint firmware experiments in this guide require these tools for accurate real-time validation.

---
