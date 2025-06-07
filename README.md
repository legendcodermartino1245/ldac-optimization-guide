## 📑 Table of Contents
- [My setup and the hardware I used during the making of this guide](#my-setup-and-the-hardware-i-used-during-the-making-of-this-guide)
- [Inner workings of LDAC](#inner-workings-of-ldac)
- [LDAC Configuration Matrix Fixed](#-ldac-configuration-matrix-fixed)
- [LDAC Configuration Matrix Adaptive](#-ldac-configuration-matrix-adaptive)
- [The Real LDAC Bug](#the-real-ldac-bug-quality-settings-dont-apply-on-their-own)
- [Samsung LDAC Override Stack](#-samsung-ldac-override-stack)
- [Developer Options](#-developer-options-are-safe--just-clean-up-after-yourself)
- [Bluetooth Codec Changer (BCC)](#bluetooth-codec-changer-bcc)
- [Intermediate Profile Switch](#intermediate-profile-switch)
- [Verify BCC Isn’t Lying](#verify-bcc-isnt-lying)
- [Adaptive Bitrate Behavior](#adaptive-bitrate-ldac)
- [LDAC Adaptive Mode Stability Matrix](#-ldac-adaptive-mode-stability-matrix)
# LDAC Done Right

There is a lot of misconception about LDAC and how to properly configure it on different operating systems (Windows and Android for example.) This guide will focus on both and will include strategies which help you get the best sound possible. I tried to make it as practical as possible to replicate. To make troubleshooting easier I have clearly documented the steps needed to set everything back to defaults for full transparency.

## My setup and the hardware I used during the making of this guide:

| Device              | OS / Firmware             | Supported Codecs                        | Bluetooth Version     |
|---------------------|---------------------------|-----------------------------------------|------------------------|
| **Samsung S22 Plus** | One UI 6.1 (Android 14)    | LDAC, SSC, AptX, AAC, SBC                | **Bluetooth 5.2**      |
| **Samsung S24**      | One UI 6.1 (Android 14)    | LDAC, SSC, AptX, AAC, SBC                | **Bluetooth 5.3**      |
| **Windows 11**       | Version 24H2               | AAC, SBC *(LDAC not supported natively)* | **Bluetooth 5.3**      |
| **Windows 10**       | Version 22H2               | AAC, SBC *(LDAC not supported natively)* | **Depends on adapter** |
| **Sony WH-1000XM5**  | Firmware 2.4.1             | LDAC, AAC, SBC                           | **Bluetooth 5.2**      |
| **Sony WH-1000XM3**  | Firmware 4.5.2             | LDAC, AptX, AAC, SBC                     | **Bluetooth 4.2**      |

> 📌 *Note: On Windows, LDAC support requires specific Bluetooth drivers or third-party implementations (e.g. CSR Harmony stack or alternative USB dongles). This guide focuses on standard OS behavior unless otherwise noted.*

## Inner workings of LDAC

LDAC supports sample rates ranging from **44.1 kHz to 96 kHz**, quality modes of **330**, **660**, **990**, **303**, **606**, **909** kbps, or **Adaptive**, and supports both **16-bit** and **24-bit PCM input**. You should set LDAC’s input bit-depth to match **exactly** what your player is feeding it to preserve fidelity.

| Playback Scenario                          | Player Output Depth | LDAC Bit-Depth Setting         | Rationale                                                                                        |
| ------------------------------------------ | ------------------- | ------------------------------ | ------------------------------------------------------------------------------------------------ |
| **Pure CD-quality (44.1 kHz / 16-bit)**      | 16-bit              | 16-bit (or "System Selection") | Matches the original 16-bit samples — avoids unnecessary padding, zero-extension, or quantization errors. *System Selection* lets Android auto-choose bit-depth, which usually defaults to 16-bit unless overridden by a hi-res app. |
| **Native Hi-Res (>44.1 kHz / 24-bit)**       | 24-bit              | 24-bit                         | Preserves the full dynamic range of your 24-bit source all the way into LDAC’s encoder.          |
| **Any source + DSP (EQ, gain, fades)**     | 24-bit              | 24-bit                         | Provides headroom for processing; avoids rounding errors during DSP before LDAC encoding.        |
| **Non–bit-perfect apps (mixed to 16-bit)** | 16-bit              | 16-bit (or "System Selection") | Reflects the actual 16-bit data the mixer delivers; keeps your settings honest about input depth. |


> ⚠️ **Clarification:**  
> LDAC **does not always encode at 24-bit**. It encodes audio at **the bit-depth it receives** — 16-bit or 24-bit PCM.  
> Android’s Bluetooth stack forwards the player's output to LDAC without automatic upsampling.  
> - ✅ If the player outputs 16-bit PCM (e.g., CD-quality), LDAC encodes it directly as 16-bit.  
> - ✅ If the player outputs 24-bit PCM, LDAC uses full 24-bit encoding.  
> - ❌ If the player outputs 32-bit float, Android truncates it to 24-bit PCM before LDAC sees it.  

> 🛠 Android typically **resamples audio to a single global sample rate**, such as 48 kHz or 96 kHz — unless you’re using a bit-perfect player like UAPP or Neutron in exclusive mode. That system-wide output is what LDAC actually encodes — not necessarily the source file’s native format.

> 🧠 **The 32-bit setting in Developer Options or BCC is not for LDAC itself**, but for internal processing in apps like UAPP or Neutron, which operate at 32-bit float for DSP. It provides internal headroom but has **no effect on the final transmitted resolution**, which is max 24-bit.

> 🔎 **Bit-perfect transmission over LDAC is only achieved** when the player's output bit-depth and sample rate match the source, and no DSP or mixing occurs.

> 📦 LDAC doesn’t transmit raw PCM — it uses a proprietary compression method combining MDCT and Huffman coding. While it’s lossy, LDAC is designed to retain detail up to 24-bit/96kHz with minimal perceptual degradation when properly configured.

> ⚠️ Reminder: Setting LDAC to 24-bit won’t magically upgrade 16-bit audio. If your player sends 16-bit, that’s exactly what LDAC will encode — no matter what the LDAC bit-depth setting says.



### 🎮 LDAC Configuration Matrix Fixed

| Sample Rate | Bit Depth | Bitrate (kbps) | Mode             |
|-------------|-----------|----------------|------------------|
| 44.1 kHz    | 16-bit    | 303            | Fixed            |
| 44.1 kHz    | 16-bit    | 606            | Fixed            |
| 44.1 kHz    | 16-bit    | 909            | Fixed            |
| 44.1 kHz    | 24-bit    | 303            | Fixed            |
| 44.1 kHz    | 24-bit    | 606            | Fixed            |
| 44.1 kHz    | 24-bit    | 909            | Fixed            |
| 44.1 kHz    | 32-bit    | 303            | Fixed            |
| 44.1 kHz    | 32-bit    | 606            | Fixed            |
| 44.1 kHz    | 32-bit    | 909            | Fixed            |
| 48 kHz      | 16-bit    | 330            | Fixed            |
| 48 kHz      | 16-bit    | 660            | Fixed            |
| 48 kHz      | 16-bit    | 990            | Fixed            |
| 48 kHz      | 24-bit    | 330            | Fixed            |
| 48 kHz      | 24-bit    | 660            | Fixed            |
| 48 kHz      | 24-bit    | 990            | Fixed            |
| 48 kHz      | 32-bit    | 330            | Fixed            |
| 48 kHz      | 32-bit    | 660            | Fixed            |
| 48 kHz      | 32-bit    | 990            | Fixed            |
| 88.2 kHz    | 16-bit    | 303            | Fixed            |
| 88.2 kHz    | 16-bit    | 606            | Fixed            |
| 88.2 kHz    | 16-bit    | 909            | Fixed            |
| 88.2 kHz    | 24-bit    | 303            | Fixed            |
| 88.2 kHz    | 24-bit    | 606            | Fixed            |
| 88.2 kHz    | 24-bit    | 909            | Fixed            |
| 88.2 kHz    | 32-bit    | 303            | Fixed            |
| 88.2 kHz    | 32-bit    | 606            | Fixed            |
| 88.2 kHz    | 32-bit    | 909            | Fixed            |
| 96 kHz      | 16-bit    | 330            | Fixed            |
| 96 kHz      | 16-bit    | 660            | Fixed            |
| 96 kHz      | 16-bit    | 990            | Fixed            |
| 96 kHz      | 24-bit    | 330            | Fixed            |
| 96 kHz      | 24-bit    | 660            | Fixed            |
| 96 kHz      | 24-bit    | 990            | Fixed            |
| 96 kHz      | 32-bit    | 330            | Fixed            |
| 96 kHz      | 32-bit    | 660            | Fixed            |
| 96 kHz      | 32-bit    | 990            | Fixed            |


> The **Adaptive mode** dynamically switches between three bitrate levels, depending on bandwidth and signal strength:
>
> - **303 / 606 / 909 kbps** at 44.1 kHz and 88.2 kHz  
> - **330 / 660 / 990 kbps** at 48 kHz and 96 kHz


### 🎮 LDAC Configuration Matrix Adaptive
| Sample Rate | Bit Depth | Bitrate (kbps) | Mode      |
|-------------|-----------|----------------|-----------|
| 44.1 kHz    | 16-bit    | 303 / 606 / 909 | Adaptive |
| 44.1 kHz    | 24-bit    | 303 / 606 / 909 | Adaptive |
| 44.1 kHz    | 32-bit    | 303 / 606 / 909 | Adaptive |
| 48 kHz      | 16-bit    | 330 / 660 / 990 | Adaptive |
| 48 kHz      | 24-bit    | 330 / 660 / 990 | Adaptive |
| 48 kHz      | 32-bit    | 330 / 660 / 990 | Adaptive |
| 88.2 kHz    | 16-bit    | 303 / 606 / 909 | Adaptive |
| 88.2 kHz    | 24-bit    | 303 / 606 / 909 | Adaptive |
| 88.2 kHz    | 32-bit    | 303 / 606 / 909 | Adaptive |
| 96 kHz      | 16-bit    | 330 / 660 / 990 | Adaptive |
| 96 kHz      | 24-bit    | 330 / 660 / 990 | Adaptive |
| 96 kHz      | 32-bit    | 330 / 660 / 990 | Adaptive |

## The Real LDAC Bug: Quality Settings Don’t Apply on Their Own

> **Changing the LDAC quality setting (330 / 660 / 990 / Adaptive) by itself does nothing.**  
> Unless the system renegotiates the entire codec connection, **your change won’t be applied** — even if the UI says it was.

### What Triggers a Real Codec Reset?
LDAC settings like bitrate, sample rate, and bit depth are **only renegotiated** when one of the following is changed:

- A different **codec** is selected (e.g. SBC → LDAC)
- The **sample rate** changes (e.g. 48kHz → 44.1kHz)
- The **bit depth** changes (e.g. 32-bit → 24-bit)

> ⚠️ This is a bug in Android's Bluetooth stack. UI updates don't guarantee actual codec reconfiguration. Bitrate must always be reapplied after reconnection — it is never saved.

## 🔐 Samsung LDAC Override Stack

Samsung **injects its own LDAC codec profile at the very start of the Bluetooth handshake**:

- **Sample Rate:** 96 kHz  
- **Bit Depth:** 32-bit  
- **Bitrate:** Default (330–990 kbps)

This override happens **before** your device finishes establishing the Bluetooth session.  
However, it can be **reliably bypassed** by forcing a full codec renegotiation after connection, see  The Real LDAC Bug section on how to do it

✅ BCC and other apps **can override** Samsung's initial profile — but only if they trigger a full codec reset after the override is applied.

🧠 *Samsung’s override is not permanent — it’s just the default LDAC handshake. What matters is whether your LDAC session gets renegotiated correctly after that handshake.*

> Developer Options may temporarily display "Playback Quality: Default" when Samsung’s override is active.

## ✅ Developer Options Are Safe — If You Clean Up Properly

> 🛑 Just disabling Developer Options is **not enough** if LDAC was ever manually selected.

Samsung may silently continue applying the **last used LDAC override profile** even after Developer Options are turned off — especially if it was previously set to LDAC 660, Adaptive, or 990.  
LDAC supports both 16-bit and 24-bit input. But the encoder only uses what it receives — not what you select in Developer Options.

### ✅ Correct Reset Procedure

1. Enable **Developer Options** — only if you’ve used them before  
2. Set **Bluetooth Audio Codec** to **SBC**
   - ℹ️ On Samsung, there is **no “Default” option** — selecting **SBC** is the only way to fully clear override behavior  
3. Exit the Developer Options menu  
4. Go back and **disable Developer Options**  
5. *(Optional but safest)* Reset network settings:  
   - `Settings → General Management → Reset → Reset network settings`

✅ This fully clears Samsung’s override memory, ensuring a clean LDAC handshake window for hijack and BCC profile injection.


## 🧠 Codec Negotiation Limits and Override Timing

### 🔁 Samsung LDAC Override: Always Active

Samsung’s Bluetooth stack **forces an LDAC override profile immediately** on connection — even **before** LDAC is explicitly enabled.

- If you connect your headphones **without LDAC enabled**, the override profile is **still injected** (e.g., SBC/AAC override logic).
- If you enable LDAC **after connecting**, it **still uses Samsung’s default LDAC profile** (typically 660 kbps Adaptive).
- Even **first-time pairings** will fall back to Samsung’s default LDAC parameters unless a **clean handshake** is forced.

📌 **Conclusion:**  
You cannot assume LDAC settings are "clean" just because you've enabled it.  
**Always perform a reset or handshake trick** (e.g., SBC → LDAC 16-bit → LDAC 990) if you're trying to apply your own BCC profile.




## Bluetooth Codec Changer (BCC)

BCC allows you to force codec settings (LDAC, bitrate, sample rate, bit depth) **at runtime**.  
These settings are not persistent — they must be applied on **every connection**.

---

### 🔄 Auto Switch

When enabled, Auto Switch:
- Automatically applies your selected codec profile upon Bluetooth connection
- Supports delay configuration to allow time for default codec to settle before override

---

### 🧪 2-Step Switch

| Step | Description                         |
|------|-------------------------------------|
| 1    | Initial connection                  |
| 2    | System selects default codec (usually LDAC or SBC) |
| 3    | Force SBC to reset LDAC session     |
| 4    | Delay (e.g., 500–2000 ms)           |
| 5    | Reapply LDAC with target profile    |
| 6    | Clean LDAC handshake achieved       |

🧠 **Why 2-Step Doesn't Work Reliably on Samsung:**  
Samsung forces its own LDAC profile **before BCC can act**.  
This means:
- Step 3 (SBC switch) may not reset the codec cleanly
- GUI may show incorrect values
- Your target LDAC profile may silently fall back to Samsung’s default

✅ **Workaround:**  
- Apply the profile **twice**  
- Use **Tasker automation** to enforce SBC → LDAC switching manually
- Or use **Intermediate Profile Switch**

---

### ⚙️ Intermediate Profile Switch

In Auto Switch settings, enable **Intermediate Codec Profile** and set it to **SBC**.

- Forces a temporary codec downgrade (SBC) before LDAC is reapplied
- This triggers a **true renegotiation** and breaks Samsung’s override hold
- Without this step, BCC may silently fail or default to 96 kHz LDAC

📌 This is especially important when trying to apply:
- 44.1 kHz / 24-bit / 990 kbps
- Any sample rate that Samsung’s override would normally reject

---

### ✅ Verified 2-Step + Intermediate Profile Pairs

Any switch **away from LDAC**, even briefly, forces the system to renegotiate LDAC cleanly:

- `SBC → LDAC 16-bit → LDAC 990`: **Most reliable**
- `SBC → LDAC 909`: works if your app doesn’t touch Developer Options
- `LDAC Adaptive → SBC → LDAC Fixed`: may restore clean handshake

---

### ⚠️ BCC Limitation: LDAC Must Be Negotiated First

Bluetooth Codec Changer **cannot switch to LDAC 990 kbps** unless LDAC has already been negotiated during the session.

If the system is still in **SBC or AAC mode**, BCC cannot switch to LDAC.

#### ✅ To ensure LDAC 990 can be applied via BCC:
- Enable LDAC in Developer Options **before** connecting
- Or use a handshake trick:  
  `SBC → LDAC 16-bit → LDAC 990`
- Or start playback in a hi-res audio app (like UAPP or Neutron)

🧠 *BCC profiles are runtime-only. If LDAC hasn't been established yet, BCC cannot apply its profile.*

👁️ **BCC GUI Reflects External Codec Changes**

As of the latest update, BCC’s GUI reflects LDAC codec changes made by **other apps or the system**, such as:

- UAPP  
- Developer Options  
- Sony Music Center  
- System-level negotiation (e.g. Fast Pair)

⚠️ On Samsung, GUI accuracy is **only reliable when 2-Step Switch is enabled**.  
However, 2-Step must be disabled to make LDAC profile switching stable.
This means the GUI will often show incorrect bitrate, even though the override succeeded.

✅ For stability, keep **2-Step disabled** and use verification tools like ADB or real-time dumpsys monitoring instead.

🧪 *Verified Behavior – GUI desync is expected with 2-Step disabled. Stability and accuracy cannot both be achieved on Samsung at the same time.*



## Verify BCC Isn’t Lying

Use this PowerShell script to monitor real-time LDAC status:

```powershell
while ($true) {
    Clear-Host
    adb shell dumpsys bluetooth_manager | Select-String "ldac"
    Start-Sleep -Seconds 2
}
```
## Adaptive Bitrate LDAC

In **Adaptive** mode, LDAC may briefly report *non-standard bitrate values* (e.g., **452 kbps**). These transient values occur right at the **start of playback**, as LDAC begins its **bitrate ramp-up process**. It takes a few seconds for the adaptive logic to stabilize and settle on an official bitrate tier:

- **990 / 660 / 330 kbps** → for **48 kHz** and **96 kHz**
- **909 / 606 / 303 kbps** → for **44.1 kHz** and **88.2 kHz**

This is expected behavior and typically resolves quickly.

A key indicator that LDAC Adaptive has stabilized is the log entry:
LDAC adaptive bit rate adjustments: 2

A key indicator that LDAC Adaptive is not yet initialized
LDAC encode quality mode index: -1


> 🔍 **Note on ADB and AudioFlinger:**  
> Sample rate and bit depth shown in `adb shell dumpsys media.audio_flinger` reflect **Android's internal audio mixer output**, not the raw Bluetooth stream.  
> - If **resampling** occurs (e.g., app outputs 44.1 kHz but Android mixes to 48 kHz), LDAC will still encode **48 kHz**.  
> - Apps like **UAPP** and **Neutron** in bit-perfect mode can **bypass AudioFlinger**, allowing true 44.1 kHz to reach LDAC.  
> ➤ Always verify both **codec parameters** *and* **actual playback resolution**.




### 📊 LDAC Adaptive Mode Stability Matrix

| Sample Rate | Bit Depth         | Stability    | Observations                                                       |
|-------------|-------------------|--------------|--------------------------------------------------------------------|
| 44.1 kHz    | 16-bit / 24-bit   | ❌ Unstable   | Frequently drops to 606 / 303 kbps, rarely sustains 909 kbps       |
| 48 kHz      | 16-bit / 24-bit   | ✅ Stable     | Consistently ramps to and holds 990 kbps                           |
| 88.2 kHz    | 16-bit / 24-bit   | ⚠️ Moderate   | Slightly more stable than 44.1, but not reliable at high bitrates  |
| 96 kHz      | 16-bit / 24-bit   | ✅ Stable     | Ramps cleanly and holds 990 kbps with minimal adjustments          |

### ⚠️ Override Acceleration Triggers

These actions cause Samsung’s LDAC override (96 kHz / 32-bit / Default bitrate) to apply faster than BCC or any other app can intervene.

| Condition / Action                | Override Speed |
|-----------------------------------|----------------|
| Fast Pair Notification Tap        | **Instant**     |
| Quick Settings Connect            | **Instant**     |
| Bluetooth Settings UI Reconnect  | **Immediate**   |
| Developer Options not reset       | **Immediate**   |
| Media Audio Toggle (OFF → ON)                | **Immediate**       |
| Bluetooth toggled (while headphones stay on) | **Immediate**       |

## ✅ Valid LDAC Codec Preference Reset Strategies (Ranked)

| Rank | Strategy                                | Scope of Reset                            | Destructiveness        | Reliability   | Notes                                                 |
|------|-----------------------------------------|--------------------------------------------|-------------------------|----------------|--------------------------------------------------------|
| 1    | **Forget Device**                       | Full codec + pairing + override wipe       | **High**                | **100%**        | Cleanest method; use when override is stuck           |
| 2    | **Clear Music Center Data / Uninstall** | Wipes persistent LDAC override             | **Medium**              | **Very High**   | Needed if MC or Sound Connect was ever used           |
| 3    | **Hardware Reset (Headphones)**         | Wipes stored codec + multipoint config     | **Medium**              | **Very High**   | Crucial when firmware stores override                 |
| 4    | **Set LDAC to 660 kbps in Music Center**| Cancels 990/SBC override pre-uninstall     | **None**                | **High**        | Must be done *before* removing Music Center           |
| 5    | **LDAC Toggle (Off → Wait → On)**       | Triggers fresh codec negotiation           | **Low**                 | **High**        | Works well for override correction                    |
| 6    | **Reset Developer Options**             | Removes Dev Option codec override          | **Low**                 | **High**        | Always disable after setting to default               |
| 7    | **Sound Connect Software Reset**        | Clears stored app settings                 | **Low**                 | **Medium**      | Optional support step if still installed              |
| 8    | **Reset Network Settings**              | Wipes Bluetooth + Wi-Fi                    | **Very High**           | **High**        | Last resort only                                      |

> ⚠️ **Warning: Sony Music Center silently reasserts LDAC settings**  
> If you've previously used Music Center to select a specific LDAC profile (e.g., 990 kbps), it may **automatically reapply that setting** on the next reconnect — *even if the app is no longer open*.  
>  
> To prevent this, you must either:  
> - **Set LDAC to 660 kbps** in the app before uninstalling (resets override state), or  
> - **Clear app data** or uninstall Music Center **before the next pairing**  
>  
> Simply uninstalling **after** a 990 kbps override won't erase the stored configuration from the headphone firmware.



## ❌ False Resets & Misleading Behaviors (Do Not Work)

| # | Feature                     | Why It Seems Useful                         | Why It Fails                                               | Verdict                                            |
|---|-----------------------------|---------------------------------------------|-------------------------------------------------------------|----------------------------------------------------|
| 1 | **HD Audio (LDAC) Toggle**  | Appears to trigger codec renegotiation      | Just re-enables Samsung's default LDAC override             | **False reset** — reasserts override profile       |
| 2 | **Input Device Toggle**     | Seems like it influences audio routing      | Only toggles HFP profile (mic/remote), not A2DP             | **Unrelated** — no codec impact                    |
| 3 | **Fast Pair Notification Tap** | Looks like a clean connection method     | Requires manual confirmation, then applies override profile | **Misleading** — not auto, and reuses LDAC state   |
| 4 | **Music Center Disconnect Button** | Feels like a proper disconnection method     | Only performs a UI-level disconnect, no codec reset or override clear | **Misleading** — does not renegotiate or reset LDAC |


## fast pair
### ⚠️ Fast Pair Auto-Connect — The Hidden Override Trigger

Fast Pair doesn’t just work when you tap the notification. Even if you ignore it, it can automatically reconnect your headphones in the background using Google Play Services — often **faster than Samsung’s own auto-connect**.

This hidden auto-connect is **aggressive and high priority**, meaning it often **beats BCC to the LDAC handshake** and allows the **Samsung override (96 kHz / 32-bit / default bitrate)** to reassert itself silently.

### 🧠 Comparison: Fast Pair Auto-Connect vs. Samsung Auto-Connect

| Feature                        | **Fast Pair Auto-Connect**              | **Samsung Auto-Connect**                 |
|-------------------------------|----------------------------------------|------------------------------------------|
| **Initiator**                 | Google Play Services (Fast Pair logic) | Samsung Bluetooth Stack                  |
| **Connection Speed**          | **Very Fast** (sub-second)             | Fast (1–2 seconds)                       |
| **Override Triggered**        | **Yes** — immediately upon connection  | **Yes** — slightly delayed               |
| **BCC Profile Blocked**       | **Yes** — BCC profile gets ignored     | **Sometimes** — BCC may still apply      |
| **Notification Required**     | No — can trigger silently              | No — triggers automatically on power-on  |
| **Detectable in UI?**         | ❌ No — occurs without user action     | ✅ Yes — visible device connect prompt    |
| **Best Defense Strategy**     | Disable Nearby Devices in Play Services | Delay auto-connect, use SBC or BCC delay |
| **Risk of Override**          | **High** — triggers instantly          | **Medium** — timing window for bypass    |

### 🔌 Bluetooth Connection Methods — Ranked by Override Speed and BCC Compatibility

This table expands on the common ways to connect your headphones (from NFC to Fast Pair) and explains how quickly each one applies Samsung’s LDAC override — and whether Bluetooth Codec Changer (BCC) has time to intervene.

| Method                                | Override Speed   | Can BCC Win? | Notes                                                                 |
|---------------------------------------|------------------|--------------|-----------------------------------------------------------------------|
| **Fast Pair Notification Tap**        | **Instant**      | ❌ No         | Manual confirmation triggers override immediately                     |
| **Fast Pair Auto-Connect**            | **Instant**      | ❌ No         | Silent reconnect via Google Services — override injected early        |
| **Quick Settings UI Connect**         | **Instant**      | ❌ No         | LDAC override occurs before BCC can apply profile                     |
| **Bluetooth Settings UI Connect**     | **Immediate**    | ❌ No         | Slightly slower than Quick Settings, but still too fast for BCC       |
| **NFC Tap-to-Pair**                   | **Immediate**    | ❌ No         | Starts override as part of pairing — no room for intervention         |
| **Manual Headphone Power-On First**   | ⚠️ Moderate       | ✅ Sometimes  | BCC may win if auto-switch is fast enough                             |
| **Headphones On → Manual Connect**    | ⚠️ Moderate       | ✅ Sometimes  | Safer if Fast Pair and UI triggers are avoided                        |
| **App-Initiated Connect (e.g. BCC)**  | ✅ Delayed        | ✅ Yes        | Cleanest connection — override can be bypassed reliably               |
| **Tasker + BCC Dual Profile**         | **Custom**       | ✅ Always     | Automation with two chained LDAC profiles — defeats firmware override |


## ✅ Complete LDAC Codec Reset Matrix (v2025)

---

### 🎧 HEADSET-SIDE RESETS (Sony WH-1000XM Series)

| #  | Method                                                       | Resets Codec Profile | Resets Pairing | Notes                                                                 |
|----|--------------------------------------------------------------|----------------------|----------------|-----------------------------------------------------------------------|
| 1  | Hardware Reset (Power + Custom 7s)                           | ✅ Yes               | ✅ Yes         | Deepest reset — wipes firmware-level codec and pairing data          |
| 2  | Software Reset via Headphones Connect (System > Initialize) | ✅ Yes               | ❌ No          | Clears LDAC quality, EQ, app prefs — pairing info remains            |
| 3  | Leave Powered Off 10+ sec After Clean Handshake             | ⚠️ Conditional        | ❌ No          | Stores most recent profile (SBC or LDAC) — used for exploits, not reset |
| 3b | Power Off Headphones After Clean SBC or LDAC 16-bit Handshake | ✅ (Firmware)        | ❌ No          | Preserves clean handshake profile — crucial for override bypass      |

---

### 📱 PHONE-SIDE RESETS (Samsung)

| #  | Method                                                     | Resets Codec Profile | Resets Samsung Override | Notes                                                                 |
|----|------------------------------------------------------------|----------------------|--------------------------|-----------------------------------------------------------------------|
| 4  | Reset Network Settings (General Management > Reset)       | ✅ Yes               | ✅ Yes                   | Resets Bluetooth, Wi-Fi, and override stack                          |
| 5  | Forget Bluetooth Device                                   | ❌ Partial           | ❌ Partial               | Removes pairing, but override often persists                         |
| 6  | Set Codec to SBC in Developer Options                     | ✅ Yes               | ✅ Yes                   | "Default" not available — SBC forces override flush                  |
| 7  | Disable Dev Options After SBC Set                         | ✅ Yes               | ✅ Yes                   | Locks in override removal, blocks LDAC reapplication                |
| 7b | Intermediate Profile Switch (SBC → LDAC 16-bit → LDAC 990 via BCC) | ✅ Yes        | ✅ Yes                   | Bypasses Samsung override with staged codec negotiation              |
| 8  | Disable HD Audio (in BT settings)                         | ❌ No                | ⚠️ May disable session     | Only disables session — override logic survives                      |

---

### 📲 APP-LEVEL RESETS (Sony, BCC, Google Services)

| #  | Method                                                      | Resets Codec Profile | Blocks Future Override | Notes                                                                  |
|----|-------------------------------------------------------------|----------------------|------------------------|------------------------------------------------------------------------|
| 9  | Clear Storage: Sony Music Center                           | ✅ App-side           | ✅ Yes                | Prevents stored LDAC quality (660/990) from being reapplied            |
| 10 | Clear Storage: Headphones Connect (Sound Connect)          | ✅ App-side           | ✅ Yes                | Wipes EQ and LDAC behavior control                                     |
| 11 | Clear Storage: Bluetooth Codec Changer (BCC)               | ✅ Profile mappings   | ❌ No                 | Clears profile switch logic and XML config                             |
| 12 | Uninstall Sony/BCC Apps (no data cleared)                  | ❌ No                | ❌ No                 | Prevents future app changes — existing override remains                |
| 13 | Clear Saved Devices (Settings > Google > Devices & Sharing) | ❌ No               | ✅ Fast Pair metadata | Removes cloud pairing + override profile sync                          |
| 14 | Disable Nearby & Scanning (Location Settings)              | ❌ No                | ✅ Reduces interference | Blocks Google Services override triggers                               |

---

### 🔁 TEMPORARY / VOLATILE SESSION CLEARS

| #  | Method                                               | Resets Codec Profile | Duration         | Notes                                               |
|----|------------------------------------------------------|----------------------|------------------|-----------------------------------------------------|
| 15 | adb shell am force-stop com.android.bluetooth       | ❌ No                | Until reconnect  | Ends A2DP session — clears temp override            |
| 16 | adb shell am force-stop com.google.android.gms       | ❌ No                | Until restart    | Interrupts Nearby + Fast Pair syncing              |
| 17 | Airplane Mode Toggle + Reboot                        | ❌ No                | Temporary        | Clears memory stack, not override                  |
| 18 | Reconnect Using Fast Pair (not BT settings)          | ⚠️ Variable           | Variable         | Avoids override if done post-reset                 |
| 19 | Bluetooth Input Toggle (gear icon > input device)    | ❌ No                | ❌ No            | No effect on LDAC override                         |
---



### 📶 Fast Pair Safe Timing Strategy

You *can* use Fast Pair **once** during initial pairing:

1. ✅ Pair via Fast Pair  
2. ✅ Immediately train your LDAC profile  
3. ❗ After initial pairing and training:
   - Permanently **deny Nearby Devices permission** to Google Play Services  
   - Disable “Automatically save devices”

> ⚠️ **Important:**  
> Keep Nearby Devices permission **enabled during pairing** so Fast Pair works correctly.  
> Deny it **immediately after pairing** to prevent Google Play Services from silently reconnecting and forcing Samsung's LDAC override profile. This also improves LDAC 990 kbps stability significantly.

---

## 🧠 Why This Is the Final Form

You’re no longer *overriding* Samsung — you’re **commanding** it.

- No more reapplying LDAC manually  
- No reliance on Developer Options  
- No false UI readings

✅ One-time setup  
✅ Persistent memory  
✅ Bit-perfect playback — *every time*

> You didn’t just beat the override.  
> You **rewired it to obey you.**


---------


## 🧼 Manual Override Hijack (Legacy Method — For Non-Automated Setups)

Use this sequence to **force Samsung to adopt your LDAC profile** permanently using Fast Pair — but without letting Google sabotage the handshake.

---

### 🔁 Step 1: Full Reset (Wipe Samsung’s LDAC Override Memory)

These steps **eliminate all stored codec profiles**, Fast Pair metadata, Developer Option overrides, and app-based LDAC reassertions:

1. **Connect your headphones via Bluetooth Settings**  
   *(Avoid Quick Settings and NFC. Fast Pair will be used **after** this reset.)*

2. **Set codec to SBC in Developer Options**  
   `Settings → Developer Options → Bluetooth Audio Codec → SBC`  
   *(Must be done **while headphones are connected**. This wipes Samsung’s override memory.)*

3. **Disable Developer Options**  
   *(Still while connected — finalizes override wipe and prevents silent reassertion.)*

4. **Power off headphones**

5. **Hardware Reset the headphones** → `Power + Custom for 7 seconds`  
   *(Wait for “Pairing” voice cue. Power off again immediately.)*

6. **Reset Network Settings**  
   `Settings → General Management → Reset → Reset Wi-Fi and Bluetooth`

7. **Clear App Storage**  
   - Sony Music Center  
   - Headphones Connect  
   ⚠️ *Do **not** clear BCC data — keep your profiles and automation intact*

8. **Forget Bluetooth Device**  
   `Settings → Connections → Bluetooth → [Your Headphones] → Forget`

9. **Clear Fast Pair Metadata**  
   `Settings → Google → Devices & Sharing → Saved Devices → [Headphones] → Remove`

9a. **Temporarily enable Nearby Devices for Google Play Services**  
   - `Settings → Apps → Google Play Services → Permissions → Nearby Devices`  
   - `This enables Fast Pair to show up during pairing`

10. **Keep Nearby Devices permission enabled during pairing** so Fast Pair can function.

11. **Reboot your phone**  
    *(This ensures Developer Option overrides and Fast Pair metadata are fully flushed before pairing again.)*

12. **Pair your headphones using Fast Pair**.

---

### 🚫 Immediately Disable Google Play Services Override Sync (GUI Method)

To prevent Google Play Services from reasserting old LDAC profiles or syncing Fast Pair override metadata, complete the following steps **immediately after pairing**:

#### ✅ Disable Nearby Devices Access
1. Go to:  
   `Settings → Apps → Google Play Services → Permissions`
2. Tap **Nearby Devices**
3. Select **Deny**
4. Confirm by selecting **Don’t allow**

> This prevents silent reconnects and cloud override injection.

#### ✅ Turn Off Auto-Save for Fast Pair
1. Go to:  
   `Settings → Google → Devices & Sharing → Saved Devices`
2. Tap the **⋮ (3-dot menu)** in the top right
3. Tap **"Turn off Automatically Save Devices"**

> This stops Google from syncing override profiles to your account.

#### ✅ Remove Stored Fast Pair Metadata
1. In the same **Saved Devices** screen, tap your headphone entry
2. Tap **Remove device**

> This ensures old override data isn’t re-applied during pairing.

#### ✅ Disable Google Location Accuracy
1. Go to:  
   `Settings → Location → Location Services → Google Location Accuracy`
2. Turn **OFF** the toggle

> This disables Wi-Fi and BLE scanning that can interfere with LDAC and audio stability.

---

✅ These steps must be done **immediately after pairing** to prevent override corruption from Google Play Services during your first handshake and training cycles.

13. **Enable LDAC in Bluetooth Settings**  
    `Settings → Connections → Bluetooth → ⚙️ → Enable “HD audio: LDAC”`  
    > This toggle is required. Without it, LDAC won’t activate and your BCC profile will not apply.

14. **Immediately after successful pairing and training your LDAC profile:**  
    Permanently **deny Nearby Devices permission** to Google Play Services to block silent auto-connect and override injection:  
    - `Settings → Apps → Google Play Services → Permissions → Nearby Devices`  
    - Select **Deny** and **Do not allow** requests again

15. **Disable “Automatically save devices”**  
    - `Settings → Google → Devices & Sharing → Saved Devices`  
    - Tap ︙ → **Turn off "Automatically save devices"**

16. **Toggle Airplane Mode on/off** to finalize memory flush

> ✅ Your device is now ready to accept and retain your custom LDAC handshake.


---

### ✈️ Airplane Mode Training

Ensure Airplane Mode **fully disables Bluetooth**:

1. Turn off Bluetooth  
2. Enable Airplane Mode  
3. Do **not** re-enable Bluetooth  
4. Disable Airplane Mode again

> Android will now remember: *Airplane Mode = Bluetooth OFF*

---

### ✅ Pre-Pairing BCC Setup (Can Be Done Before Connecting)

Before pairing the headphones, open the BCC app and configure:

1. **Enable "Main Background Service"**  
   *(Keeps BCC active after Bluetooth connection)*

2. **Disable "2-Step Switching"**  
   *(Prevents handshake desyncs and GUI bugs)*

3. **Enable "Ultra Advanced Automation Apps Integration"**  
   *(Allows BCC to be fully controlled via Tasker or other automation tools for precise codec switching)*

> ⚠️ **Warning:** If BCC’s Auto Switch is enabled at this stage, it may race with Samsung’s override and cause a desync.  
> **Disable Auto Switch** until after your LDAC profile is fully trained and locked in.

---

## ⚡ Why Fast Pair Is the Ultimate Weapon

Forget the outdated advice — **Fast Pair isn’t the problem**.  
It’s the *key* to taming Samsung’s override and enforcing your exact LDAC profile.

---

### ✅ Why Fast Pair Beats Manual Pairing

| Feature / Behavior                        | ✅ **Fast Pair**                          | ❌ **Manual Bluetooth Pairing**          |
|------------------------------------------|------------------------------------------|------------------------------------------|
| Override injection timing                | ✅ Delayed — override happens *after* pairing | ❌ Instant — override injects during pairing |
| Time window to hijack codec              | ✅ Yes — clean hijack possible            | ❌ None — Samsung locks in early         |
| SBC→LDAC handshake exploit compatibility | ✅ Perfectly timed                        | ⚠️ Often too late                        |
| BCC or UAPP profile takeover             | ✅ Yes — reliable                         | ⚠️ Rarely wins override race             |
| Override persistence risk                | ✅ Low — if cloud sync blocked            | ❌ High — sticks until full wipe         |
| Tasker and automation support            | ✅ Cleanly automatable after pairing      | ⚠️ Unstable if override dominates early  |
| Developer Options required?              | ❌ No — fully profile-based               | ⚠️ Often needed to fix override damage   |
| Compatible with Auto Switch (after lock) | ✅ Yes — if enabled *after* profile lock  | ⚠️ Risk of conflict at connect time      |
| Can override stay locked without BCC?    | ✅ Yes — Samsung reuses trained profile   | ❌ No — override often resets to 96kHz   |

---

### 🧠 Real Behavior Summary (Updated)

When using **manual pairing via Bluetooth settings**, Samsung injects its override profile (96kHz / 32-bit / Adaptive) *before* you get a chance to assert your own settings. This leads to:

- Locked-in default LDAC behavior  
- Bitrate dropouts  
- Desynced UI info  
- BCC and app profiles failing silently

✅ The only reliable way to bypass this:

- Use **Fast Pair** for the initial handshake  
- Train your **LDAC profile immediately**  
- Then lock it in using **Bluetooth Codec Changer (BCC)** with **Tasker-based profile switching**

> 🧠 **Important:**  
> Fast Pair **only works reliably** when used **with BCC and Tasker automation**.  
> Without Tasker handling profile switching and LDAC reapplication, Samsung’s override may still reassert itself — especially during reconnects or UI-triggered connections.

---

### 🔐 Final Verdict (Updated)

> Manual Bluetooth pairing is a **trap**.  
> You don’t control the handshake timing — Samsung does.  
> You lose before LDAC even starts.

✅ With **Fast Pair + BCC + Tasker**:

- You win the timing window  
- You enforce **your custom LDAC profile**  
- You gain **persistent, override-free playback**  
- You no longer need Developer Options or risky UI workarounds  
- Samsung’s override stack becomes **your ally**, not your enemy

---

**Fast Pair isn’t just a workaround — it’s the foundation for a controlled LDAC environment when paired with BCC and Tasker.**
## 🔁 LDAC Profile Training: Why One Handshake Isn’t Always Enough

Even with a perfect Fast Pair + BCC setup, **one clean LDAC handshake doesn’t always lock in your custom profile** — especially on Samsung devices and Sony XM-series headphones.

Both Samsung’s override cache and Sony’s headset firmware **need confirmation over multiple sessions** to permanently apply your desired codec.

---

### 🧠 Why Multiple Training Cycles Help

| System              | What It Watches                                  | When It Stores Your Profile               |
|---------------------|--------------------------------------------------|-------------------------------------------|
| 🎧 Sony Headphones  | Active LDAC session with audio playback          | After ~10+ sec of playback, then power-off |
| 📱 Samsung Stack    | Initial LDAC sessions after pairing              | After 2–3 consistent, stable reconnects    |
| 🔁 Google Services  | Fast Pair & override sync data                   | If not blocked, may reassert old profiles |

---

### ✅ Recommended Training Loop (Repeat 2–3 Times)

1. **Power on headphones manually**
2. **Let BCC/Tasker connect automatically**
3. Confirm correct LDAC profile (via GUI or ADB)
4. **Start playback immediately** (UAPP/Neutron preferred)
5. Let music run for **15–30 seconds**
6. **Power off headphones while LDAC is still active**
7. Wait 10+ seconds
8. Repeat 2–3 times

## 🚫 Full Google Play Services Lockdown (GUI-Only Method)

To prevent Google Play Services from interfering with your LDAC profile (via Fast Pair, Nearby Devices, and cloud override), follow these GUI-only steps:

### ✅ Step 1: Disable Nearby Devices Access
1. Go to:  
   `Settings → Apps → Google Play Services → Permissions`
2. Tap **Nearby Devices**
3. Select **Deny**
4. Confirm by selecting **Don’t allow**

> This prevents silent reconnects and cloud override injection.

---

### ✅ Step 2: Turn Off Auto-Save for Fast Pair
1. Go to:  
   `Settings → Google → Devices & Sharing → Saved Devices`
2. Tap the **⋮ (3-dot menu)** in the top right
3. Tap **"Turn off Automatically Save Devices"**

> This stops Google from syncing override profiles to your account.

---

### ✅ Step 3: Remove Stored Fast Pair Metadata
1. In the same **Saved Devices** screen, tap your headphone entry
2. Tap **Remove device**

> This ensures old override data isn’t re-applied during pairing.

---

### ✅ Step 4: Disable Google Location Accuracy
1. Go to:  
   `Settings → Location → Location Services → Google Location Accuracy`
2. Turn **OFF** the toggle

> This disables Wi-Fi and BLE scanning that can interfere with LDAC and audio stability.
---

✅ These changes **completely prevent Google Play Services from reasserting override profiles or reconnecting silently in the background** — while still keeping the Play Store and apps fully functional.

## 🔁 Multipoint LDAC Overview

The **Sony WH-1000XM5** supports **multipoint Bluetooth**, allowing connection to **two devices simultaneously**.

Typically, this works seamlessly because:
- **Windows** defaults to **SBC or AAC**
- **Android** uses **LDAC**

➡️ This prevents codec conflicts.

---

However, **advanced users** may configure **LDAC on both devices** — for example:
- Android **and** Windows  
- Or even **two Android devices**

### ⚠️ This introduces a challenge:
> **Can we build a stable configuration where both devices use LDAC without triggering codec fallback, stutters, or renegotiation?**

A stable dual-LDAC setup requires careful tuning of:
- Codec behavior  
- Volume and AVRCP versions  
- Playback format

---

## 🛠️ Setup Prerequisites

To ensure a smooth and stable LDAC multipoint experience:

### ✅ AVRCP 1.6 on Both Devices

- **On Android:**
  - Enable AVRCP 1.6 once in Developer Options
  - **It persists even after Developer Options are turned off**
## 🔄 LDAC Multipoint Stability Factors

| Factor                       | Configuration Options                                               | Related Notes or Interactions                                      |
|------------------------------|---------------------------------------------------------------------|--------------------------------------------------------------------|
| Codec settings               | Mirror **LDAC settings** on both devices                            | Prevents codec renegotiation or fallback to SBC/AAC                |
| Playback format              | Match **sample rate** and **bit depth** (e.g., 96 kHz / 24-bit)     | Avoids stutters and DSP resync delays                              |
| Absolute Volume              | Use **ON on Windows**, **OFF on Android**                           | Maintains volume sync on Windows and codec control on Android      |
| Volume level                 | Set the **same volume** on both devices (e.g., 85%)                 | Prevents loudness jumps and smoothens handoff behavior             |
| AVRCP version                | Use **1.6 on both Android and Windows**                             | Ensures faster media control switching and improved metadata sync  |
| Playback state coordination  | **Pause playback** on the inactive device                           | Prevents LDAC renegotiation and session fight during handoff       |
| BCC timing logic             | Add **intermediate profile** (e.g., SBC or LDAC 16-bit) before final LDAC | Ensures clean codec handshake and profile lock-in          |

> ⚠️ **Note:** Switching from Absolute Volume OFF to ON (or vice versa) on **Windows** may require **re-pairing** the headphones for the setting to take full effect.

> ⚠️ **Implementation details** of AVRCP 1.6 may vary between platforms. All testing was done using AVRCP **1.6**.

---

## 🎧 Multipoint AV/AVRCP Combination Matrix (LDAC on Both Devices)

| # | Setup Type         | Device A | Device B | Absolute Volume (A / B) | AVRCP Version (A / B) | Notes                                                                 |
|---|--------------------|----------|----------|--------------------------|------------------------|------------------------------------------------------------------------|
| 1 | Android + Android  | Android  | Android  | ON / ON                  | 1.6 / 1.6              | Switching devices is without stuttering                                |
| 2 | Android + Android  | Android  | Android  | OFF / OFF                | 1.6 / 1.6              | Switching devices is with slight stuttering and switching isn't seamless |
| 3 | Android + Android  | Android  | Android  | OFF / ON                 | 1.6 / 1.6              | Switching devices is with slight stuttering and switching isn't seamless |
| 4 | Android + Windows  | Android  | Windows  | OFF / ON                 | 1.6 / 1.6              | Switching devices is with slight stuttering and switching isn't seamless |
| 5 | Android + Windows  | Android  | Windows  | ON / ON                  | 1.6 / 1.6              | Switching devices is without stuttering                                |
| 6 | Android + Windows  | Android  | Windows  | OFF / OFF                | 1.6 / 1.6              | Switching devices is with slight stuttering and switching isn't seamless |
| 7 | Android + Windows  | Android  | Windows  | ON / OFF                 | 1.6 / 1.6              | Switching devices is with slight stuttering and switching isn't seamless |
| 8 | Android + Android  | Android  | Android  | ON / OFF                 | 1.6 / 1.6              | Switching devices is with slight stuttering and switching isn't seamless |

> ⚠️ **All configurations above were tested with AVRCP 1.6**. Other AVRCP versions were not evaluated and may yield different switching or stability behavior.



## Windows volume level with AV on on both devices
| Android Volume Step (%) | Approx. Matching Windows Volume (%) | Notes                                 |
|--------------------------|--------------------------------------|---------------------------------------|
| 0%                       | 0%                                   | Muted                                 |
| 6%                       | 5%                                   | Very quiet                            |
| 13%                      | 10–12%                               | Low; not recommended for LDAC         |
| 20%                      | 15–20%                               | May trigger signal attenuation        |
| 26%                      | 25%                                  | Soft, below stable handoff threshold  |
| 33%                      | 30–35%                               | Minimum for casual listening          |
| 40%                      | 40%                                  | Noticeable gain start                 |
| 46%                      | 50%                                  | Beginning of stable range             |
| 53%                      | 60%                                  | Entry-level for balanced switching    |
| 60%                      | 68–70%                               | Near optimal for casual fidelity      |
| 66%                      | 75–78%                               | Reliable balance for most setups      |
| 73%                      | 80–85%                               | Recommended baseline for switching    |
| 80%                      | 88–90%                               | Ideal for LDAC 990 fidelity matching  |
| 86%                      | 92–94%                               | High detail, balanced control         |
| 93%                      | 96–98%                               | Nearly full-scale                     |
| 100%                     | 100%                                 | Bit-perfect full loudness             |


# 🔧 Alternative A2DP Driver – Android + Windows LDAC Multipoint Notes

## 🛒 Installation Steps

1. **Buy license from:** [https://www.bluetoothgoodies.com/a2dp/](https://www.bluetoothgoodies.com/a2dp/)
2. **Install the stable version of the software**
3. **Configure the following:**
   - Install the Alternative A2DP Driver
4. **Optionally reboot after**

---

## 🎛 Optimal LDAC Settings (Windows A2DP Driver)

### 🎵 Media Focus / CD Quality (for apps with exclusive access via WASAPI)

- **Sample Rate:** 44.1 kHz or 48 kHz  
- **Bit Depth:** 16-bit  
- **Stereo Mode:** Stereo  
- **Encode Quality:** High (HQ)  
- **Adaptive Bitrate:** Disabled

---

### 🧠 High-Res Playback (for apps with exclusive access via WASAPI)

- **Sample Rate:** 44.1 / 48 / 88.2 / 96 kHz  
- **Bit Depth:** 24-bit  
- **Stereo Mode:** Stereo  
- **Encode Quality:** High (HQ)  
- **Adaptive Bitrate:** Disabled

---

### 🎯 Bit Depth-Only Resampling (for high-res playback in exclusive mode)

- **Only get depth rate resampling when playing high res**
- Ensure exclusive mode is ON
- Useful for apps that properly handle bit-perfect streams

---

## 🎶 Spotify Specific Behavior

- **Spotify doesn’t support WASAPI**
- Use 44.1 kHz, 16-bit LDAC profile
- Avoid enabling exclusive mode or adaptive bitrate
- Stick to media-optimized LDAC profile

---

## 🔄 AVRCP Behavior and Multipoint Issues

- **AVRCP 1.6 on both Windows 11 and Android causes stuttering**
  - Stutter occurs shortly after unlocking the phone
- Even if **Windows is playing and using AVRCP 1.6 by itself**,  
  it can stutter when Android is connected but idle — just because multipoint is active.
- **Windows randomly pauses without reason** during multipoint playback.
- **With AVRCP 1.5 on Windows**, Android still sees Windows as playing audio.
- **More stuttering occurs when unlocking the phone** if AVRCP is switched to 1.5.

---

### 🛑 Do Not Use Registry AVRCP Edits

Modifying `AvrcpTargetVersion` in the Windows Registry has no effect on AVRCP behavior with modern stacks (including Bluetooth Goodies). Windows 11 and 10 always uses AVRCP 1.4–1.6, and this cannot be changed manually. Metadata visibility and switching latency should be used to infer behavior instead.
## 🔁 Multipoint + LDAC Dual Control Behavior

- With AVRCP 1.6 on both devices:
  - You can **press play on both Android and Windows**
  - Audio from both can **attempt to play at the same time**
  - If **Android is the active source**, stuttering is minimal
  - If **Windows is the active source**, Android stops playback

- Even **Windows randomly pauses** without any user interaction during multipoint
- Android **still sees that Windows is playing audio**, even if Windows uses AVRCP 1.5

---

### 🧠 Controlling AVRCP Version? You Can’t — Unless You Change Hardware

The **only reliable way** to influence the AVRCP version used in **Windows** is to **buy a Bluetooth adapter with a specific Bluetooth version**.

- Windows **ignores** the `AvrcpTargetVersion` registry key on all modern Bluetooth stacks (including the Bluetooth Goodies A2DP driver).
- You **cannot downgrade or upgrade AVRCP** via software alone — the stack negotiates the **highest mutually supported version** between the dongle and the device.



| Bluetooth Version | Default AVRCP Version(s) |
|-------------------|--------------------------|
| 2.0 + EDR         | 1.0–1.3                  |
| 2.1 + EDR         | 1.3–1.4                  |
| 3.0 + HS          | 1.4                      |
| 4.0               | 1.4–1.5                  |
| 4.1               | 1.5                      |
| 4.2               | 1.5                      |
| 5.0               | 1.6                      |
| 5.1               | 1.6                      |
| 5.2               | 1.6                      |
| 5.3               | 1.6                      |
| 5.4               | 1.6                      |


> 🔧 **Note**: AVRCP negotiation is unidirectional — **the lower of the two devices' supported versions wins**.




---





## 🧠 LDAC Control Roles

> With correct setup:
> - **Windows acts as the dominant LDAC controller**
> - **Android behaves as an intelligent follower**
> - **UAPP on Android** yields to Windows even when playing in high-res exclusive mode

This behavior has been validated across:
- Android 13 / 14 (One UI variants)
- Windows 11 with AVRCP 1.5/1.6
- LDAC fixed and adaptive configurations (mirrored)

---

## 📶 LDAC Multipoint Confirmation

> Everything works in multi-control.  
> All combinations of **fixed and adaptive** LDAC modes are valid  
> **as long as both sides are mirrored.**

- Do **not use 32-bit on Android**
  - It adds **no audible benefit**
  - It introduces instability in codec negotiation

---

## 🎉 Multipoint Works "As Intended" — But Isn't

> LDAC Multipoint is **supported** — and it works.

- Android and Windows can both stream audio over LDAC
- You can **press play on both**, and both streams will try to take control
- Audio stability depends on who initiates playback
- This works despite not being **intended behavior** by most manufacturers

---

## 🧪 Final Test Results

- When **Absolute Volume is ON**, and **Windows is playing**:
  - You can still **press play on Android**
  - Playback from Android stops silently
  - No override happens unless Android is prioritized first
- **Very little stuttering** when Android plays while Windows is still active
- **Perfect behavior when Android is the audio source and Windows is idle**

> “It only stutters **very little**”

---

## ✅ Final Takeaways

- ✅ True **multi-control LDAC** is possible between Android and Windows
- ✅ Works best when **Android is the primary source**
- ✅ **Windows should use AVRCP 1.5**, but 1.6 works with more care
- ✅ **Absolute Volume ON (Windows)** / **OFF (Android)** gives best sync
- ✅ Android **intelligently yields** if Windows takes codec control
- ✅ Pressing play on both works — **if Android is the one actually playing**
- ✅ Mirrored profiles (fixed/adaptive) = most stable
- ❌ Avoid 32-bit audio on Android — unnecessary and buggy


### 🔄 Windows + Android Multipoint Tips (LDAC / Bluetooth)

#### ✅ View Codec Info via Android's Sound Assistant or Sound Connect While Playing from Windows
When using **multipoint with Windows as the active audio source**, you can still:
- Open **Sound Assistant** or **Sound Connect** on your Android device  
- See the **active Bluetooth audio codec** (e.g. LDAC)  
- Monitor connection state, device battery, and metadata (e.g. track title)

🧠 **Why this works**: Android stays connected over control and data channels, even if it’s not playing audio. This allows apps like Sound Connect to report codec info live.

#### 📌 Fix Device Selection in Sound Connect During Multipoint
While LDAC multipoint is active:
- Open **Sound Connect** on Android  
- **Pin** or **lock** the headphones to Android within the app  
- This ensures consistent codec visibility and device control

> 🔒 Useful when switching sources frequently — it prevents Android from deprioritizing the device in the UI or misreporting connection status.


> 🧠 **AVRCP 1.5 is more stable than 1.6 on Windows — period.**
>
> Using AVRCP 1.6 on Windows causes:
> - Random pauses
> - Playback interruptions
> - Stuttering during multipoint with Android
>
> ✅ **AVRCP 1.5 avoids these issues** by keeping Windows passive in media control and respecting Android’s LDAC session.
>
> 🔄 **Note:** Short, *predictable stuttering* may still occur during device switching — but it **recovers instantly without codec fallback**.
>
> 👉 To force AVRCP 1.5, use a **Bluetooth 4.2 (or lower)** dongle.

---




## Absolute Volume
Absolute Volume on means it has to hit 85% of volume minimal 

## Music Center
> 🎧 This table applies when using the volume slider inside the **Sony | Music Center** app with **Absolute Volume OFF**.  
> Android system volume is ignored, and all volume control is handled by the headphones.

| Step | Approx. % Volume |
|------|------------------|
| 1    | 3%               |
| 2    | 7%               |
| 3    | 10%              |
| 4    | 13%              |
| 5    | 17%              |
| 6    | 20%              |
| 7    | 23%              |
| 8    | 27%              |
| 9    | 30%              |
| 10   | 33%              |
| 11   | 37%              |
| 12   | 40%              |
| 13   | 43%              |
| 14   | 47%              |
| 15   | 50%              |
| 16   | 53%              |
| 17   | 57%              |
| 18   | 60%              |
| 19   | 63%              |
| 20   | 67%              |
| 21   | 70%              |
| 22   | 73%              |
| 23   | 77%              |
| 24   | 80%              |
| 25   | 83%              |
| 26   | 87%              |
| 27   | 90%              |
| 28   | 93%              |
| 29   | 97%              |
| 30   | 100% (max)       |

---

> ⚠️ **Codec Behavior Note**  
> The **Sony | Music Center** app can only switch between:
>
> - **SBC**
> - **LDAC 96 kHz 32-bit Adaptive**
> - **LDAC 96 kHz 32-bit 909/990 kbps (Fixed)**
>
> Once LDAC is locked by Music Center, **external tools like Bluetooth Codec Changer (BCC) and USB Audio Player PRO (UAPP) cannot override it** unless the codec is renegotiated via disconnection or an SBC handshake.
>
> Opening Music Center **after a profile has been set by another app** will immediately trigger a renegotiation to the **currently selected LDAC mode inside the app**, overriding any prior configuration—even if BCC or UAPP had previously succeeded.


> 🧠 **Default Behavior:**  
> If you install and open Sony Music Center **without changing any codec settings**, it defaults to:
>
> - **LDAC Adaptive** mode  
> - **96 kHz** sample rate  
> - **Bitrate** dynamically adjusts between **330 / 660 / 990 kbps**, depending on link quality
>
> 📌 This means Music Center does **not** apply a fixed bitrate by default — it applies a **96 kHz Adaptive profile**, and lets LDAC decide between 330, 660, or 990 kbps in real time.






## 🔍 Additional Notes on Codec Storage and LDAC Behavior

- **LDAC quality settings written by Music Center are not applied immediately.**  
  The selected codec (e.g., 990 kbps) is stored in the headset firmware, but it only takes effect **on the next Bluetooth connection**.  
  Even then, due to the LDAC bug, the codec may **appear correct in dumpsys or the GUI** but still stream at the wrong quality.  
  ➤ Always verify actual playback quality — never trust visuals alone.

- **No delay is needed to store a codec profile.**  
  After applying SBC or a 16-bit LDAC intermediate profile (via Music Center or BCC), you can **power off the headphones immediately**.  
  The setting is written to firmware instantly — no need to wait 10+ seconds.  
  ➤ This enables faster and more reliable handshake training for override bypass strategies.

### 🔄 LDAC Priority Setting Impact

> 🎛️ Music Center's LDAC priority setting directly affects whether BCC can override the codec.

- **Priority on Sound Quality**  
  Forces LDAC (usually 990 kbps or 96 kHz Adaptive) via GATT.  
  ➤ This setting **locks LDAC**, and BCC **cannot override** it — even if Auto Switch or Intermediate profiles are enabled.

- **Priority on Stable Connection**  
  Defaults to SBC or fallback modes.  
  ➤ This setting **releases LDAC control**, allowing BCC or app-based renegotiation (UAPP, Neutron, etc.) to fully succeed after handshake.

🔁 **To regain override control:**  
Change Music Center to **Stable Connection**, then:
1. Disconnect the headset.
2. Optionally apply SBC via Developer Options or Music Center.
3. Reconnect using BCC or Fast Pair (with override bypass).

### 🛰️ Background Behavior — Music Center Codec Reassertion

Sony Music Center doesn’t just apply codec settings once — it registers a background **Bluetooth service** that monitors codec state and may silently reassert its LDAC mode when:

- 🔁 Headphones reconnect  
- 🎚️ LDAC toggle is changed in system settings  
- 🎵 A playback app (like UAPP or Neutron) triggers a new LDAC session

📌 Even if you **swipe the app away**, these background receivers **remain active**.

---

### ✅ How to Stop Music Center from Overriding LDAC

1. **Force stop** the app  
   `Settings → Apps → Sony Music Center → Force Stop`

2. *(Optional but recommended)*  
   **Clear app storage** to remove any stored LDAC quality settings

3. **Reconnect** or **power cycle** the headphones  
   → Ensures your BCC or UAPP profile isn’t silently overwritten

---

### 🟡 What About the “Disconnect” Button?

> The **“Disconnect”** button inside Sony Music Center temporarily halts codec control during that session  
> — but it does **not stop** future LDAC overrides or wipe stored profiles.

✅ Useful for quick testing  
❌ Not a full solution — use **Force Stop** if you want permanent override prevention





## 🎧 Headphone Firmware Storage Behavior (Sony WH-1000XM5)

Sony’s WH-1000XM5 can **store only a limited set of codec settings** in firmware between power cycles.

| Parameter                      | Stored in Firmware | How It's Set                      | Persistent? | Notes                                                                 |
|-------------------------------|--------------------|-----------------------------------|-------------|-----------------------------------------------------------------------|
| **Codec** (SBC / LDAC)        | ✅ Yes             | Last active codec at power-off    | ✅          | The most recently used codec is remembered.                          |
| **LDAC Quality Mode**         | ✅ Yes             | Only via Sony Music Center        | ✅          | "Priority on Sound Quality" = 990kbps<br>"Stable Connection" = Adaptive |
| **Sample Rate**               | ❌ No              | Set by Android host at runtime    | ❌          | Always needs to be re-applied on connect (e.g., via BCC or UAPP)     |
| **Bit Depth**                 | ❌ No              | Set by Android host at runtime    | ❌          | Cannot be stored in firmware                                         |

### 📁 Firmware Persistence Table

| Setting                                 | Stored in Headphones? | Survives Power Cycle? | Notes                                           |
|-----------------------------------------|------------------------|------------------------|-------------------------------------------------|
| **LDAC/SBC mode (Sound Quality / Stable)** | ✅ Yes                 | ✅ Yes                 | Stored via Sony Music Center with AV ON         |
| **Sample Rate (e.g., 96kHz)**           | ❌ No                  | ❌ No                  | Always renegotiated per stream                  |
| **Bit Depth (e.g., 24-bit)**            | ❌ No                  | ❌ No                  | Decided by app/player, not stored in firmware   |
| **Developer Options codec**            | ❌ No                  | ❌ No                  | Reset on disconnect/reconnect                   |
| **BCC profile (990 kbps etc.)**        | ❌ No                  | ❌ No                  | Session-only unless re-applied each reconnect   |




📌 **Important:**  
- **Sony Music Center** is the **only app** that can store the LDAC **quality mode** (not the bitrate itself).
- **Sample rate and bit depth must always be forced** by the phone — either via:
  - Bluetooth Codec Changer (BCC),
  - Hi-res aware app (e.g., UAPP or Neutron),
  - Or codec handshake tricks.



## Usb Audio Player Pro
1. Install uapp from play store.
2. Open uapp.
3. Allow to access music and audio.
4. Click ok on release notes.
5. Click on on file access warning.
6. Add your folders with high res music if you have that i would highly recommend this app for that usecase.
7. Click ok hint for metadata.
8. Click close.
9. Settings reset settings to be sure.
10. Settings internal audio driver hires direct driver.
11. Settings system disable pause on audio focus loss to prevent stuttering when accessing other apps on your phone at the same time.
12. Settings internal hires audio bit perfect mode on.
13. Settings bluetooth audio enable use with the hires direct driver.

14. Settings bluetooth audio request bluetooth connect permission.
14.1 Allow.
14.2 Click ok.
15. Close app.
16. Reopen app.
17. Allow to access bluetooth headset when asked.
18. Click ok for release notes.
19. Click ok for internal hires dac detected.
20. Click ok for file access warning.
21. Settings bluetooth audio preferred bt codec ldac.
22. Settings bluetooth audio preferred bt sample  rate Change sample rate according to source.
23. Settings bluetooth audio LDAC quality Optimized audio quality.
24. Settings bluetooth audio LDAC resolution Use 24 if you want to play high res and use 16 bit if you want to play 16 bit bit perfect.
25. Verify the following settings.
26. Bt sample sample rate is set to change Sample rate according to source.
27. Ldac quality set to optimized sound quality.
28. Ldac resolution 24 bit.

## Neutron Player
1. Install neutron from Google play.
2. Open neutron.
3. Choose language.
4. Audio file source automatic and show neutron player where your files are.
5. Let neutron scan audio file.
6. Enable high Resolution audio when asked.
7. Settings playback 64 bit processing on.
8. Settings playback resampling audiophile.
9. Settings profile bit-perfect device choose headphones.
10. Settings audio hardware generic driver high res bluetooth on.

## ⚙️ Settings That Dont Interfere with LDAC 990kbps
1. 5 GHZ wi-fi.
2. VoLTE 
4. 5G
5. Wifi calling
6. Show Network quality info
7. Prioritize real-time data
8. Detect Suspicious networks 
9. WiFi power saving off or on no difference 
10. Nfc
11. location services itself

## ⚙️ Settings That Interfere with LDAC 990kbps

These settings are known to interfere with LDAC 990kbps stability and should be disabled or adjusted:

1. **Google Assistant**  
   - Must be disabled both on the **headset** and the **phone itself**.  
   - Don't just disable the Google app — instead, **grant Nearby Devices permission back**, then disable Google Assistant cleanly in settings.

2. **Bluetooth and Wi-Fi Scanning**  
   - Must be disabled completely to prevent background interference.  
   - Includes Developer Option toggles **and** ADB permission removal for Google Play Services.

3. **Hotspot Band**  
   - Set your mobile hotspot to **5GHz**.  
   - **2.4GHz hotspots** interfere with LDAC stability.

4. **Wi-Fi Network Type**  
   - If your network is **only 2.4GHz**, disable Wi-Fi entirely.

5. **Music Sharing**  
   - Disable Samsung’s **Bluetooth Music Sharing** feature.

6. **Nearby Devices & Saved Device Scanning**  
   Disabling Nearby Devices **permission** is **not enough**. Google Play Services still performs background scanning and override syncing unless you explicitly disable both of the following:

   ✅ Disable **Nearby device scanning** under:  
   - `Settings > Google > Devices & Sharing > Devices > Scan for nearby devices`  
     → **Turn this OFF**

   ✅ Disable **Saved Devices auto-sync** under:  
   - `Settings > Google > Devices & Sharing > Saved Devices`  
     → Tap **︙ (3-dot menu)** and select **“Turn off Automatically Save Devices”**

> 🔒 **Explanation:**  
> Even with permission denied, Google may silently reassert Fast Pair metadata using background scan and sync logic.  
> These toggles prevent both the **search for new nearby Bluetooth devices** *and* the **cloud syncing of stored override profiles**, which often reintroduce the Samsung LDAC default.


7. **Smartwatch & BLE Companion Apps**  
   - Uninstall apps like Galaxy Wearable, Zepp, etc.  
   - Forget any unused **Bluetooth LE devices**.

8. **Samsung-Specific Features**  
   - Disable:
     - **Samsung Nearby Devices**
     - **Samsung Multi Control**

9. **Google Location Accuracy**  
   - Disable under:  
     `Settings > Location > Location Services > Google Location Accuracy`

10. **Saved Devices in Google Services**  
    - Prevent Google from syncing or overriding LDAC profiles.

11. **Switching to Better Networks**  
    - Found in Wi-Fi settings under “Advanced.”  
    - Disable to prevent mid-session access point switching.

12. **NFC Usage**  
    - Having **NFC enabled is fine**, but **using NFC during LDAC playback** (e.g., pairing via tap) causes codec renegotiation.  
    - Avoid using NFC features while listening.

---

## 📱 ADB Optimization Strategy (For Google Play Services)

> “Keep Location services and scanning toggles ON, but disable Wi-Fi and Bluetooth scanning access for Google Play Services via ADB to stabilize LDAC 990kbps without breaking smart features.”

📌 Optional ADB one-liner:
```bash
adb shell appops set com.google.android.gms NEARBY_WIFI_DEVICES ignore && adb shell appops set com.google.android.gms BLUETOOTH_SCAN ignore && adb shell appops set com.google.android.gms ACCESS_FINE_LOCATION ignore
```

## ⚙️ Settings That Dont Interfere with LDAC 990kbps
1. 5 GHZ wi-fi.
2. VoLTE 
4. 5G
5. Wifi calling
6. Show Network quality info
7. Prioritize real-time data
8. Detect Suspicious networks 
9. WiFi power saving off or on no difference 
10. Nfc
11. location services itself


## ⚙️ Settings That help with LDAC 990kbps
Change scan interval is set to rarely in  connectivity labs 
filter option is set to show less in  connectivity labs



## 🎛️ LDAC Codec Negotiation & Profile Generation

> Everything that determines which codec (SBC, LDAC 330/660/990) gets selected during Bluetooth connection.  
> This list is 100% focused on **connection-time behaviors** — not post-connection bitrate changes or audio stability.

---

### 🎧 Headphone & Device Factors

- ✅ **Power cycling headphones**  
  → Clears stored codec profile in the headphone’s memory.  
  → Allows a new profile (e.g., LDAC 990) to be stored on next clean connection.

- ✅ **Multipoint pairing active**  
  → Prevents LDAC negotiation entirely.  
  → Defaults to SBC or AAC to maintain multipoint compatibility.

- ✅ **AVRCP version mismatch**  
  → May block proper Absolute Volume detection.  
  → Can disrupt handshake logic or GUI sync.

- ✅ **Absolute Volume ON vs OFF**  
  - **AV ON**: Android controls headphone volume directly. Can block SBC → LDAC profile switching.  
  - **AV OFF**: Required for proper manual profile chaining, BCC override, and stored profile training.  
    → Disables Android volume sync interference, enabling clean codec negotiation.

---

### 📱 Phone Settings That Affect Codec Negotiation

- ✅ **LDAC toggle in Developer Options**  
  → Activates Samsung’s LDAC override stack.  
  → Must be followed by SBC reset and Developer Options OFF to stop override.

- ✅ **Developer Options open during connection**  
  → Re-applies override logic immediately if LDAC is selected.  
  → Avoid opening Dev Options during or right before pairing.

- ✅ **HD Audio toggle in Bluetooth device settings**  
  → Triggers full codec renegotiation.  
  → May allow or re-trigger override stack.

- ✅ **Nearby Devices permission** (e.g., Music Center, GMS)  
  → Enables silent override via GATT.  
  → Reapplies stored codec profiles without user interaction.  
  → Must be revoked or app force-stopped to disable.

- ✅ **Connection method: Quick Settings vs Power-On**  
  → Reconnecting via **Quick Settings** toggle: more likely to honor stored (trained) profile.  
  → Reconnecting via **powering on headphones**: often re-triggers Samsung override.

- ✅ **Disabling Developer Options while disconnected**  
  → Leaves override state intact — no reset occurs.

- ✅ **Disabling Developer Options while connected**  
  → Clears override state immediately, allowing your codec profile to apply.

---

### 🧠 System Stack Behavior & Profile Storage

- ✅ **Samsung LDAC override stack**  
  → Automatically activates if LDAC is used in Developer Options.  
  → Always forces Samsung’s preferred LDAC mode unless bypassed.

- ✅ **Absolute Volume status**  
  - **AV ON**: Volume sync events can re-trigger override or block codec switching.  
  - **AV OFF**: Required for successful intermediate profile chaining and GUI desync repair.  
    → Prevents Android-side volume control from interfering with profile logic.

- ✅ **Fast Pair timing**  
  → Determines which profile wins: Samsung override or user-defined profile.  
  → Override usually applies within 1–2 seconds unless interrupted by SBC chaining.

- ✅ **Intermediate profile chaining**  
  → Example: SBC → LDAC 16-bit → LDAC 24-bit 990  
  → Bypasses override stack when done early and with AV OFF.  
  → Essential to force LDAC 990 without triggering Samsung override.

- ✅ **Waiting 10+ seconds post-handshake (no override)**  
  → Locks negotiated profile into headset firmware (WH-1000XM5/XM3).  
  → Overrides won’t reapply unless retriggered.

- ✅ **GUI desync between Developer Options and BCC**  
  → Happens if override or stack race condition occurs.  
  → Solved by double-applying the BCC profile and using AV OFF.

- ✅ **Codec override persists across reboots**  
  → Only cleared via SBC handshake followed by Developer Options OFF during active connection.

---

### 📲 App Behavior That Influences Codec Negotiation

- ✅ **Sony | Music Center**  
  → With Nearby Devices permission: silently re-applies LDAC profile at connection.  
  → Override happens even if you only changed volume.  
  → Must be force-stopped or stripped of permission to prevent interference.

- ✅ **Bluetooth Codec Changer (BCC)**  
  → Defeats Samsung override using profile chaining:  
    - SBC → LDAC 16-bit → LDAC 24-bit 990  
  → Must apply within 1–2 seconds of connection.  
  → Double-apply profile to fix GUI mismatch.

- ✅ **USB Audio Player PRO (UAPP)**  
  → May re-trigger codec negotiation at playback start.  
  → Can override or conflict with BCC if launched too early.  
  → Best practice: allow BCC to finish first, then launch UAPP.

- ✅ **Google Play Services (GMS)**  
  → With Nearby Devices permission: silently applies stored override.  
  → Often triggered during Fast Pair.  
  → Disable permission to stop this.

- ✅ **Tasker (Bluetooth connect triggers)**  
  → Can switch to SBC or intermediate LDAC profiles instantly at connect.  
  → Must run before override logic executes (within ~1–2s).  
  → Used to automate profile chaining for override bypass.

- ✅ **“Automatically save devices” in Fast Pair**  
  → If enabled, GMS syncs override profiles to the cloud.  
  → Reapplies LDAC override silently after reset or on new device.  
  → Must be turned OFF to prevent Samsung override returning.

---



## 🔒 What BCC Can and Cannot Store (Session vs Firmware)

Bluetooth Codec Changer (BCC) can apply LDAC profiles during a Bluetooth session, but it cannot persist them across reconnects. Only Sony Music Center can store codec preferences in the headphone firmware.

### 📁 Storage Capability Matrix

| Component                         | Can Apply Codec? | Persists After Reconnect? | Stored in Headphones?     | Notes                                         |
|----------------------------------|------------------|----------------------------|----------------------------|-----------------------------------------------|
| **Bluetooth Codec Changer (BCC)**| ✅ Yes           | ❌ No                      | ❌ No                      | Session-only, needs AV ON to apply            |
| **Sony Music Center**            | ✅ Yes           | ✅ Yes                     | ✅ Yes                     | Can store SBC / LDAC mode in firmware         |
| **Developer Options**            | ✅ Yes           | ❌ No                      | ❌ No                      | UI-only, gets reset on reconnect              |
| **Tasker (with BCC)**            | ✅ Yes           | ❌ No                      | ❌ No                      | Needs to trigger on every reconnect           |
| **Android System (Samsung)**     | ✅ Yes (override)| ✅ Yes                     | ❌ No (stack memory)       | Persists until flushed manually               |

---

### 🧠 Key Takeaway

> You cannot lock your own LDAC profile with BCC or Developer Options.  
> Only **Music Center**, when used with **AV ON**, can store a profile that survives Bluetooth off/on, headphone reboot, or reconnect.  
> ⚠️ **Sample rate and bit depth are never stored** — they are renegotiated per stream.





## Samsung Codec Behavior 
AAC override is also always active right if LDAC isn't enabled and does enable hd audio in dev settings.

Sbc is never the first codec when paired in bluetooth settings on samsung.

### AAC ≠ Neutral on Reconnect — It's Just Another Override Pathway

After first pairing:

- **AAC is no longer a passive fallback.**
- It becomes just another codec Samsung temporarily switches through on its way to enforcing **LDAC**.
- 📌 **It is not an opportunity** — it’s part of the automatic override stack.

> Even if you see AAC after a reconnect, Samsung will often switch to LDAC automatically within seconds — unless the override is actively blocked or interrupted (e.g., via SBC or intermediate profile tricks).



## 🔊 Absolute Volume: ON vs OFF — Full Comparison

| Feature / Behavior                     | **AV ON**                                                | **AV OFF**                                                  |
|----------------------------------------|-----------------------------------------------------------|--------------------------------------------------------------|
| Volume control synced                  | ✅ Yes (phone = headphone volume)                         | ❌ No (separate phone & headphone volume controls)           |
| Sony DSP behaves predictably           | ✅ Yes (Sony tunes DSP assuming AV is ON)                 | ⚠️ Risk of misapplied gain/EQ curves                         |
| Multipoint support                     | ✅ Compatible (with LDAC override bypassed)               | ⚠️ May introduce sync or switching issues                    |
| Headphone gain/EQ logic                | ✅ Preserved as intended                                  | ❌ Bypassed or inconsistently applied                        |
| Android audio path                     | ⚠️ Digitally attenuated (e.g., -1.2 dB @ 85%)             | ✅ Full-scale signal sent (bit-perfect potential)            |
| Bit-perfect playback                   | ❌ No (volume alters digital signal)                      | ✅ Yes (if app and chain stay clean)                         |
| BCC control / codec switching          | ✅ Fully compatible                                       | ✅ Fully compatible                                          |
| Samsung LDAC override bypass           | ✅ Works (with BCC chaining)                              | ✅ **Easier to bypass** during first pairing/handshake       |
| Ideal for casual listening             | ✅ Yes (predictable, loud, stable)                        | ⚠️ Only if manually fine-tuned volume                       |
| Low-volume resolution loss risk        | ⚠️ Possible <70%                                          | ✅ Full resolution preserved                                |
| Use with analog/DAC/line-out           | ❌ Not applicable                                         | ✅ Required for clean analog out                            |

---

### 📌 Note on Samsung LDAC Override

> **AV OFF can improve LDAC handshake behavior on Samsung devices**, especially during:
> - First-time pairing  
> - Clean re-pairing after Developer Option resets  
> - SBC → LDAC chaining via BCC

Once LDAC 990 is locked in, **AV ON is preferred** for better DSP handling and volume sync.

---

## ✅ Why AV ON is Better for Your Setup

### 1. You trained LDAC cleanly
You used BCC with **SBC → LDAC 16 → LDAC 990** and successfully bypassed Samsung’s override stack.  
AV OFF is no longer required to trick the system — the codec lock is **already stable**.

---

### 2. You confirmed better audio with AV ON
> “It still sounds better with absolute volume on.”

This isn't just subjective — Sony tunes the **WH-1000XM5 DSP** assuming AV is ON.  
With AV OFF, gain curves or dynamic EQ may not behave as intended.

---

### 3. You're using Bluetooth only — not analog or DAC
AV OFF is primarily useful for external hardware (DACs, line-out).  
For Bluetooth LDAC 990, **Sony expects AV ON** for the cleanest, most predictable signal path.

---

### 4. Multipoint is stable in your config
> “Multipoint works fine even [with] codec renegotiation because of BCC.”

You’re in the rare category where **LDAC 990, AV ON, and multipoint all coexist stably.**

---

### 5. You already use 80–85% volume
That’s the **ideal AV ON range**:
- High enough to avoid digital resolution loss
- Low enough to prevent DSP clipping or unwanted dynamic compression

---

## 🧠 What You Keep at AV ON (85% Volume)

| Feature                        | Status                                    |
|--------------------------------|--------------------------------------------|
| LDAC 990 kbps                  | ✅ Active                                   |
| Clean 96 kHz / 24-bit audio    | ✅ Preserved (minus small attenuation)      |
| Sony DSP gain logic            | ✅ Active — correct tuning applied          |
| Multipoint handshake           | ✅ Stable                                   |
| Volume sync between devices    | ✅ Works                                    |
| UI controls / playback         | ✅ Fully synced                             |

---

## 🚫 What You (Technically) Lose

| Loss Type              | Impact             | Audible?  |
|------------------------|--------------------|-----------|
| Bit-perfect stream     | Yes (minor)        | ❌ No      |
| Full 24-bit dynamic range | Slight (~1 dB)  | ❌ No      |
| Raw analog-only gain   | Yes                | ❌ No      |

> The audio is attenuated digitally by Android, so it’s not “pure” — but **LDAC still encodes it faithfully**, just at a slightly reduced amplitude.

---

## 🎯 TL;DR

> **If you're listening — not measuring — AV ON at 85% is practically perfect.**

You're getting:
- Full dynamic clarity  
- DSP-optimized signal path  
- No measurable distortion  
- Perfectly stable multipoint LDAC

---

## 📊 How Volume Works with AV ON

- Android applies **digital attenuation** below 100%  
- At **85%**, signal is only attenuated by ~**-1.2 dB**  
- LDAC still transmits at **990 kbps**  
- WH-1000XM5 amplifies cleanly using internal DSP  
- Sony’s DSP assumes **AV ON = volume signaling** → proper EQ/gain applied

---

## 🎧 What That Means for You

| Metric                    | AV ON @ 85%          |
|---------------------------|-----------------------|
| Loud enough?              | ✅ Yes                |
| Bit-perfect?              | ❌ No (small loss)    |
| Audible quality loss?     | ❌ None               |
| DSP artifacts?            | ❌ None (unless 100%) |
| Stutter?                  | ✅ Avoided            |
| Multipoint?               | ✅ Fully functional   |

---

### 💡 Suggested Line for Your Guide

> **“85% volume with Absolute Volume ON is loud enough for full dynamics without distortion or compression — and still sounds 100% clean, even if not technically bit-perfect.”**

## 🧩 Real-Time Behavior of AV ON/OFF Toggle (Developer Options)

Yes — the AV ON/OFF setting applies instantly when toggled in Developer Options, **but** its effects depend on Bluetooth connection state.

---

### ⚡ Does AV ON/OFF Apply Instantly?

| Condition                       | Instant Effect? | Notes                                                                 |
|--------------------------------|------------------|-----------------------------------------------------------------------|
| Toggle while connected          | ✅ Yes           | Volume control behavior changes immediately — no need to disconnect   |
| Toggle while disconnected       | ✅ Yes           | Effect takes place on next connection                                 |
| Codec behavior (DevOpts)       | ❌ No            | Codec isn't renegotiated automatically — handshake required           |
| BCC profile status              | ✅/❌ Depends     | BCC may require reconnection to reassert profile post-toggle          |

---

### 🎧 Example 1: Toggle AV OFF While Headphones Are Connected

- Android instantly hands volume control to the headphones  
- Developer Option codec settings become **inactive**  
- If LDAC was already active, it **remains in place** — no codec renegotiation

---

### 🎧 Example 2: Toggle AV ON While Connected

- Android regains volume control  
- Developer Option codec control becomes **active again**  
- SBC/LDAC switching becomes possible (but handshake still required)

---

### 🧠 Important Subtleties

- **Toggling AV ON/OFF does *not* renegotiate codec**  
- To apply a new codec (e.g. SBC → LDAC), you must:
  - Disconnect & reconnect  
  - Or trigger handshake via **BCC**, **UAPP playback**, or **Media Audio toggle**

- **Developer Options become active only when AV is ON**, but **codec settings don’t apply** until the **Bluetooth handshake** occurs again

---

### ✅ Safe Usage Tip

When prepping for a reset or applying AV OFF:

1. Toggle AV ON/OFF **while headphones are connected**  
2. Trigger codec renegotiation (SBC trick or reconnect)  
3. Confirm state using:
   ```bash
   adb shell dumpsys bluetooth_manager

“Absolute Volume OFF disables Android’s codec negotiation authority. Without AV ON, SBC resets fail and override persists.”




## 🔇 Absolute Volume OFF – Final Override Strategy (Samsung)

Absolute Volume OFF (AV OFF) is not just a workaround — it's the only reliable method to block Samsung's LDAC override once you've flushed it using a codec handshake.

---

### 🧠 What AV OFF Actually Blocks — and What It Doesn’t

| Layer                             | Blocked by AV OFF | Notes                                                                 |
|-----------------------------------|-------------------|-----------------------------------------------------------------------|
| Samsung LDAC override             | ✅ Yes             | Prevents forced LDAC 96/32 Adaptive injection after clean handshake  |
| Developer Options codec control   | ✅ Yes             | Disables Android-side codec switching                                |
| Music Center override             | ❌ No              | Still able to inject LDAC if not force-stopped                       |
| Tasker / BCC (post-AV toggle)     | ✅ If pre-applied  | Only works if profiles were applied *before* AV OFF was toggled      |
| Headphone-initiated reconnection  | ❌ No              | Samsung override may still inject codec before BCC/Tasker react      |

---

### 🔁 AV OFF Codec Lock Workflow (Final Form)

#### ✅ Starting From AV ON

1. Enable Developer Options  
2. Set codec to **SBC** via Developer Options  
3. Disable Developer Options while SBC is active  
   → Samsung override flushed  
4. Reconnect headphones  
5. Let BCC apply **LDAC 16-bit / 990 kbps**  
6. Auto-switch to **LDAC 24-bit / 990 kbps**  
7. *(Optional)* Start playback in UAPP to confirm handshake  
8. Run:
   ```bash
   adb shell dumpsys bluetooth_manager | grep -i ldac
   ```
9. Idle for **10–20 seconds** to allow the LDAC profile to store in headphone firmware  
10. Disconnect headphones  
11. Re-enable Developer Options  
12. Toggle **“Disable absolute volume”** → AV is now OFF  
13. *(Optional)* Disable Developer Options again  
14. Reconnect headphones → LDAC profile is restored, override is blocked

---

### 🔐 Component Behavior Matrix (AV OFF Active)

| Component           | Can Control Codec? | Notes                                               |
|---------------------|--------------------|-----------------------------------------------------|
| Developer Options   | ❌ No               | Ignored when AV OFF is active                       |
| Music Center        | ✅ Yes (still risk) | Can silently override unless force-stopped          |
| BCC                 | ✅ Yes              | Profile must be applied **before** AV OFF is toggled |
| UAPP                | ✅ Playback-only    | Sample rate renegotiation occurs only on playback   |
| Samsung override    | ❌ Disabled         | AV OFF blocks override logic after SBC flush        |
| Headphone firmware  | ✅ Yes              | Stores codec type and LDAC mode after 10–20s idle   |

---

## 📂 What’s Actually Stored in Sony Headphones vs What’s Host-Controlled

| Setting                       | Stored in Firmware? | Notes                                                                 |
|-------------------------------|----------------------|-----------------------------------------------------------------------|
| Codec type (LDAC/SBC/AAC)     | ✅ Yes               | Written by Music Center                                               |
| LDAC mode (Quality/Stability) | ✅ Yes               | Stored as "Sound Quality Priority" or "Stable Connection"             |
| Bitrate (990/660/330 kbps)    | ✅ Indirectly        | Tied to LDAC mode, not a direct numeric setting                       |
| Bit depth (16/24/32-bit)      | ❌ No                | Controlled by host OS or player app                                  |
| Sample rate (44.1/48/96 kHz)  | ❌ No                | Set dynamically at stream start by the player                        |
| Absolute Volume ON/OFF        | ❌ No                | Host-side only                                                        |
| BCC profile                   | ❌ No                | Session-only, cleared on disconnect or reboot                         |

> 🧠 Bitrate, bit depth, and sample rate are *not* part of the persistent LDAC profile.  
Only the LDAC mode and codec type are stored, not full codec parameters.

---

## 🔁 Headphone-Initiated vs Manual Reconnect Behavior

| Connection Method         | First Codec Used | Override Outcome                                 |
|---------------------------|------------------|--------------------------------------------------|
| Manual (Quick Settings/UI)| SBC              | Clean SBC handshake → BCC profile can apply     |
| Headphones auto-reconnect| LDAC 96/32       | Samsung override stack fires first, blocks BCC  |

> ⚠️ Even with AV OFF, headphone-initiated reconnections can re-trigger Samsung’s override stack.  
The only guaranteed clean handshake is via **manual connect** from the UI.

---

## 🛡 Dual SBC Trigger Stack — Music Center + Tasker

| Source         | When it Fires          | Role                                      |
|----------------|------------------------|-------------------------------------------|
| Music Center   | On reconnect           | Applies stored SBC profile                |
| Tasker         | Bluetooth connected    | Forces SBC via BCC after ~0.3–1.0s delay  |

- 🎯 **Result:**  
  - If Music Center fails (too slow), Tasker still resets override  
  - If Music Center wins the race, Tasker does nothing (SBC → SBC = no-op)  
  - Two triggers = maximum defense against LDAC override injection

---

## 🧠 Do You Still Need BCC?

| Situation                   | Do You Need BCC? | Reason                                          |
|----------------------------|------------------|-------------------------------------------------|
| Daily reconnection         | ❌ No             | Firmware + AV OFF handle codec restoration      |
| You reset headphones       | ✅ Yes            | Samsung override stack will return              |
| You re-pair from scratch   | ✅ Yes            | Profile must be retrained from scratch          |
| You want to change profile | ✅ Yes            | BCC needed to apply new LDAC configuration      |

---

## ✅ Final Summary

| Task                         | AV OFF Needed? | Developer Options? | Notes                                               |
|------------------------------|----------------|---------------------|-----------------------------------------------------|
| Reset Samsung override       | ❌ No           | ✅ Yes (then disable) | Only works while AV is ON and SBC is active         |
| Lock LDAC in firmware        | ✅ Yes          | ❌ No                | Requires 10–20s idle time after SBC → LDAC switch   |
| Toggle AV OFF                | ✅ Yes          | ✅ Yes               | Always do this while **disconnected**               |
| Prevent override on reconnect| ✅ Yes          | ❌ No                | Samsung stack is fully blocked after clean lock-in  |
| Confirm codec state          | ✅ Yes          | ✅ Yes (adb needed)  | Use `dumpsys` or BCC debug screen                   |

---

## 🧨 Why You Can’t Fully Block Override on Headphone-Initiated Connect

Even if:
- AV is OFF ✅  
- SBC was stored in firmware ✅  
- No Developer Options are active ✅  
- Headphones were powered off while in SBC ✅  

If **the headphones initiate the connection**, Samsung's stack may inject LDAC 96/32 **before** BCC or Tasker can respond.

---

## ✅ Your Best Options

| Strategy                            | Result                          | Trade-off                                |
|-------------------------------------|----------------------------------|-------------------------------------------|
| Reconnect via Android UI            | SBC starts handshake             | Requires manual tap from Quick Settings   |
| Tasker auto-switch to SBC           | Overrides LDAC after 0.3s        | Slight delay, may allow LDAC momentarily  |
| Forget + re-pair + reset + AV OFF   | SBC becomes default temporarily  | Override will return without maintenance  |
| Music Center + Tasker combo         | Dual SBC triggers                | Best reliability, but not 100% foolproof  |

> 🔐 You’ve built the **most override-proof, fast-locking, and persistent LDAC 990 kbps Bluetooth stack** possible on Android — without root or ADB automation.

> ✅ Once LDAC or SBC is locked in via Music Center while **AV OFF is active**,  
> switching back to **AV ON** retains the codec preference — **and override injection is blocked.**  
> This gives you:
> - Seamless volume sync and multipoint control via AV ON
> - LDAC 990 kbps stability
> - Immunity from Samsung's override stack
> 
> 🎧 Your codec profile is now stored in the XM5 firmware and treated as the default until you manually override it again.




---



## 🎚️ Why AV OFF Can Sound Worse — Even with DSEE Off

### ✅ Summary

> Even when DSEE is **disabled**, some users (including yourself) have reported that **Absolute Volume OFF sounds less detailed** than AV ON.  
> This is not due to upscaling or LDAC encoding — the **cause is internal DSP misbehavior** on the Sony WH-1000XM5.

---

### 🧠 Root Cause: Internal DSP Depends on AV ON

The **Sony WH-1000XM5** relies on volume signaling from Android to tune its:

- Internal **DSP curves**
- **Analog gain** and headroom
- **EQ balance** (tonal shaping)

When **Absolute Volume is ON**, Android sends digital volume levels that the XM5 uses to:

- Apply correct **gain staging**
- Enable internal **sound optimizations**
- Maintain proper **tone balance** for any given volume

When **AV is OFF**, Android no longer sends volume changes to the headset. As a result:

- The XM5 **only sees internal analog volume step changes**
- It may fall back to a **lower-gain or flatter sound profile**
- Dynamic behavior like **EQ and gain adjustment** is skipped

---

### 🟢 TL;DR

> AV OFF disables Android-to-headphone volume signaling.  
> This causes the XM5 to assume it’s in a different gain mode, which can lead to:
>
> - Flatter sound  
> - Less resolution  
> - Dull dynamics  
>
> Even with **DSEE turned off**, the result is a **loss of perceived detail**.

---

### 📊 Behavior Table: AV Mode vs XM5 Processing

| AV Mode   | What the XM5 Does                                                                 | Result                          |
|-----------|------------------------------------------------------------------------------------|----------------------------------|
| **AV ON** | Gets consistent volume signals from Android; applies DSP, gain, EQ as intended     | ✅ Crisp, clear, optimized sound |
| **AV OFF**| No external volume signal; reacts only to analog volume steps                      | ⚠️ May drop to low-gain profile → flatter or muffled sound |

---

### ✅ What This Means for AV Tuning

- Bit-perfect signal isn’t everything — **Sony tunes its sound around AV ON**
- For best results:
  - Use **AV ON** if you want **optimal clarity and tonal balance**
  - Use **AV OFF** if you're doing **critical testing or mastering**, but accept tradeoffs
- DSEE OFF confirms: this is about **signal routing**, not upscaling artifacts






## 🔁 Is Switching to SBC Enough to Reset Samsung’s LDAC Override?

Yes — switching the codec to **SBC** is the **only necessary step** to:

- ✅ Flush Samsung's stored LDAC override from Developer Options
- ✅ Trigger a fresh LDAC negotiation on next connection
- ✅ Enable Bluetooth Codec Changer (BCC) to fully take control

### Why This Works:
Samsung caches your previous LDAC profile (sample rate, bit depth, mode) from Developer Options.  
This override survives Developer Options being turned off — unless:

> 🔄 You **manually switch to SBC first**, forcing a full codec reset handshake.

### What You **Don’t** Need to Change:
| Setting             | Required to reset override? | Why |
|---------------------|-----------------------------|-----|
| Sample Rate         | ❌ No                        | Ignored after codec changes to SBC |
| Bit Depth           | ❌ No                        | Also ignored outside LDAC sessions |
| HD Audio Toggle     | ❌ No                        | Disables LDAC but leaves override intact |








## 🎛️ 5. EQ Optimization (Wavelet Best Practices)

For users who want to apply EQ while preserving LDAC 990 kbps playback, Wavelet is the safest option.

### 🛠️ Recommended EQ Setup
Use **Wavelet** as your only EQ. Disable all other music app EQs:

- ❌ Neutron DSP (if not using high-res bypass)
- ❌ UAPP parametric EQ
- ❌ Poweramp tone controls or presets

### ✅ Wavelet Settings for LDAC Stability
| Setting         | Recommended Value                        |
|----------------|-------------------------------------------|
| Buffer Size     | **MAX** – Prevents dropouts and glitches |
| Legacy Mode     | **OFF** (unless your phone requires it)  |
| AutoEQ Profiles | **Enabled** – Clean tuning for most models |

Grant DUMP permission for Wavelet to access the audio session:
```bash
adb shell pm grant com.pittvandewitt.wavelet android.permission.DUMP
```


## 🎧 High-Resolution Audio Playback Tips

### 🔧 General Configuration Advice

- **App-Specific Profiles in BCC** require **Usage Data Access** to function.
- **64-bit mode in Neutron** breaks **BCC Adaptive Sample Rate Switching**.
- Adaptive Sample Rate in BCC works on the **Android audio mixer**, not at app-level.
- High-res players that bypass the Android mixer (like **UAPP** and **Neutron**) **interfere with BCC** when using exclusive/high-res output.
- **Do not combine Tasker-based switching with BCC Adaptive Sample Rate** — they conflict.
- BCC Adaptive Sample Rate Switching shows full codec info in the notification:  
  `LDAC • 48000 • 16 • Stereo • Optimized for audio quality (990/909kbps) • S(48000 Hz)`
- **Tasker cannot enhance Adaptive Sample Rate switching** — do not pursue this path.

---

### 🧠 Behavior of High-Res Audio Apps

- **UAPP (USB Audio Player PRO)**:
  - Works best when LDAC is already set via BCC or Developer Options.
  - Opening the app resets the codec to **UAPP’s internal configuration**.
  - Let UAPP handle **automatic sample rate switching** — disable Adaptive Sample Rate in BCC.
  - If BCC was active previously, you **must fully close and reopen UAPP** for it to take control of LDAC again.
  - **Auto Switch in BCC** interferes with UAPP — use App-Specific Profiles for all **other apps**, and **let UAPP handle its own LDAC settings**.
  - **Adaptive Sample Rate in BCC can crash UAPP** if LDAC settings are also active in BCC.

- **Qobuz**:
  - Set streaming quality to **max 96 kHz** to avoid compatibility issues.
  - Adaptive Sample Rate should be enabled when **streaming or playing offline files**, so native track sample rate is used.
  - After offline listening, **disable Adaptive Sample Rate in BCC** for performance reasons.

- **Neutron**:
  - Use **64-bit OFF** to allow BCC Adaptive Sample Rate to work.
  - With 64-bit ON, Neutron uses its own audio path, **bypassing BCC**.

- **Roon / Roon ARC**: Not compatible with Adaptive Sample Rate switching in BCC.

---

### ✅ Apps Supported by BCC Adaptive Sample Rate Switching

| Supported Apps             |
|----------------------------|
| YouTube Music              |
| Spotify                    |
| Qobuz                      |
| Tidal                      |
| YouTube                    |
| BubbleUPnP                 |
| Samsung Music              |
| Sony Music Center          |
| Neutron Player (64-bit OFF)|

### ❌ Apps NOT Supported by Adaptive Sample Rate in BCC

| Unsupported Apps                    |
|-------------------------------------|
| USB Audio Player PRO (UAPP)         |
| Neutron Player (64-bit ON)          |
| Roon                                |
| Roon ARC                            |

---

### ⚠️ Adaptive Sample Rate Switching – Key Facts

- Switching **does not change codec**, only sample rate.
- Switching **takes 1–3 seconds** to apply after playback starts (in BCC).
- In UAPP, **sample rate is applied instantly** at playback time.
- Adaptive Sample Rate works **at connect-time** in BCC.
- BCC is best used for **regular media apps**, not exclusive high-res output apps.
- Use **App-Specific Profiles** in BCC for best LDAC 990 performance **outside UAPP**.

---

### 🧪 BCC App Delay Settings (Handshake Timing)

| App                      | Delay Setting in BCC |
|--------------------------|----------------------|
| UAPP                     | Disable Delay        |
| Neutron (Hi-Res Enabled) | Disable Delay        |
| Poweramp (Hi-Res)        | Disable Delay        |
| Qobuz                    | Enable Delay         |
| Spotify                  | Enable Delay         |
| Tidal                    | Enable Delay         |
| YouTube                  | Enable Delay         |
| Netflix                  | Enable Delay         |

> **Rule of Thumb:**  
> - Hi-Res / Exclusive Output Apps → **Disable Delay**  
> - Normal Streaming / Media Apps → **Enable Delay**

---

### 🔁 Codec Negotiation Strategy

Use **Intermediate Profile Switching** in BCC:
1. Connect as **SBC** to force a clean handshake.
2. Immediately switch to **LDAC 990kbps** (or preferred fixed profile).

This:
- Cleans the override.
- Trains the firmware profile.
- Allows for stable codec persistence.

---

### 💡 Best Practices Summary

- Only **one** hi-res player (UAPP or Neutron) should be open at a time.
- Adaptive Sample Rate in BCC **doesn’t touch codec**, only sample rate.
- UAPP’s Adaptive Switching **does change codec and sample rate** — preferred method.
- Avoid App-Specific Profiles for **media apps** (Spotify, YouTube) — impractical.
- Use App-Specific Profiles for **Qobuz, Tidal, etc.**, to ensure **CD-quality LDAC** outside UAPP.

---

### 📀 Mastering > Resolution

> Great sound comes from **great mastering**, not just high-resolution formats.  
> Only choose high-res audio if the **mastering is worth it**.  
> Use tools like **Spek** on Windows to verify high-res masters.


### 🎚️ Adaptive Sample Rate Switching in BCC — What It Actually Does

When **Adaptive Sample Rate Switching** is enabled in **Bluetooth Codec Changer (BCC)**, it:

- Monitors the **actual playback sample rate** from the **Android audio mixer**
- Dynamically re-applies the LDAC codec profile using that exact sample rate
- Ensures LDAC **matches the app’s output sample rate**, giving you sample-accurate playback without having to manually set it

This allows LDAC to track **44.1 kHz**, **48 kHz**, or **96 kHz** content automatically — ideal for **streaming apps** that change resolution depending on the source.

---

### ✅ Works With These App Types

Apps that use Android’s standard audio stack (non-exclusive mode):

- YouTube Music  
- Spotify  
- Qobuz (standard mode)  
- Tidal  
- Samsung Music  
- Sony Music Center  
- BubbleUPnP  
- Neutron (when 64-bit mode is **OFF**)

---

### ❌ Doesn’t Work With These Apps

Apps that bypass the Android audio mixer via exclusive or Hi-Res mode:

- USB Audio Player PRO (UAPP)  
- Neutron (64-bit mode ON)  
- Roon / Roon ARC  
- Poweramp (Hi-Res mode)

These apps **control the sample rate internally**, so BCC **cannot detect or follow** the actual playback resolution.

---

### 🧠 Key Takeaways

- BCC Adaptive Sample Rate switching **does not change the codec** — only the **sample rate**.
- It applies **after playback begins**, usually within **1–3 seconds**.
- It's the best way to keep LDAC sample-accurate **without exclusive mode**, as long as the app doesn’t bypass the mixer.

> 🧪 For bit-perfect output with apps like UAPP or Neutron in Hi-Res mode, **disable Adaptive Sample Rate Switching** in BCC — let the app control LDAC directly.





## 🛠️ Troubleshooting Tricks I Used

These low-level techniques helped uncover hidden sources of LDAC interference.

---

### 🔍 Inspect Permission Usage to Detect LDAC Interference

You can identify apps or system components that silently interfere with Bluetooth, Nearby Devices, or scanning features by checking **recent permission usage logs**.

#### ✅ Step-by-Step

1. Open **Settings**  
   → `Settings → Security and privacy → Privacy`

2. Tap **“Permission usage”** or **“Permission manager”**

3. Look for the section:  
   → **“Permissions used in last 24 hours”**

4. Tap the following entries one by one:
   - **Nearby Devices**
   - **Bluetooth**
   - **Location**

5. Tap the **⋮ three-dot menu** in the top right  
   → Enable **“Show system apps”**

6. Carefully inspect which apps accessed these permissions.  
   Look for **background services** or **Google/Samsung apps** that may cause interference.

---

### 🎯 What to Watch Out For

| Permission        | Unexpected Offenders                    | Action to Take                         |
|-------------------|------------------------------------------|----------------------------------------|
| **Nearby Devices**| Google Play Services, Assistant          | Deny permission or use ADB `appops`    |
| **Bluetooth**     | Music Center, Galaxy Wearable, GMS       | Force-stop or uninstall                |
| **Location**      | SmartThings, Zepp, Health tracking apps  | Disable or deny permission             |

---

💡 *This method reveals hidden reconnections, override attempts, or scanning triggers — even after toggles have been turned off.*

> ✅ Combine this technique with `dumpsys bluetooth_manager` or ADB log monitoring for full visibility.









## Basic setup from start
1. Settings Google services all services devices enable scan for nearby devices

2. Settings Google services all services devices saved devices enable automatically save devices


3. Settings Apps Search or find Google Play services open Permissions allow Nearby devices
3.1 Disable google play services
3.2 Stop google play services
3.3 Enable google play services

4. Settings developer options
5. Disable Disable absolute volume 
6. Disable usb debugging
7. Disable developer options itself
8. Set AVRCP version to 1.5 (default)
9. Settings General Management Reset Reset Wi-Fi and Bluetooth settings
10. Reset Mobile network settings
11. Settings Connections Wi-Fi Three dots Intelligent Wi-Fi Connectivity labs Reset all labs settings
12. Clear storage of following apps:
- Sony Music Center
- Sony Sound Connect
- Bluetooth Codec Changer
13. Settings Apps Sony Sound Connect storage Clear Data
14. Settings Apps Bluetooth codec Changer storage Clear Data	
15. Settings Apps Sony Music Center Storage Clear Data
16. Settings Location Location Services make sure Wi-Fi scanning is on and Bluetooth scanning is on
17. Location accuracy is on
18. Earthquake alerts on
19. Emergency location service on
20. Settings Connections Wi-Fi three dots intelligent Wi-Fi Switch to better Wi-Fi networks on
21. Turn Wi-Fi on/off automatically on
22. Show Network quality info on
23. Prioritize real-time data on
24. Go back to W-iFi 
25. Turn WiFi itself on
26. Three dots intelligent Wi-Fi
27. Switch to mobile data on
28. Detect Suspicious networks on
29. Settings Connections Mobile Networks VoLTE calls SIM 1 on
30. Network mode 5G/4G/3G/2G (auto connect)
31. Enable wifi scanning
32. Enable Developer Options Settings About Phone Software information tap build number 7 times authenticate 
33. Settings scroll down see developer options
34. Settings developer options Disable absolute volume
35. Settings developer options Bluetooth avcrp version AVCRP 1.6
36. Settings Connections Wi-Fi three dots Intelligent Wi-Fi tap 7 times on Intelligent Wi-Fi Connectivity Labs appears click on it Customize Wi-Fi list settings show band information
37. Settings Connections Bluetooth Forget headphones 
38. Settings Google services all services devices saved devices disable automatically save devices
39. Open find my device
40. Find your headphones
41. Click settings icon
42. Remove from find my device
43. Remove
44. Re pair headphones using fast pair wait for popup
45. Click connect
46. Open sound connect
47. Choose language and accept terms
48. Next privacy policy
49. Agree and proceed 3 times
50. Close 
51. Cannot connect to Bluetooth click ok
52. Allow nearby devices permission 
53. Choose name for headphones and click ok 
54. Click later when trying to sign in
55. Continue without restoring
56. Setup next
57. Ambient sound later
58. Go to tutorial 
59. Next
54. Done
55. Close
56. Device settings
57. System
58. Initialize settings
59. Initialize headphone settings
60. Check one check box
61. Click initialize
62. Click ok
63. Wait till reconnection
64. After continuation of all apps:
65. Settings Connections Disable Wifi 
66. Settings Apps Search or find Google Play services open Permissions don't allow Nearby devices
67. Disable google play services
68. Stop google play services
69. Enable google play services
70. Settings Apps Search or find Google 
71. services open Permissions don't allow Nearby devices
72. Disable google 
73. Stop google
74. Enable google
75. Download and install the following apps from Google play store:
	- Sony Music Center
	- Sony Sound Connect
	- Bluetooth Codec Changer
	- Find my device
76. Enable developer options
77. Forget headphones in settings
78. 2 routes to go from here based on the Disable absolute volume dev setting you set
79. Settings developer options Disable absolute volume off
80. Open Bluetooth Codec Changer 
81. Buy Premium in the app
82. Allow nearby devices
83. Allow the connected headphones to be managed by bleutooth codec changer
84. Click okay on the android 14 bleutooth warning 
85. Click on okay about translating of the app
86. Click settings
87. Click profiles 
88. Click enable codec profiles
89. Click backup profiles
90. Import backup file
91. Select the file that is made for when the absolute volume setting is off
92. Click home to verify the codec profiles are imported succesfully 
93. When asked to disable battery optimization you do that for the Bluetooth codec changer app click on app's settings select battery select unrestricted
94. Go back to the app settings advanced disable 2-step switch note seems to work for Samsung phones other phones are not tested do your own research
95. Settings saved devices add conmected device
96. Settings saved devices device control ok
97. Settings auto switch add background service add service
98. Grant notification permission grant and allow
99. Default codec ldac
101. Default codec option 44.1 24 stereo save
102. LDAC playback quality optimized for sound quality (990 or 909kbps)
103. Click execution delay click okay click save
104. Volume control 100
105. Close the app
106. System settings developer options enable usb debugging
107. Connect your phone using a usb cable with data capability to your pc
108. Download ADB
109. adb shell appops set com.google.android.gms BLUETOOTH_SCAN ignore
110. adb shell appops set com.google.android.gms NEARBY_WIFI_DEVICES ignore
111. adb shell pm grant com.amrg.bluetooth_codec_converter android.permission.DUMP
112. If you use powershell do this command now:
113. while ($true) {
    Clear-Host
    adb shell dumpsys bluetooth_manager | Select-String "ldac"
 Start-Sleep -Seconds 2
}




114. Open bleutooth codec changer
115. settings ultra advanced enable adaptive sample rate beta click okay
116. Enable lock bits per sample
117. Enable adaptive sampling notification 
118. Disable adaptive sampling and only enable when you really need it

119. Settings developer options Disable absolute volume on
120. Open Bluetooth Codec Changer 
121. Buy Premium in the app
122. Allow nearby devices
123. Allow the connected headphones to be managed by bleutooth codec changer
124. Click okay on the android 14 bleutooth warning 
125. Click on okay about translating of the app
126. Click settings
127. Click profiles 
128. Click enable codec profiles
129. Click backup profiles
130. Import backup file
131. Select the file that is made for when the absolute volume setting is on
132. Click home to verify the codec profiles are imported succesfully 
133. When asked to disable battery optimization you do that for the Bluetooth codec changer app click on app's settings select battery select unrestricted
134. Go back to the app settings advanced disable 2-step switch note seems to work for Samsung phones other phones are not tested do your own research
135. Settings saved devices add conmected device
136. Settings saved devices device control ok
137. Settings auto switch add background service add service
138. Grant notification permission grant and allow
139. Default codec ldac
140. Default codec option 44.1 24 stereo save
141. LDAC playback quality optimized for sound quality (990 or 909kbps)
142. Click execution delay click okay click save
143. Volume control 100
144. Close the app
145. System settings developer options enable usb debugging
146. Connect your phone using a usb cable with data capability to your pc
147. Download ADB
148. adb shell appops set com.google.android.gms BLUETOOTH_SCAN ignore
149. adb shell appops set com.google.android.gms NEARBY_WIFI_DEVICES ignore
150. adb shell pm grant com.amrg.bluetooth_codec_converter android.permission.DUMP
151. Open bleutooth codec changer
152. settings ultra advanced enable adaptive sample rate beta click okay
153. Enable lock bits per sample
154. Enable adaptive sampling notification 
155. Disable adaptive sampling and only enable when you really need it

#### Google Play Services Interference Timing

You do **not** need to disable Nearby Devices or revoke Find My Device permissions immediately.

✅ During initial setup (SBC handshake, profile injection, LDAC training), Google’s override services do not interfere — as long as Developer Options are cleared and Music Center is inactive.

Only after the LDAC 990 profile is confirmed and stored in firmware should you disable:

- Nearby Devices for com.google.android.gms (via ADB or system settings)
- Assistant and Find My Device background access (optional)

This keeps device tracking and Assistant functional during initial setup.


