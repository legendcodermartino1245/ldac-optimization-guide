#  High-Resolution Audio
###  General Configuration Advice
- **App-Specific Profiles in BCC** require **Usage Data Access** to function.
- **64-bit mode in Neutron** breaks **BCC Adaptive Sample Rate Switching**.
- Adaptive Sample Rate in BCC works on the **Android audio mixer**, not at app-level.
- High-res players that bypass the Android mixer (like **UAPP** and **Neutron**) **interfere with BCC** when using exclusive/high-res output.
- **Do not combine Tasker-based switching with BCC Adaptive Sample Rate** — they conflict.
- BCC Adaptive Sample Rate Switching shows full codec info in the notification:  
  `LDAC • 48000 • 16 • Stereo • Optimized for audio quality (990/909kbps) • S(48000 Hz)`
- **Tasker cannot enhance Adaptive Sample Rate switching** — do not pursue this path.
---
###  Behavior of High-Res Audio Apps
- **UAPP (USB Audio Player PRO)**:
  - Works best when LDAC is already set via BCC or Developer Options.
  - Opening the app resets the codec to **UAPP’s internal configuration**.
  - Let UAPP handle **automatic sample rate switching** — disable Adaptive Sample Rate in BCC.
  - Set UAPP LDAC resolution to 24 bit
  - If BCC was active previously, you **must fully close and reopen UAPP** for it to take control of LDAC again.
  - **Auto Switch in BCC** interferes with UAPP — use App-Specific Profiles for all **other apps**, and **let UAPP handle its own LDAC settings**.
  - **Adaptive Sample Rate in BCC can crash UAPP** if LDAC settings are also active in BCC.
- **Qobuz**:
  - Set streaming quality to **max 96 kHz** to avoid compatibility issues.
  - Adaptive Sample Rate should be enabled when **streaming or playing offline files**, so native track sample rate is used.
  - After offline listening, **disable Adaptive Sample Rate in BCC** for performance reasons.
- **Neutron**:
  - Use **64-bit OFF** to allow BCC Adaptive Sample Rate to work.
  - 64-bit mode uses its own direct audio engine, bypassing Android’s mixer entirely. This prevents BCC from seeing or adapting to the sample rate.
  - With 64-bit ON, Neutron uses its own audio path, **bypassing BCC**.
- **Roon / Roon ARC**: Not compatible with Adaptive Sample Rate switching in BCC.
- **Music Center**:
  - If music center has got the capability to change codec you gave that permission it will override the uapp codec and bit rate by the setings set by msuic center as soon as you open music center 
---
###  Apps Supported by BCC Adaptive Sample Rate Switching
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
###  Apps NOT Supported by Adaptive Sample Rate in BCC
| Unsupported Apps                    |
|-------------------------------------|
| USB Audio Player PRO (UAPP)         |
| Neutron Player (64-bit ON)          |
| Roon                                |
| Roon ARC                            |
---
###  Adaptive Sample Rate Switching – Key Facts
- Switching **does not change codec**, only sample rate.
- Switching **takes 1–3 seconds** to apply after playback starts (in BCC).
- In UAPP, **sample rate is applied instantly** at playback time.
- Adaptive Sample Rate works **at connect-time** in BCC.
- BCC is best used for **regular media apps**, not exclusive high-res output apps.
- Use **App-Specific Profiles** in BCC for best LDAC 990 performance **outside UAPP**.
---
###  BCC App Delay Settings (Handshake Timing)
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
###  Codec Negotiation Strategy
Use **Intermediate Profile Switching** in BCC:
1. Connect as **SBC** to force a clean handshake.
2. Immediately switch to **LDAC 990kbps** (or preferred fixed profile).
This:
- Cleans the override.
- Trains the firmware profile.
- Allows for stable codec persistence.
---
###  Best Practices Summary
- Only **one** hi-res player (UAPP or Neutron) should be open at a time.
- Adaptive Sample Rate in BCC **doesn’t touch codec**, only sample rate.
- UAPP’s Adaptive Switching **does change codec and sample rate** — preferred method.
- Avoid App-Specific Profiles for **media apps** (Spotify, YouTube) — impractical.
- Use App-Specific Profiles for **Qobuz, Tidal, etc.**, to ensure **CD-quality LDAC** outside UAPP.
---
###  Mastering > Resolution
> Great sound comes from **great mastering**, not just high-resolution formats.  
> Only choose high-res audio if the **mastering is worth it**.  
> Use tools like **Spek** on Windows to verify high-res masters.
###  Adaptive Sample Rate Switching in BCC — What It Actually Does
When **Adaptive Sample Rate Switching** is enabled in **Bluetooth Codec Changer (BCC)**, it:
- Monitors the **actual playback sample rate** from the **Android audio mixer**
- Dynamically re-applies the LDAC codec profile using that exact sample rate
- Ensures LDAC **matches the app’s output sample rate**, giving you sample-accurate playback without having to manually set it
This allows LDAC to track **44.1 kHz**, **48 kHz**, or **96 kHz** content automatically — ideal for **streaming apps** that change resolution depending on the source.
---
###  Works With These App Types
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
###  Doesn’t Work With These Apps
Apps that bypass the Android audio mixer via exclusive or Hi-Res mode:
- USB Audio Player PRO (UAPP)  
- Neutron (64-bit mode ON)  
- Roon / Roon ARC  
- Poweramp (Hi-Res mode)
These apps **control the sample rate internally**, so BCC **cannot detect or follow** the actual playback resolution.
---
###  Key Takeaways
- BCC Adaptive Sample Rate switching **does not change the codec** — only the **sample rate**.
- It applies **after playback begins**, usually within **1–3 seconds**.
- It's the best way to keep LDAC sample-accurate **without exclusive mode**, as long as the app doesn’t bypass the mixer.
>  For bit-perfect output with apps like UAPP or Neutron in Hi-Res mode, **disable Adaptive Sample Rate Switching** in BCC — let the app control LDAC directly.

- **"Use What is Currently Active"** in UAPP under **Bluetooth Codec**, **BT Sample Rate**, **LDAC Quality**, and **LDAC Resolution** reflects the **codec settings that were last applied at the system level**.
  - This includes any LDAC configuration previously set by **BCC**, **Developer Options**, or another app.
  - UAPP does **not override** those settings unless it regains exclusive codec control — this usually requires **fully restarting UAPP** or **reconnecting the headphones** with UAPP already open.
