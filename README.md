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

9b. **Temporarily enable Nearby Devices for Google Play Services**  
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

The Sony WH-1000XM5 supports **multipoint Bluetooth**, allowing connection to **two devices simultaneously**.  
Typically, this works seamlessly because **Windows defaults to SBC or AAC**, while **Android uses LDAC**, avoiding codec conflict.

However, advanced users may configure **LDAC on both devices** — for example, on **Android and Windows**, or even **two Android devices**.

This presents a unique challenge:

> **Can we build a stable configuration where both devices use LDAC without triggering codec fallback, stutters, or renegotiation?**

Simultaneous LDAC use on two devices depends on system configuration, codec settings, and volume/control options.

---

## 🛠️ Setup Prerequisites

- **AVRCP 1.6 on both devices**  
  - On Android: enable it once in Developer Options — it persists even after Dev Options are turned off.  
  - On Windows: use a registry edit.

```reg
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Bluetooth\Audio\AVRCP]
"AvrcpTargetVersion"=dword:00000006
```
Lets breakdown what you need in order to succeed
AVCRP 1.6 on both devices. It can be enabled in dev options once and isnt affected by disabling dev options and is always set unnless you change it again
AVCRP 1.6 on windows can be activated by using reg edits.
for controlling Absolute volume on windows I recommend Bluetooth Tweaker
## 🔄 LDAC Multipoint Stability Factors

| Factor                        | Recommendation                                                 | Purpose / Effect                                                  |
|------------------------------|-----------------------------------------------------------------|-------------------------------------------------------------------|
| Codec settings               | Mirror LDAC settings on both devices                            | Prevents codec renegotiation or fallback to SBC/AAC               |
| Playback format              | Match sample rate and bit depth (e.g., 96 kHz / 24-bit)         | Avoids stutters and DSP resync delays                            |
| Absolute Volume              | Use **ON on Windows**, **OFF on Android**                      | Maintains volume sync on Windows and codec control on Android    |
| Volume level                 | Set the **same volume** on both devices (e.g., 85%)             | Prevents loudness jumps and smoothens handoff behavior           |
| AVRCP version                | Use **1.6** on both Android and Windows                        | Ensures faster media control switching and improved metadata sync |
| Playback state coordination  | **Pause playback** on the inactive device                      | Prevents LDAC renegotiation and session fight during handoff      |
| Fast Pair behavior           | Disable **"Automatically save devices"** in Android settings    | Prevents Fast Pair from reapplying stale codec/AV states          |
| BCC timing logic             | Add **intermediate profile (e.g., SBC or LDAC 16-bit)** before final LDAC | Ensures clean codec handshake and profile lock-in                 |



reparing required to swtich from av off and on on windows
i feel like i cant recommend a best combination i feel the avcrp version 1.6 on windows is too badly implemented is that valid

## 🎧 Multipoint AV/AVRCP Combination Matrix (LDAC on Both Devices)

| # | Setup Type         | Device A | Device B | Absolute Volume (A / B) | AVRCP Version (A / B) |
|---|--------------------|----------|----------|--------------------------|------------------------|
| 1 | Android + Android  | Android  | Android  | ON / ON                  | 1.6 / 1.6              |
| 2 | Android + Android  | Android  | Android  | OFF / OFF                | 1.6 / 1.6              |
| 3 | Android + Android  | Android  | Android  | OFF / ON                 | 1.6 / 1.6              |
| 4 | Android + Windows  | Android  | Windows  | OFF / ON                 | 1.6 / 1.6              |
| 5 | Android + Windows  | Android  | Windows  | ON / ON                  | 1.6 / 1.6              |
| 6 | Android + Windows  | Android  | Windows  | OFF / OFF                | 1.6 / 1.6              |
| 7 | Android + Windows  | Android  | Windows  | ON / OFF                 | 1.6 / 1.6              |
> ⚠️ **Note:** All combinations in this table were tested using **AVRCP 1.6** only.  
> I did not test other AVRCP versions, as 1.6 provided noticeably faster switching behavior between devices.





