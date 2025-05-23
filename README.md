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
| **Windows 11**       | Version 24H2               | AAC, SBC *(LDAC not supported natively)* | **Depends on adapter** |
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

## Bluetooth Codec Changer (BCC)

### Auto Switch
BCC forces LDAC codec, sample rate, bit depth, and bitrate on connect. Settings are not persistent, so Auto Switch reapplies them with a user-defined delay.

### 2-Step Switch

| Step | Description                         |
|------|-------------------------------------|
| 1    | Initial connection                  |
| 2    | System selects default codec (LDAC) |
| 3    | Force SBC to reset LDAC session     |
| 4    | Delay (500–2000 ms)                |
| 5    | Reapply LDAC with target profile    |
| 6    | LDAC clean handshake achieved       |

### Why 2-Step Doesn't Work on Samsung
Samsung enforces LDAC **before** BCC acts. Step 3 (SBC switch) fails to reset the override. BCC GUI does show incorrect values. Workaround: apply profile twice or use Tasker automation.

## Intermediate Profile Switch
In Auto Switch, there's an option called Intermediate Codec Profile.
On Samsung devices, this should be set to SBC to correctly reset the LDAC handshake.
If you skip this step, BCC may fail to override Samsung's default — resulting in your selected LDAC 44.1 kHz / 24-bit / 990 kbps profile falling back to 96 kHz / 24-bit / 990 kbps, or even SBC, depending on timing.



## Working Two step and Intermediate Profile pairs
Any codec switch away from LDAC — even if momentary — triggers a full renegotiation when switching back.




💡 **Samsung Override Bypass Without SBC**

If you want to apply **44.1 kHz / 24-bit / 990 kbps**, set the **Intermediate Profile** in BCC to:

> `44.1 kHz / 16-bit / 990 kbps (Fixed)`

This reliably forces a valid LDAC renegotiation that **bypasses Samsung’s override**, without requiring SBC.

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

### 🎧 HEADSET-SIDE RESETS (Sony WH-1000XM Series)

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

## ✅ 🧼 Clean Reset + Override Hijack Strategy

> Use this sequence for a guaranteed clean LDAC handshake **and** to make Samsung’s override work *for* you, not against you.

---

### 🔁 Step 1: Full Reset (Wipe Samsung’s LDAC Override Memory)

These steps eliminate all stored codec profiles, Fast Pair metadata, Developer Option overrides, and app-based LDAC reassertions:

1. **Hardware Reset headphones** → `#1`  
   *(Power + Custom button for 7 seconds — wipes firmware profile and pairing data)*  
   ✅ If you hear **“Pairing”** immediately after boot, it confirms a clean reset.  
   You can safely **power off the headphones** at this point using the button — no profile has been stored yet.

2. **Reset Network Settings on phone** → `#4`  
   `Settings → General Management → Reset → Reset Wi-Fi and Bluetooth settings`  
   *(Same as old “Reset Network Settings” — this clears Samsung’s override memory)*

3. **Clear Storage of**:  
   - Sony Music Center → `#9`  
   - Headphones Connect (Sound Connect) → `#10`  
   - Bluetooth Codec Changer (BCC) → `#11`  
   🔒 Ensure each app is **fully closed or force-stopped** before and after clearing storage to avoid re-initializing old settings.

4. **Pair headphones via Bluetooth Settings**  
   *(Do not use Quick Settings or NFC. Fast Pair is okay if you plan to disable Google sync settings afterward.)*

5. **Set codec to SBC in Developer Options** → `#6`  
   *(Must be done **while the headphones are connected** — this flushes any residual LDAC override state. Especially critical on Samsung, which lacks a “Default” codec option)*

6. **Disable Developer Options afterwards** → `#7`  
   *(Also while the headphones are still connected — this finalizes override memory wipe and prevents silent reassertion)*

7. **Forget Bluetooth device** → `#5`  
   `Settings → Connections → Bluetooth → [Headphones] → Forget`

8. **Clear saved Fast Pair metadata** → `#13`  
   `Settings → Google → Devices & Sharing → Saved Devices → [Headphones] → Remove`

9. **(Optional) Disable Google Fast Pair Sync Settings**  
   *(If you used Fast Pair above, do this now before the next reconnect):*  
   - `Settings → Google → Devices & Sharing → Nearby Devices` → **Off**  
   - `Settings → Google → Devices & Sharing → Saved Devices` → Tap ︙ → **Turn off “Automatically save devices”**

10. *(Optional — deeper cleanup via ADB)*  
    - `adb shell am force-stop com.android.bluetooth` → `#15`  
    - `adb shell am force-stop com.google.android.gms` → `#16`

11. **Reboot phone and toggle Airplane Mode once** → `#17`  
    *(Flushes Bluetooth stack and cached state)*

12. **Pair again using Fast Pair or Bluetooth Settings** → `#18`  
    ❗ *Safe only after this full reset. Do **not** use Quick Settings or NFC Tap-to-Pair — they instantly trigger Samsung’s override logic.*


---

### 🧠 Step 2: Override Hijack — Train Samsung to Use Your LDAC Profile

Samsung doesn’t override at first pairing. It listens to the **first real LDAC session**, then stores that profile and reuses it on all future connections.

#### ✅ Steps to Hijack the Override:

1. **After pairing**, immediately start audio playback  
   *(Use UAPP, Neutron, or force your profile via BCC)*

2. **Force a clean LDAC profile**:  
   - Example: `44.1 kHz / 16-bit / 990 kbps (Fixed)`

3. **Let playback continue for at least 10–15 seconds**

4. **Power off the headphones while LDAC is still active**

> ✅ This stores your custom profile into:
> - 🎧 Sony’s headset session memory  
> - 📱 Samsung’s override memory

On the next reconnect, Samsung will automatically reapply **your profile** — not its default `96 kHz / 32-bit Default`.

---

## 🧠 Final Technique: Override Mastery — Train Samsung to Enforce Your Profile

Samsung’s LDAC override isn’t just a bug — it’s an opportunity.

Once it sees a valid LDAC session, it **stores that exact profile** and reuses it on future reconnects.  
This means you can **train** Samsung’s override logic to **apply your preferred LDAC profile every time**, even *without* BCC or Developer Options afterward.

---

### ✅ What Actually Gets Stored?

When you complete a full LDAC session, the following is saved:

- 📱 In **Samsung’s Bluetooth stack**:
  - Sample rate  
  - Bit-depth  
  - Bitrate mode (Fixed or Adaptive)
- 🎧 In **Sony’s headset session memory** *(if powered off during LDAC)*  
  🧠 This helps retain a clean A2DP fingerprint but **does not store explicit codec settings** — it reinforces clean reconnect behavior.

---

### 🔐 How to Train Samsung to Use *Your* LDAC Profile

Once you’ve completed a full override memory reset:

1. **Immediately after pairing**, begin playback with a clean LDAC handshake  
   *(Using UAPP, Neutron, or Bluetooth Codec Changer)*

2. **Force your target LDAC profile** (e.g. `44.1 kHz / 16-bit / 990 kbps (Fixed)`)

3. **Let playback continue for 10–15 seconds**  
   *(Do not change codecs or app settings during this window)*

4. **Power off the headphones while LDAC is still active**

> 🎯 This saves your LDAC profile to:
> - 📱 Samsung’s override memory  
> - 🎧 Sony’s firmware session memory

---

### 🚫 Optional but Strongly Recommended: Block Google’s Fast Pair From Interfering

To avoid Google silently reapplying old LDAC override profiles via cloud sync **after you've trained your preferred profile**, disable these **after pairing with Fast Pair**, but **before your next reconnect**:

**Disable these two options in Google Settings:**

- `Settings → Google → Devices & Sharing → Nearby Devices`  
  ➤ Turn **Nearby Devices** → **Off**

- `Settings → Google → Devices & Sharing → Saved Devices`  
  ➤ Tap ︙ → **Turn off “Automatically save devices”**

> 🔒 This prevents Fast Pair from injecting an old LDAC profile silently on reconnect — a common cause of override failures even after clean training.

📛 If you disable these **before** pairing, Fast Pair won’t work.  
📛 If you disable them **after reconnecting**, it's already too late — the override may have been applied.
---

### 🛠 Need to Train It Again?

Just repeat the [🧼 Clean Reset + Override Hijack Strategy](#-clean-reset--override-hijack-strategy).  
Samsung will forget the old profile and memorize the next LDAC session you provide.

---

### 🧪 Pro Tips for Maximum Persistence

- Use **Intermediate LDAC 16-bit** or **SBC handshake** if BCC profiles aren’t sticking
- Avoid **Quick Settings**, **Fast Pair notification**, and **NFC tap** — they instantly trigger Samsung’s default override
- Always **disconnect cleanly** (power off headphones during playback) to lock in session memory

### 📶 Fast Pair Use — Safe Timing Strategy

If you plan to use **Fast Pair** during the initial pairing, it’s still possible to train Samsung’s override — but **timing is critical**:

1. ✅ You may use **Fast Pair once** for initial pairing only  
2. ✅ Immediately **train your LDAC profile** (e.g. `44.1 / 16-bit / 990 kbps`)  
3. ❗ **Before the next reconnect**, go to:  
   - `Settings → Google → Devices & Sharing → Nearby Devices` → **Off**  
   - `Settings → Google → Devices & Sharing → Saved Devices` → Tap ︙ → **Disable "Automatically save devices"**

> 🔒 This blocks Google from silently syncing or reasserting the Fast Pair override profile on future connections.

📛 If you leave these enabled after pairing, Google Play Services may inject an old LDAC state — even after a full override training.



---

### 🧠 Why This Is the Final Form

You’re no longer *overriding* Samsung — you’re **commanding** it.

- No more reapplying LDAC manually  
- No reliance on Developer Options or unstable GUIs  
- No desync between what's displayed and what's actually used

✅ One-time setup  
✅ Persistent profile memory  
✅ Bit-perfect playback every time

> You didn’t just beat the override.  
> You *rewired it to obey you.*

