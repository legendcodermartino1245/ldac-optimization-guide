# LDAC Done Right

There is a lot of misconception about LDAC and how to properly configure it on different operating systems (Windows and Android for example.) This guide will focus on both and will include strategies which help you get the best sound possible. I tried to make it as practical as possible to replicate. To make troubleshooting easier I have clearly documented the steps needed to set everything back to defaults for full transparency.

## My setup and the hardware I used during the making of this guide:

| Device              | OS / Firmware             | Supported Codecs                       |
|---------------------|---------------------------|----------------------------------------|
| **Samsung S22 Plus** | One UI 6.1 (Android 14)    | LDAC, SSC, AptX, AAC, SBC               |
| **Samsung S24**      | One UI 6.1 (Android 14)    | LDAC, SSC, AptX, AAC, SBC               |
| **Windows 11**       | Version 24H2               | AAC, SBC *(LDAC not supported natively)* |
| **Windows 10**       | Version 22H2               | AAC, SBC *(LDAC not supported natively)* |
| **Sony WH-1000XM5**  | 2.4.1                      | LDAC, AAC, SBC                          |
| **Sony WH-1000XM3**  | 4.5.2                      | LDAC, AptX, AAC, SBC                    |

> 📌 *Note: On Windows, LDAC support requires specific Bluetooth drivers or third-party implementations (e.g. CSR Harmony stack or alternative USB dongles). This guide focuses on standard OS behavior unless otherwise noted.*

## Inner workings of LDAC

LDAC supports sample rates ranging from **44.1 kHz to 96 kHz**, quality modes of **330**, **660**, **990 kbps**, or **Adaptive**, and always uses **24-bit** precision internally. However, you should set LDAC’s input bit-depth to match **exactly** what your player is feeding it:

| Playback Scenario                          | Player Output Depth | LDAC Bit-Depth Setting         | Rationale                                                                                        |
| ------------------------------------------ | ------------------- | ------------------------------ | ------------------------------------------------------------------------------------------------ |
| **Pure CD-quality (44.1 kHz / 16-bit)**      | 16-bit              | 16-bit (or "System Selection") | Matches the original 16-bit samples—no unnecessary padding or noise.                             |
| **Native Hi-Res (>44.1 kHz / 24-bit)**       | 24-bit              | 24-bit                         | Preserves the full dynamic range of your 24-bit source all the way into LDAC’s encoder.          |
| **Any source + DSP (EQ, gain, fades)**     | 24-bit              | 24-bit                         | Provides headroom for processing; avoids rounding errors during DSP before LDAC encoding.        |
| **Non–bit-perfect apps (mixed to 16-bit)** | 16-bit              | 16-bit (or "System Selection") | Reflects the actual 16-bit data the mixer delivers; keeps your settings honest about input depth. |

> ⚠️ **Clarification:**  
> LDAC always encodes audio at **24-bit precision** — never higher.  
> Selecting **32-bit** in Developer Options or BCC has **no effect** on actual audio quality; it’s just an Android-side format wrapper.  
> Even if an app outputs **32-bit float** (as many do), Android’s audio system **downmixes it to 24-bit PCM** before it reaches the LDAC encoder.  
> There is **no resolution benefit** from using 32-bit with LDAC — anything above 24-bit is **truncated or dithered** during encoding.  
> 🧠 The **32-bit setting is not for LDAC itself**, but for **apps like UAPP or Neutron** that internally process audio at 32-bit float.  
> It gives these apps more headroom and avoids precision loss before the audio is handed off to Android’s Bluetooth stack, which will always deliver 24-bit to LDAC.


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
- **Bitrate:** Adaptive (330–990 kbps)

This override happens **before** your device finishes establishing the Bluetooth session.  
However, it can be **reliably bypassed** by forcing a full codec renegotiation after connection, see  The Real LDAC Bug section on how to do it

✅ BCC and other apps **can override** Samsung's initial profile — but only if they trigger a full codec reset after the override is applied.

🧠 *Samsung’s override is not permanent — it’s just the default LDAC handshake. What matters is whether your LDAC session gets renegotiated correctly after that handshake.*

> Developer Options may temporarily display "Playback Quality: Default" when Samsung’s override is active.


## ✅ Developer Options Are Safe — Just Clean Up After Yourself

Set these **back to default** before disabling Developer Options or handing off control:

- Sample Rate → "Use System Selection"
- Bits Per Sample → "Use System Selection"
- Playback Quality → "Best Effort"

This needs to be done for all previously via developer options controlled codecs 

> ✅ *You can safely leave Developer Options enabled — just make sure all codec-related settings are returned to default. No harm is done if they’re inactive.*

> Leaving overridden settings can cause LDAC to renegotiate, trigger Samsung's override again, or break BCC's control.

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

💡 **Samsung Override Bypass Without SBC**

If you want to apply **44.1 kHz / 24-bit / 990 kbps**, set the **Intermediate Profile** in BCC to:

> `44.1 kHz / 16-bit / 990 kbps (Fixed)`

This reliably forces a valid LDAC renegotiation that **bypasses Samsung’s override**, without requiring SBC.

✅ Benefits:
- No need for 2-step switching
- Avoids fallback to 96 kHz / 24-bit or SBC
- Works even when BCC switches quickly (short or 0ms delay)
- Requires no Music Center involvement

👁️ **BCC GUI Reflects External Codec Changes**

As of the latest update, BCC’s GUI reflects LDAC codec changes made by **other apps or the system**, such as:

- UAPP  
- Developer Options  
- Sony Music Center  
- System-level negotiation (e.g. Fast Pair)

⚠️ On Samsung, GUI accuracy is **only reliable when 2-Step Switch is enabled**.  
However, **2-Step must be disabled** to make LDAC profile switching stable — especially when using intermediate profile logic (e.g., 44.1 kHz 16-bit → 24-bit).  
This means the GUI will often **show incorrect sample rate, bit depth, or bitrate** even though the override succeeded.

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
## Adaptive bitrate ldac
Non standard codec values other than 990 660 330 303 606 909 can appear here when adaptive bitrate is on like 452
The moment a stream starts, LDAC selects a bitrate tier (adaptive logic kicks in).
LDAC adaptive bit rate adjustments: 2 is a key indicator of when LDAC Adaptive has finished ramping and stabilized at a specific bitrate tier.


### 📊 LDAC Adaptive Mode Stability Matrix

| Sample Rate | Bit Depth         | Stability    | Observations                                                       |
|-------------|-------------------|--------------|--------------------------------------------------------------------|
| 44.1 kHz    | 16-bit / 24-bit   | ❌ Unstable   | Frequently drops to 606 / 303 kbps, rarely sustains 909 kbps       |
| 48 kHz      | 16-bit / 24-bit   | ✅ Stable     | Consistently ramps to and holds 990 kbps                           |
| 88.2 kHz    | 16-bit / 24-bit   | ⚠️ Moderate   | Slightly more stable than 44.1, but not reliable at high bitrates  |
| 96 kHz      | 16-bit / 24-bit   | ✅ Stable     | Ramps cleanly and holds 990 kbps with minimal adjustments          |
