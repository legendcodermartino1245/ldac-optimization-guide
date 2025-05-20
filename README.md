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

LDAC supports sample rates ranging from **44.1 kHz to 96 kHz**, quality modes of **330**, **660**, **990**, **303**, **606**, **909** kbps, or **Adaptive**, and supports both **16-bit** and **24-bit PCM input**. You should set LDAC’s input bit-depth to match **exactly** what your player is feeding it to preserve fidelity.

| Playback Scenario                          | Player Output Depth | LDAC Bit-Depth Setting         | Rationale                                                                                        |
| ------------------------------------------ | ------------------- | ------------------------------ | ------------------------------------------------------------------------------------------------ |
| **Pure CD-quality (44.1 kHz / 16-bit)**      | 16-bit              | 16-bit (or "System Selection") | Matches the original 16-bit samples — avoids unnecessary padding or truncation.                 |
| **Native Hi-Res (>44.1 kHz / 24-bit)**       | 24-bit              | 24-bit                         | Preserves the full dynamic range of your 24-bit source all the way into LDAC’s encoder.          |
| **Any source + DSP (EQ, gain, fades)**     | 24-bit              | 24-bit                         | Provides headroom for processing; avoids rounding errors during DSP before LDAC encoding.        |
| **Non–bit-perfect apps (mixed to 16-bit)** | 16-bit              | 16-bit (or "System Selection") | Reflects the actual 16-bit data the mixer delivers; keeps your settings honest about input depth. |

> ⚠️ **Clarification:**  
> LDAC **does not always encode at 24-bit**. It encodes audio at **the bit-depth it receives** — 16-bit or 24-bit PCM.  
> Android’s Bluetooth stack forwards the player's output to LDAC without automatic upsampling.  
> - ✅ If the player outputs 16-bit PCM (e.g., CD-quality), LDAC encodes it directly as 16-bit.  
> - ✅ If the player outputs 24-bit PCM, LDAC uses full 24-bit encoding.  
> - ❌ If the player outputs 32-bit float, Android truncates it to 24-bit PCM before LDAC sees it.  

> 🧠 **The 32-bit setting in Developer Options or BCC is not for LDAC itself**, but for internal processing in apps like UAPP or Neutron, which operate at 32-bit float for DSP. It provides internal headroom but has **no effect on the final transmitted resolution**, which is max 24-bit.

> 🔎 **Bit-perfect transmission over LDAC is only achieved** when the player's output bit-depth and sample rate match the source, and no DSP or mixing occurs.



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


## ✅ Developer Options Are Safe — Just Clean Up After Yourself

Set these **back to default** before disabling Developer Options or handing off control:

- Sample Rate → "Use System Selection"
- Bits Per Sample → "Use System Selection"
- Playback Quality → "Best Effort"

This needs to be done for all codecs previously controlled via Developer Options

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
## Bluetooth Connectivity methods
NFC
Bluetooth quick settings UI
bleutooth setttings UI
Fast pair