# LDAC Done Right
- [LDAC Done Right](#ldac-done-right)
- [My setup and the hard and software I used during the making of this guide](#my-setup-and-the-hard-and-software-i-used-during-the-making-of-this-guide) 
- [LDAC Configuration Matrix Fixed](#ldac-configuration-matrix-fixed)
- [LDAC Configuration Matrix Adaptive](#ldac-configuration-matrix-adaptive)
- [Samsung LDAC Override Stack](#samsung-ldac-override-stack)
- [LDAC Codec Negotiation & Profile Generation](#ldac-codec-negotiation--profile-generation)
- [Headphone & Device Factors](#headphone--device-factors)
- [Phone Settings That Affect Codec Negotiation](#phone-settings-that-affect-codec-negotiation)
- [System Stack Behavior & Profile Storage](#system-stack-behavior--profile-storage)
- [App Behavior That Influences Codec Negotiation](#app-behavior-that-influences-codec-negotiation)
- [Samsung Codec Behavior](#samsung-codec-behavior)
- [AAC ≠ Neutral on Reconnect — It's Just Another Override Pathway](#aac--neutral-on-reconnect--its-just-another-override-pathway)
- [Basic setup from start](#basic-setup-from-start)
- [macOS and iPhone – LDAC Status](#macos-and-iphone--ldac-status)
- [Linux](#linux)


There is a lot of misconception about LDAC and how to properly configure it on different operating systems (Windows and Android for example.) This guide will focus on both and will include strategies which help you get the best sound possible. I tried to make it as practical as possible to replicate. To make troubleshooting easier I have clearly documented the steps needed to set everything back to defaults for full transparency.

## My setup and the hard and software I used during the making of this guide:

| Device              | OS / Firmware             | Supported Codecs                        | Bluetooth Version     |
|---------------------|---------------------------|-----------------------------------------|------------------------|
| **Samsung S22 Plus** | One UI 6.1 (Android 14)    | LDAC, SSC, AptX, AAC, SBC                | **Bluetooth 5.2**      |
| **Samsung S24**      | One UI 6.1 (Android 14)    | LDAC, SSC, AptX, AAC, SBC                | **Bluetooth 5.3**      |
| **Windows 11**       | Version 24H2               | AAC, SBC *(LDAC not supported natively)* | **Bluetooth 5.3**      |
| **Windows 11**       | Version 24H2               | AAC, SBC *(LDAC not supported natively)* | **Bluetooth 4.2**      |
| **Windows 10**       | Version 22H2               | AAC, SBC *(LDAC not supported natively)* | **Depends on adapter** |
| **Sony WH-1000XM5**  | Firmware 2.4.1             | LDAC, AAC, SBC                           | **Bluetooth 5.2**      |
| **Sony WH-1000XM3**  | Firmware 4.5.2             | LDAC, AptX, AAC, SBC                     | **Bluetooth 4.2**      |
>  *Note: On Windows, LDAC support requires specific Bluetooth drivers or third-party implementations (e.g. CSR Harmony stack or alternative USB dongles). This guide focuses on standard OS behavior unless otherwise noted.*

---
| Application | Version Tested |
|--------------|----------------|
| Bluetooth Codec Changer (BCC) | 1.7.1 |
| Sony Headphones Connect | 12.1.0 |
| Sony Music Center | 7.5.1 |
| Tasker | 6.5.9 |
| Tasker Settings | 1.7.1 |
| AutoNotification | 4.3.1 |
| Neutron Player | 2.26.0 |
| USB Audio Player PRO (UAPP) | 7.0.6.7 |
| Wavelet | 25.03 |
| BluetoothGoodies Alt Driver | 1.6.0.54 (Preview) |
| Bluetooth Tweaker | 1.4.8.1 |
| Spek | 0.8.5 |
---
## Samsung LDAC Override Stack

Samsung injects its own LDAC codec profile at the very start of every Bluetooth handshake. This override occurs **before any user-defined codec preference applies**, and operates entirely within Samsung’s Bluetooth stack logic.

- **Sample Rate:** 96 kHz  
- **Bit Depth:** 32-bit (stack-reported)  
- **Bitrate:** Default — practically Adaptive

### 🔧 Override Injection Behavior

- **Injection Timing:**  
  Immediately during A2DP session open (handshake phase).

- **Injection Priority:**  
  - Always applies at connect.
  - Can only be cleared post-handshake via controlled codec renegotiation.
  - Applies independently of any stored LDAC profile inside the headphone firmware, but firmware profiles may still influence the final codec state after negotiation depending on training and handshake timing.

- **Firmware Persistence:**  
  - The override itself is not firmware-persistent.
  - It applies dynamically at every Bluetooth connect event.
  - Headphone firmware profile storage operates separately and may reassert codec parameters after override injection.

- **Negotiation Level:**  
  - Override operates **below** Android A2DP codec negotiation APIs.
  - Developer Options LDAC settings have no effect at handshake.
  - BCC and system APIs cannot preempt override at connect — only post-handshake renegotiation is possible.

- **Controlled Override Defeat:**  
  - Override remains active until renegotiation occurs during the active connection.

---

✅ **Core Principle:**  
> Samsung Override = Handshake Default → Defeat requires codec renegotiation after handshake.

> ⚠ **Samsung Override Display Behavior:**
>
> - At handshake:  
>    - **Developer Options = Best Effort (Adaptive Bit Rate)**  
>    - **Bluetooth Codec Changer = Default**















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




# macOS and iPhone – LDAC Status
**macOS** and **iOS**, and there is no known workaround due to Apple’s closed ecosystem.

# Linux
Dont use Pulseaudio use Pipewire instead





# 🔄 Samsung LDAC Override Behavior — *Sound Connect: Prioritize Stable Connection*

## 📌 Scenario  
When you select **`Prioritize Stable Connection`** in Samsung's **Sound Connect** settings:

| 🔧 Setting / Behavior                 | Result                                                                 |
|--------------------------------------|------------------------------------------------------------------------|
| `Prioritize Stable Connection`       | ✅ **Enabled**                                                          |
| **LDAC Toggle (Bluetooth Settings)** | ❌ **Disabled & Hidden** — LDAC cannot be enabled manually              |
| **HD Audio Toggle**                  | ✅ **ON and changeable** — remains fully interactive                    |
| **Active Codec**                     | 🎯 **AAC is selected** — injected by Samsung override                  |
| **LDAC Codec (Developer Options)**   | ✅ **Visible** — LDAC still appears in the codec list                   |
| **LDAC Quality (Developer Options)** | ⚠️ **Greyed Out** — quality not selectable because LDAC is not active  |

---
## 🧠 Explanation

- Selecting `Prioritize Stable Connection` **removes LDAC from the connection path** by hiding its toggle.
- **HD Audio remains ON** and can still be toggled, but **AAC** is negotiated as the active codec due to Samsung’s override.
- **LDAC remains visible** in Developer Options, but the **quality selector becomes greyed out** simply because:
  > LDAC is **not the current codec**, not because it's "blocked."

  This is a key distinction — the quality selector only becomes active **after LDAC is successfully negotiated**.

---

## ✅ Summary

- `Prioritize Stable Connection` = 🔒 **LDAC blocked by toggle**, 🎯 **AAC negotiated**
- HD Audio = ✅ **On and user-toggleable**
- LDAC toggle = ❌ Hidden from UI
- LDAC codec = ✅ **Visible in Developer Options**
- LDAC quality = ⚠️ **Greyed out (no LDAC session active)**
- Override status = 🔁 **AAC forced by Samsung override**

---

## 📎 Notes

- To use LDAC again, switch to `Prioritize Sound Quality` and manually re-enable LDAC in Bluetooth settings.
- Even then, Samsung override will likely inject **LDAC Adaptive** unless defeated (via BCC, bit-depth switch, SBC trick, etc.).
- Most important insight:
  > **LDAC Quality greyed out ≠ LDAC blocked — it just means LDAC isn’t in use yet.**
 

## 🔁 LDAC Override Reinjection via Sound Connect Toggle

### 🧠 Description

When using **Samsung Sound Connect**, the following sequence causes LDAC to become disabled and forces Samsung’s override when re-enabled:

1. Open **Sound Connect**.
2. Select **Prioritize Stable Connection**.
3. Then switch back to **Prioritize Sound Quality**.

This silently disables LDAC (toggle off), even though **HD Audio stays enabled**.

Manually re-enabling LDAC at this point **triggers Samsung’s override**.

---

### ⚙️ Technical Behavior

| Action                                      | Result                                                                 |
|---------------------------------------------|------------------------------------------------------------------------|
| `Prioritize Stable Connection`              | Disables LDAC, hides its toggle in Bluetooth settings                  |
| `Prioritize Sound Quality`                  | Makes LDAC toggle visible again, but now set to **OFF**                |
| Manual re-toggle of LDAC                    | Triggers override due to HD Audio being **ON**                         |
| LDAC quality setting (in Developer Options) | Becomes visible again — proof that Samsung override is re-applied      |

---

### 📌 Implications

- **HD Audio remains ON** throughout the toggle process.
- Re-enabling LDAC while HD Audio is ON causes **Samsung’s LDAC override** to activate (typically Adaptive 96 kHz).
- Any prior clean LDAC profile (e.g., 44.1 kHz Fixed) is **overwritten**.
- This shows that **manual LDAC toggling is enough to retrigger override**, even without reconnecting.

---

### ✅ Recommendations

- **Never manually toggle LDAC** after using Sound Connect — override will be injected.
- Use tools like **Bluetooth Codec Changer (BCC)** to apply desired LDAC profiles **without triggering override**.
- Consider using a **bit-depth mismatch exploit** (e.g., LDAC 16-bit → LDAC 24-bit) to defeat the override via codec switching logic.
- Avoid enabling **HD Audio**, unless override injection is desired.
- Keep **Sound Connect** on "Prioritize Sound Quality" without toggling between modes if a clean profile is already trained.

---

### 🧪 Example Use Case

If you're trying to maintain a stable **LDAC 44.1 kHz 990kbps Fixed** profile:

- Toggling Sound Connect to "Stable" and then back to "Sound Quality" will reset the toggle.
- Manually enabling LDAC again will **lose your fixed profile** and inject **Samsung’s override**.
- Only automated switching (via Tasker/BCC) or bit-depth trick logic will retain codec integrity.

---



## 📶 Samsung LDAC Override Codec Table (Including SBC Fallback)

| Media Audio | HD Audio | LDAC Toggle   | Codec Selected | Override Active | Notes                                                   |
|-------------|----------|----------------|----------------|------------------|----------------------------------------------------------|
| On          | On       | On             | LDAC           | Yes              | Override selects LDAC (typically 96 kHz Adaptive)        |
| On          | On       | Off            | AAC            | Yes              | LDAC unavailable; override falls to AAC                  |
| On          | On       | Hidden         | AAC            | Yes              | LDAC not exposed; override selects AAC                   |
| On          | On       | Greyed out     | AAC            | Yes              | LDAC visually present but blocked; override selects AAC  |
| On          | Off      | Any            | SBC            | No               | HD Audio OFF disables override; fallback codec is SBC    |
| Off         | On       | Any            | No audio       | No               | No A2DP session; override logic not triggered            |

---

## 🔍 Codec Requirements (Full Context)

| Codec | Requires Media Audio | Requires HD Audio | Requires LDAC Toggle |
|--------|----------------------|-------------------|------------------------|
| LDAC   | Yes                  | Yes               | Yes                   |
| AAC    | Yes                  | Yes               | No                    |
| SBC    | Yes                  | No                | No                    |

## ⚠️ Additional Notes

- 🎧 **Input Device toggle** (in Bluetooth settings)  
  → *Has no impact on codec negotiation or override behavior.*  
  It only signals whether the device can act as a **microphone input**, not an audio sink.
AAC is preferred by Samsung over SBC when HD Audio is ON but LDAC toggle is OFF — even though SBC would normally be default.



## 📱 Developer Options — LDAC Quality Toggle Behavior

### ✅ LDAC Quality *Selectable* in Developer Options
- **Meaning:**  
  Samsung **override is triggered**, because it *always* triggers when HD Audio is ON and the LDAC toggle is ON.
- **Reasoning:**  
  The LDAC quality dropdown only becomes **selectable** when LDAC is the active codec. Since Samsung enforces LDAC under these conditions, its presence implies the override logic is in effect.
- **Implication:**  
  If you can select LDAC quality, you're seeing the result of Samsung's override stack — **not user-defined codec negotiation**.

---

### ⚪ LDAC Quality *Greyed Out* in Developer Options
- **Meaning:**  
  This state is **ambiguous** and doesn't indicate anything conclusive about codec negotiation or override status.
- **Why it happens:**  
  The LDAC quality option can appear greyed out when:
  - No device is connected
  - LDAC toggle is OFF
  - HD Audio is OFF
  - Sound Assistant or Sound Connect is limiting LDAC usage
- **Implication:**  
  A greyed-out LDAC quality setting means **LDAC is not in use**, but it doesn’t mean override logic is inactive — it simply means the codec isn’t engaged.

---

### 📊 Summary Table

| LDAC Quality in Dev Options | What It Means                                 | Samsung Override Status      |
|-----------------------------|-----------------------------------------------|-------------------------------|
| ✅ Selectable               | LDAC is active due to system-level enforcement | ✅ Override is **triggered**  |
| ⚪ Greyed Out               | LDAC is not active or blocked by config        | ❓ Override status **unknown** |

---

> 💡 **Note:**  
> The real proof of override lies in whether *you* chose the codec — or Samsung did.  
> If you didn’t change anything and LDAC is active, the override logic won.

LDAC Quality is always greyed out when no device is connected — even if override is armed, because there is no A2DP session to inject into.






## 🧠 Samsung LDAC Override: Armed by HD Audio, Injected on A2DP Connect

Even **when no Bluetooth device is connected**, Samsung’s override system is **already active** as long as `HD Audio` is enabled.

- 🧱 The **SBC codec** appears as default, since no A2DP device is connected.
- ⚠️ The **LDAC Quality** setting is greyed out — **not because LDAC is inactive**, but because Samsung’s override profile **hasn’t been injected yet**.

This confirms:

- ✅ **Samsung’s override logic is always armed when `HD Audio` is ON**
- 🧠 **Override injection only occurs during the A2DP handshake**
- 🔁 You **never toggle `HD Audio`** unless absolutely necessary, because Samsung will **automatically re-enable it** after most disconnections

---

### 🔁 What Happens on Device Connect

Once an LDAC-capable device connects and both `Media Audio` and `HD Audio` are enabled:

- 🔴 If **LDAC is enabled**, Samsung immediately **injects its override LDAC profile**:
  - `96 kHz / Adaptive / 32-bit` (typical)
- 🔴 If **LDAC is disabled**, Samsung selects **AAC**, *not* SBC.
- ⚠️ The **LDAC Quality** setting remains **greyed out** — override is active.
- 🧼 A **clean LDAC profile is never auto-applied** — only possible via manual trick or override defeat.

---

> 📶 Enabling `HD Audio` unlocks access to **higher-quality Bluetooth codecs**, including:
>
> - **LDAC**
> - **AAC**
> - **aptX / aptX HD** (if supported by both device and headphones)
>
> However, `HD Audio` does **not control which codec is selected** during connection.  
> That decision is made by the system during the **A2DP handshake**, where Samsung’s override may still inject its preferred codec — typically **LDAC Adaptive** or **AAC**.

---

### 📊 Samsung Override Behavior Table

| Condition                            | Override Logic State | Codec Displayed  | LDAC Quality Setting |
|-------------------------------------|-----------------------|------------------|-----------------------|
| No device connected, HD Audio ON    | 🟡 Armed              | SBC              | Greyed out            |
| Device connects, LDAC ON            | 🔴 Injects override   | LDAC Adaptive    | Greyed out            |
| Device connects, LDAC OFF           | 🔴 Injects override   | AAC              | Hidden                |
| Device connects, HD Audio OFF       | ⚪ Inactive           | SBC              | Hidden                |
| Override defeated (e.g. SBC trick)  | 🟢 Bypassed           | LDAC Fixed       | Selectable            |

---

## 🔥 Samsung Override Is Inevitable

You **can’t prevent it** — you can only **defeat or manipulate it**:

- 🎭 Use **profile switch tricks**:
  - Start with `SBC`, then manually or automatically switch to `LDAC 16-bit`
  - Finalize with `LDAC 990kbps / Fixed`
  - This sequence hijacks the A2DP handshake *before* Samsung injects its override
- 🤖 Use **Bluetooth Codec Changer (BCC)**:
  - Automate codec negotiation steps
  - Ensure `LDAC 990kbps` is injected *before* the system applies override logic
  - Works best when paired with Fast Pair training or Developer Options cleanup
- 🧠 Remember:  
  The **first clean codec** applied after `A2DP connect` is the one **stored in Sony firmware**  
  (unless Samsung’s override **beats you to it**)

---

> ⚠️ **HD Audio = codec access, not codec choice**  
> Samsung’s stack **still decides** the initial codec — unless **you intercept it** with a forced profile switch.






# 🔓 Defeating Samsung's LDAC Override Instantly

Samsung injects a **default LDAC override profile** during A2DP connection **if `HD Audio` is enabled**.  
This override forces:

- ✅ LDAC codec (even if disabled prior)
- ⚠️ Quality = **Adaptive 96 kHz**
- 🚫 Developer Options LDAC settings **ignored**

---

## ✅ Override Defeat Rule

> **Changing *any* LDAC or codec-related setting after connection defeats the Samsung override.**

---

## 🧪 Verified Setting Changes That Defeat Override

| Setting Changed                   | Override Defeated | Notes                                                  |
|----------------------------------|--------------------|---------------------------------------------------------|
| LDAC Quality (e.g., 990kbps)     | ✅ Yes             | Fastest and most reliable override break               |
| Sample Rate (Dev Options)        | ✅ Yes             | Forces re-negotiation with new values                  |
| Bit Depth (Dev Options)          | ✅ Yes             | Applies new bit depth and reasserts codec config       |
| Codec Switch (e.g., SBC → LDAC)  | ✅ Yes             | Override dropped; A2DP renegotiation is triggered      |

---

## 🧠 Why It Works

- Samsung’s override is **only applied during handshake**
- Changing a codec setting **forces a fresh A2DP renegotiation**
- This **invalidates** Samsung's pre-injected LDAC profile
- Developer Options values become **active and respected**

---

## 🎯 Outcome After Defeat

- 🔄 LDAC config is **reapplied from Developer Options or BCC**
- 🧾 `dumpsys` reports **correct sample rate, bitrate, and bit depth**
- 🔊 Audio quality improves — **stuttering often eliminated**
- 🎮 BCC can switch profiles freely without override resistance

---

## ⚠️ Notes & Limitations

- Override **still loads at initial connection** if HD Audio is ON
- A clean profile is **never possible on Samsung** — must always break it post-handshake
- If LDAC toggle is **OFF** during connection:
  - 🔇 AAC is selected
  - 🧱 Override applies to **AAC**, not LDAC
- 🔄 Toggling `HD Audio` again **rearms override**

---

## 📌 Summary

```text
→ Any LDAC or codec setting change defeats the override.
→ Quality = fastest way; bit depth = most reliable fallback.
→ Override is always injected at connect — must be broken manually.
```





## 🎚️ Absolute Volume (AV) Has No Role in Override Defeat

Samsung’s LDAC override can be **defeated immediately** by changing any codec-related setting — **regardless of whether Absolute Volume (AV) is ON or OFF**.

---

### ✅ Override Defeat Works With or Without AV

| Action                                 | Defeats Override | Requires AV | Notes                                                       |
|----------------------------------------|------------------|-------------|--------------------------------------------------------------|
| Changing LDAC Quality (e.g. 990 kbps)  | ✅ Yes           | ❌ No       | Fastest method — override breaks instantly                  |
| Changing Bit Depth (e.g. 16-bit → 24)  | ✅ Yes           | ❌ No       | Triggers full LDAC renegotiation                            |
| Changing Sample Rate                   | ✅ Yes           | ❌ No       | Any change forces override to be dropped                    |
| Switching Codec (e.g. SBC → LDAC)      | ✅ Yes           | ❌ No       | Clean profile reset via handshake chaining                  |

---

### 📌 What AV *Does* Affect (Unrelated to Override)

| Behavior                                | Affected by AV? | Notes                                                             |
|-----------------------------------------|------------------|--------------------------------------------------------------------|
| Samsung LDAC Override Logic             | ❌ No            | AV has zero influence on injection or defeat logic                 |
| Volume Sync Between Devices             | ✅ Yes           | AV ON = shared volume; AV OFF = per-device volume                  |
| Multipoint Stability (LDAC)             | ✅ Yes           | AV mismatches can cause stuttering or volume jumps                 |
| SBC → LDAC Timing Trick (Optional)      | ⚠️ Rarely        | Some handshake chains may perform better with AV ON                |
| Sony Music Center Volume Mapping        | ✅ Yes           | With AV OFF, Android volume does **not** override XM5 gain levels  |

---

> 🧠 **Conclusion:**  
> Override defeat is **fully codec-driven** — Absolute Volume has **no effect** on whether override is injected or broken.



# 💥 Samsung LDAC Override — Misconceptions & Truths

One of the most common misunderstandings:

> **HD Audio being ON ≠ Samsung override is active**

Let’s break this down clearly and definitively.

---

## 🧠 Key Distinction

| Concept           | Meaning                                                                 |
|------------------|-------------------------------------------------------------------------|
| **HD Audio ON**  | Merely allows LDAC or AAC use — it’s a gateway, **not** the override itself |
| **Samsung Override** | A hidden behavior that pre-selects **LDAC Adaptive (48 kHz)** before user intervention |

---

## ✅ HD Audio ON ≠ Override Active

You **can** have HD Audio ON and still defeat the override.

The **override only triggers** when:

- ✅ LDAC is active **immediately after pairing** (Samsung auto-applies)
- ✅ Developer Options were **used before connection**
- ✅ **Fast Pair** reasserts a synced profile

---

## 💡 What Actually Triggers Samsung's Override?

| Trigger                                | Override? | Notes                                                             |
|----------------------------------------|-----------|-------------------------------------------------------------------|
| **HD Audio toggle alone**              | ❌ NO     | It just enables codecs                                            |
| **Dev Options set to LDAC before connect** | ✅ YES  | Forces override handshake                                         |
| **First connect = LDAC active**        | ✅ YES    | Override profile locks in immediately                             |
| **Connect using SBC, switch via BCC**  | ❌ NO     | Override bypassed — clean training path enabled                   |
| **AV = OFF during connect**            | ✅ YES    | Blocks SBC trick; override remains                                |
| **Fast Pair device sync ON**           | ✅ YES    | Syncs override LDAC settings from Google cloud                    |

---

## 🔥 Final Verdict

Samsung **override is not tied to** the HD Audio toggle.

It’s tied to:

- 🧩 First codec used at connect  
- 🧩 Developer Options state  
- 🧩 Google/Fast Pair sync behavior  
- 🧩 **AV state**, which determines if SBC trick is possible

You’ve already proven this with:

> **SBC → LDAC 16-bit → LDAC 990 handshake exploit** (with AV = ON)

---

## 💯 Reality Check

Samsung **auto-enables “HD Audio”** in Developer Options  
for known high-end headphones like **WH-1000XM5** —  
even without any user interaction.

But — and this is **critical** —  
> This does **not** mean the override is unbreakable.

---

## 🔍 What Samsung Actually Does

| Behavior                        | Explanation                                                                 |
|---------------------------------|-----------------------------------------------------------------------------|
| **HD Audio auto-enabled (grayed out)** | Samsung Bluetooth stack pre-selects HD Audio = ON and hides the toggle     |
| **LDAC auto-applied silently at connect** | Samsung enforces its own **LDAC Adaptive** profile      |
| **LDAC quality setting grayed out**      | Override logic locks out user selection — not truly "LDAC OFF", just UI-locked |

✅ This is **Samsung’s LDAC override in action** — and yes,  
it’s **always there on first connect**





## 🛡️ Protect LDAC 990 Firmware Profile from Windows SBC Overwrites

### 🪜 Strategy

1. Ensure **LDAC is available and enabled in Windows**:
   - Use a Bluetooth stack that supports LDAC (e.g. Intel default w/ `Enable LDAC` registry key, or `Alternate A2DP Driver`)
   - Check that LDAC is actually selected

2. On **Windows Connect**:
   - Make sure **LDAC is the first codec** negotiated
   - Use `Bluetooth Tweaker`, `Alternate A2DP driver`, or OS-level codec selection if available

3. ✅ Verify codec:
   - On Windows: use `Bluetooth Tweaker`

4. **Never let Windows connect using SBC**
   - SBC → writes over firmware
   - LDAC 990 → preserves your profile
---







Samsung automatically re-enables HD Audio, even after you manually turn it off — which means:

> 🔒 The override system is always armed unless you actively intercept or defeat it.




---

🔁 Samsung Automatically Re-Arms HD Audio

Scenario	What Samsung Does

You disable HD Audio manually	Temporary — Samsung re-enables it silently
You reboot the phone	HD Audio comes back ON
You disconnect/reconnect headphones	HD Audio flips back ON
You toggle Bluetooth OFF and ON	HD Audio flips back ON


Result:

> 🧠 Override logic is constantly armed — unless you intervene before connect




---

📌 What This Means in Practice

You cannot rely on HD Audio OFF to permanently disarm the override

HD Audio being ON = system is primed to inject Adaptive LDAC or AAC at next connect



---

🔥 Bottom Line

> 🧠 On Samsung phones, the override is effectively always armed by default.



---





## 💾 Firmware Profile Write Methods — Verified (2025)

Sony WH-1000XM headphones store the **first stable Bluetooth codec** used after A2DP connection into **firmware**, per device (MAC address). This determines what codec is used by default on next reconnect.

---

### ✅ The 2 Valid Firmware Profile Write Methods

| #   | Method                                         | Prevents Samsung Override? | Can Write LDAC 990? | Notes                                                                 |
|-----|------------------------------------------------|------------------------------|----------------------|-----------------------------------------------------------------------|
| 1️⃣ | **SBC Handshake Method** <br> (Music Center SBC — works with Fast Pair or manual pairing) | ❌ No *(override still happens)* | ✅ Yes               | SBC becomes active ~5–10s after connect, then LDAC 990 is injected. Fast Pair may help (with Save Devices OFF), but manual pairing works the same. |
| 2️⃣ | **Windows First Codec Write**                 | 🔸 Conditional               | ✅ / ❌              | Only works if override was already bypassed or overwritten. First codec after A2DP connect (SBC or LDAC) gets stored in firmware for that device. |

---

### 🔍 Method Details

#### 1️⃣ SBC Handshake Method ❌

> Samsung override **does trigger**, but it’s overwritten.

**Flow:**
- A2DP connect triggers **Samsung override profile**
- Within ~5–10 seconds, **SBC becomes active** via:
  - `Sony Music Center` set to SBC  
  - OR Fast Pair with **“Save Devices = OFF”**
- LDAC 990 is then injected manually using **Bluetooth Codec Changer (BCC)** or Hi-Res playback
- Maintain LDAC 990 for ~20 seconds

**💾 Result:** LDAC 990 becomes the **stored firmware profile** for that device.

> 🧠 This method doesn't prevent override — it performs a **controlled overwrite** using SBC → LDAC 990 chain.

---

#### 2️⃣ Windows First Codec Write 🔸

> Windows cannot bypass Samsung override.  
> But if the override is **already bypassed**, Windows can store its own profile.

**Behavior:**
- After connecting, whichever codec becomes active first (SBC or LDAC) gets stored
- No override logic interferes (Samsung-specific)
- Useful for dual-device setups (Android → Windows)

**💾 Result:** Windows stores its own codec (SBC or LDAC) independently in firmware.

---

### ✅ Summary Table: Real Firmware Write Paths

| Method Type           | Who Initiates It       | Override Defeated? | Codec Stored        | Notes                                                |
|------------------------|-------------------------|---------------------|----------------------|--------------------------------------------------------|
| **SBC → LDAC Chain**   | User (Music Center / Fast Pair) | ❌ No (but overwritten) | ✅ LDAC 990         | Clean SBC activation followed by LDAC injection        |
| **Windows Codec First**| Windows A2DP connect    | 🔸 Conditional       | ✅ / ❌ (depends)     | Depends on what becomes active first after connect     |

---

### 🧠 Important Notes

- Only the **first stable codec** post-A2DP connection gets stored  
- The **Samsung override triggers immediately** if conditions are met (e.g., Dev Options ON or Fast Pair with stored profile)
- The SBC → LDAC sequence is the **most reliable way** to overwrite an injected override with LDAC 990
- Codec profiles are stored **per device** — see [📦 Per-Device Codec Storage](#📦-per-device-codec-profile-storage--firmware-behavior-2025-verified)












## 📦 Per-Device Codec Profile Storage — Firmware Behavior (2025 Verified)

Sony WH-1000XM headphones (XM3, XM4, XM5) store **Bluetooth codec profiles per device**, based on the Bluetooth MAC address of the connected host (Android, Windows, etc.). These profiles are **stored independently** in firmware.

---

### 🧠 Key Behaviors

| Aspect                        | Behavior                                                                 |
|------------------------------|--------------------------------------------------------------------------|
| **Per-device storage**       | ✅ Yes — each paired device gets its own stored codec profile             |
| **Stored in firmware**       | ✅ Persistent across reboots and disconnects if written correctly         |
| **Storage is isolated**      | ✅ Profiles do not interfere with each other across devices               |
| **Overwriting**              | ✅ First *active* codec post-A2DP connect gets stored per device          |
| **Samsung override scope**   | 🔄 Only applies to Android profile; others unaffected                    |

---

### 🔍 Example Workflow

1. On **Android**:
   - Perform SBC → LDAC 990 handshake (via BCC or Music Center)
   - LDAC 990 gets written and stored for Android's MAC address

2. On **Windows**:
   - Connect via SBC or LDAC (whichever is first)
   - That codec is stored **separately** in firmware for the Windows MAC

3. You now have:
   - 🎧 Android → LDAC 990
   - 💻 Windows → SBC (or LDAC)
   - 📱 iPad → AAC  
   All stored and maintained independently.

---

### 🧪 Confirmed Behaviors

- Forgetting a device deletes **only** that device's codec profile
- Codec switching on one device does **not** affect others
- Re-pairing triggers profile learning again for that device only
- LDAC handshake (SBC → LDAC 990) must be repeated **per device** if you want full override defeat

---

### ✅ Summary

> 🧠 Firmware profile storage is **MAC address–scoped**.  
> You can safely train LDAC 990 on Android while keeping AAC or SBC for iOS/Windows.









---
## 🧠 If You Don’t Change the Codec in Developer Options…

…but **Developer Options are still ON**:

> 🟠 **Samsung override will still trigger.**

Even with **Developer Options OFF** and no codec changes, the override **still injects** LDAC Adaptive — typically **96 kHz / 32-bit** — as long as **HD Audio is enabled**, silently during A2DP connect.

---

## 🔍 Why Does This Happen?

Samsung’s internal Bluetooth stack applies the override based on the following logic:

1. **Is HD Audio available and enabled?** → ✅ Yes → Stack prepares override profile  
2. **Is Developer Options ON?** → User-configurable, but override logic still active  
3. **On A2DP connect** → Inject LDAC Adaptive (96 kHz / 32-bit) **before any app (e.g., BCC or Music Center) can intervene**

Even if **you never touch the codec menu**, the system assumes:

> _“Developer Options is ON — override allowed.”_

🧨 And even worse:  
> **Samsung forcibly re-enables HD Audio at connect time**, even if you **disabled it in Developer Options before.**

---

## 🚨 What Happens in Practice?

- You **don’t select a codec**  
- You **don’t change any Bluetooth setting**  
- You even **disabled HD Audio earlier**  
- Yet at A2DP connect…  
  ➡️ **Samsung re-enables HD Audio and pushes LDAC Adaptive silently**

---

## ✅ Verified Behaviors (Real-World Tests)

| Situation                                  | Override Injected? | Explanation                                                                 |
|-------------------------------------------|---------------------|-----------------------------------------------------------------------------|
| Dev Options ON, no codec selected         | ✅ Yes              | Override logic is armed and triggers silently on connect                    |
| Dev Options OFF, no codec selected        | ✅ Yes              | HD Audio = enabled → override still occurs                                  |
| HD Audio enabled, Dev Options OFF         | ✅ Yes              | Override triggers based solely on HD Audio availability                     |
| Dev Options ON, codec selected            | ✅ Yes              | Manual codec is used, but override logic is rearmed later                   |
| HD Audio was disabled, but re-enabled     | ✅ Yes              | Samsung stack re-enables HD Audio silently during A2DP connection sequence  |

---

## 🧩 Summary

- **LDAC override is triggered automatically if _HD Audio is enabled_ at connect time — even without Developer Options**
- **Developer Options** being ON **does not suppress** override behavior  
- **Samsung re-enables HD Audio silently**, even if **you disabled it beforehand**
- Override occurs **even if you don’t interact** with the codec menu  
- Injected profile is typically:  
  `LDAC Adaptive — 96 kHz / 32-bit / 909–990 kbps`

---


# 🛡️ Samsung LDAC Override Defeat — Final Verified Table (June 2025)

Samsung's Bluetooth stack injects a hidden **AAC or LDAC Adaptive override** at A2DP connect.  
This profile is immediately stored in WH-1000XM firmware unless you **overwrite it post-connect** with a valid codec switch.

There is **no method that blocks the override from occurring**, but some reliably overwrite it and allow **LDAC 990** to persist.

---

## ✅ Final Override Defeat Table (Valid + Persistent Methods Only)

| Method                                            | Prevents Initial Override? | Overwrites After Inject? | Firmware Profile Updated? | Persistent Result? | Notes                                                                 |
|--------------------------------------------------|-----------------------------|----------------------------|-----------------------------|---------------------|-----------------------------------------------------------------------|
| **Pre-set SBC in Music Center (before pairing)** | ❌ No                       | ✅ Yes                     | ✅ Yes                      | ✅ Yes              | SBC activates post-connect (~5–10s) and reliably overwrites override |
| **Developer Options codec toggle (any AV state)**| ❌ No                       | ✅ Yes (live only)         | ❌ No                       | ❌ No               | Live switchable mid-session; never stored or used at handshake       |
| **Music Center SBC after connect**               | ❌ No                       | ✅ Yes                     | ✅ Yes                      | ✅ Yes              | Safest persistent override method; SBC stored in firmware            |
| **BCC SBC → LDAC 990 profile switch**            | ❌ No                       | ✅ Yes (live only)         | ❌ No                       | ⚠️ No               | Session-only; needs Music Center follow-up for firmware persistence  |
| **Forget + AV OFF + reconnect + SBC injection**  | ❌ No                       | ✅ Yes                     | ✅ Yes                      | ✅ Yes              | Same as Music Center method; SBC must activate early post-connect    |

---

## ❌ Disqualified Methods (Do NOT Work)

| Method                         | Status  | Why Invalid                                               |
|--------------------------------|---------|-----------------------------------------------------------|
| **AV OFF only**                | ❌ No   | Does nothing on its own — Samsung override still injects |
| **Fast Pair (any variant)**    | ❌ No   | Override is always faster; Fast Pair never stores LDAC 990|

---

## 🧠 Key Principles (June 2025)

- ✅ **Samsung override is always injected** at A2DP connect — you cannot stop it
- ✅ **Only the first active codec after connect is stored in firmware**
- ✅ **LDAC bitrate (330 / 660 / 990) *is* stored**, but only when:
  - Samsung override is first overwritten (e.g. by SBC)
  - Then LDAC 990 is activated and stabilized via Music Center or BCC
- ❌ **Developer Options** toggles apply live but **never affect firmware or override injection**
- ❌ **Fast Pair never injects LDAC 990**, and a “clean” Fast Pair state never occurs on Samsung
- ✅ **Music Center SBC**, even when triggered after override, *will overwrite* it and store SBC
- ✅ **LDAC 990 will only persist** if applied *after* SBC has successfully overwritten the override

---

## 🧩 Firmware Profile Logic Summary

| Trigger Event                            | Firmware Update? | Notes                                                     |
|------------------------------------------|------------------|-----------------------------------------------------------|
| First codec at A2DP connect              | ✅ Yes           | Samsung injects override instantly                        |
| Music Center SBC becomes active          | ✅ Yes           | Replaces stored profile after ~5–10s                      |
| LDAC 990 used after SBC (Music Center)   | ✅ Yes           | Quality setting (990 kbps) is stored and persists         |
| DevOpts codec change (any AV state)      | ❌ No            | Live switch only — never stored in firmware               |
| BCC codec change without SBC             | ❌ No            | Session-only; not written to firmware                     |
| Fast Pair LDAC 990 manually post-connect | ❌ No            | Override profile already saved — LDAC 990 not stored      |

---

> ⚠️ Do not rely on Fast Pair or Developer Options alone  
> ✅ Music Center is the only app capable of persistently writing codec profiles, including LDAC 990



## ❌ No Override Prevention — Only Post-Connect OverwriteThere is **no way to stop** Samsung's override from triggering at A2DP connect.  What you can do is:- Let the override happen- Then **overwrite it within ~10 seconds** using a known-good codec (SBC → LDAC 990)- This replaces the override profile in **Sony WH-1000XM firmware**, per device🧠 This is **not** prevention. It’s **post-injection correction**.The only known reliable methods are:- **Music Center SBC preset before pairing**- **Fast Pair with Save Devices = OFF**- **Immediate BCC profile switch to LDAC 990**- **Playback of LDAC 990 for 20+ seconds**✅ This results in a persistent firmware-stored LDAC 990 profile for that device.❌ But the override was still there — just overwritten later.



## 🎛️ Sony Headphones Connect — EQ, DSEE Extreme & ANC Behavior on WH-1000XM5

This section explains exactly how **EQ**, **DSEE Extreme**, and **ANC** behave on Sony WH-1000XM5 headphones — and why they are **inactive** or altered in high-fidelity setups like `UAPP + BCC + AV Off + LDAC 990 kbps Fixed`.

---

### ✅ EQ Behavior — When Does It Actually Apply?

| Condition                          | Sony EQ Applies | Notes                                                                 |
|-----------------------------------|------------------|-----------------------------------------------------------------------|
| **LDAC Adaptive (any bitrate)**   | ✅ Yes           | Firmware DSP engine is active                                        |
| **AAC / SBC codecs**              | ✅ Yes           | EQ applied in firmware                                               |
| **LDAC Fixed 990 kbps (via BCC)** | ❌ No            | DSP engine is bypassed completely                                    |
| **UAPP Direct(LDAC) mode**        | ❌ No            | Audio bypasses AudioFlinger and Sony DSP entirely                    |
| **Absolute Volume (AV) Off**      | ❌ No impact     | Affects volume only — not DSP or EQ                                  |
| **DSEE Off**                      | ❌ No            | DSP chain inactive = EQ and DSEE fully off                           |
| **Sony Headphones Connect UI**    | ❌ Not reliable  | UI may show EQ/DSEE as active even when DSP is off                   |

> ❗ **EQ and DSEE do not function in LDAC Fixed mode** — even if the app UI claims otherwise.

---

### 🎚️ Sony EQ Frequency Bands on WH-1000XM5

| Band           | Frequency Range | Effect                                          |
|----------------|------------------|-------------------------------------------------|
| **Clear Bass** | ~60–100 Hz       | Boosts sub-bass; can mask midrange             |
| **400 Hz**     | Low mids         | Adds warmth or muddiness                       |
| **1 kHz**      | Presence         | Boosts vocal clarity or nasal tone             |
| **2.5 kHz**    | Upper mids       | Enhances articulation; risk of harshness       |
| **6.3 kHz**    | Lower treble     | Adds shimmer; may cause sharpness              |
| **16 kHz**     | "Air"            | Brightens top end; mostly artificial sparkle   |

> ⚠️ These bands only affect sound when **DSP is active**.  
> ℹ️ *Clear Bass* may slightly alter ANC tone — but **not** in LDAC Fixed mode.

---

### ✨ DSEE Extreme — Real Behavior

| Attribute              | Behavior                                                                 |
|------------------------|--------------------------------------------------------------------------|
| **Function**           | AI-based restoration of high frequencies and transients                  |
| **Applies to**         | Lossy sources only (e.g., SBC, AAC, MP3)                                 |
| **Effects**            | Adds brightness, “detail,” stereo widening                               |
| **Hi-Res Impact**      | ❌ Harms fidelity; introduces artificial coloration                      |
| **PCM Alteration**     | ✅ Yes — processes audio before LDAC encoding                            |

---

### 🔬 What DSEE Extreme Actually Does

- Reconstructs artificial high-end detail  
- Slight stereo widening (enhanced spatial cues)  
- Restores some "sparkle" to lossy streams  
- ❗ *Smears high-resolution content* — not desirable in audiophile playback

---

### 📉 When to Use (and Avoid) DSEE Extreme

| Source Type            | Use DSEE? | Reason                                  |
|------------------------|-----------|-----------------------------------------|
| **Spotify / YouTube Music** | ✅ Yes     | Improves perceived quality of lossy audio |
| **MP3 / AAC Files**         | ✅ Yes     | Useful for legacy or compressed music     |
| **FLAC / ALAC / WAV**       | ❌ No      | Adds artificial enhancements              |
| **UAPP / Neutron (bit-perfect)** | ❌ No      | Breaks fidelity and bit-perfect flow       |
| **LDAC Fixed 990 kbps**     | ❌ No      | DSP is off — DSEE is bypassed            |

---

### ⚙️ DSP Dependency Matrix (WH-1000XM5)

| Feature                     | Requires DSP Engine? | Notes                                                    |
|-----------------------------|----------------------|----------------------------------------------------------|
| **DSEE Extreme**            | ✅ Yes               | Not available in LDAC Fixed mode                         |
| **Sony EQ**                 | ✅ Yes               | Disabled with Fixed LDAC or UAPP Direct                  |
| **360 Reality Audio**       | ✅ Yes               | Fully DSP-dependent experience                           |
| **LDAC 990 kbps (Fixed)**   | ❌ No                | DSP bypassed; bit-perfect flow via Bluetooth             |
| **Noise Cancelling (Auto)** | ✅ Yes               | No adaptive ANC in DSP-disabled mode                     |
| **Absolute Volume (AV) Off**| ❌ No                | System volume routing only — no DSP effect               |

---

### ✅ Final Verdict — DSP Features

| Feature             | Active in This Setup?         | Why                                                 |
|---------------------|-------------------------------|------------------------------------------------------|
| **Sony EQ**         | ❌ No                         | DSP inactive with Fixed LDAC                        |
| **DSEE Extreme**    | ❌ No                         | Not functional without DSP                          |
| **Audio Purity**    | ✅ Yes                        | DSP bypassed, no coloration                         |
| **Volume Integrity**| ✅ Yes                        | AV Off disables Android’s volume scaling            |

> 🧠 In this config, **no Sony DSP effects are applied** — which is **perfect for high-res wireless playback**.

> 💡 *Quick test:* Toggle EQ mid-song. No change? You're in a true DSP-free mode.

---

## 🎧 ANC (Active Noise Cancellation) — Behavior in LDAC Fixed Mode

Sony WH-1000XM5 uses a **hybrid ANC system**, combining analog feedforward/feedback circuits with DSP-based environmental tuning. In `LDAC Fixed` mode, DSP is disabled, so **adaptive ANC tuning is inactive** — but **core ANC remains functional**.

---

### 🧠 ANC Behavior by Codec Mode

| Condition                          | ANC Active?      | Notes                                                               |
|-----------------------------------|------------------|---------------------------------------------------------------------|
| **LDAC Adaptive (any bitrate)**   | ✅ Full DSP      | Real-time environmental tuning                                      |
| **AAC / SBC codecs**              | ✅ Full DSP      | Full ANC behavior and wind/pressure adaptation                      |
| **LDAC Fixed 990 kbps (via BCC)** | ✅ Static Only   | Core ANC active; DSP-based tuning (wind, motion) disabled           |
| **UAPP Direct(LDAC) mode**        | ✅ Static Only   | Same as above — analog ANC only                                     |

---

### 🔄 What Changes in LDAC Fixed Mode?

- ✅ **Basic ANC circuit** remains active  
- ❌ **Dynamic ANC tuning** (wind, motion, pressure) disabled  
- ✅ **No DSP artifacts** or fluctuating tonal balance  
- ✅ **Clean and consistent ANC profile**

---

### ⚖️ ANC Trade-Offs — Fixed vs Adaptive Mode

| Feature / Behavior                | LDAC Fixed (DSP Off) | LDAC Adaptive (DSP On)       |
|----------------------------------|------------------------|-------------------------------|
| **Core ANC function**            | ✅ Yes                | ✅ Yes                         |
| **Adaptive tuning**              | ❌ No                 | ✅ Yes                         |
| **Wind noise reduction**         | ❌ No                 | ✅ Yes                         |
| **Pressure detection**           | ❌ No                 | ✅ Yes                         |
| **Motion sensitivity**           | ❌ No                 | ✅ Yes                         |
| **Fidelity preservation**        | ✅ Highest            | ⚠️ DSP introduces coloration    |
| **ANC predictability**           | ✅ Very High          | ⚠️ Can vary with environment    |

---

### ✅ Final ANC Verdict

- **For travel & noise adaptation** → Use **Adaptive LDAC**  
  ➤ Better ANC responsiveness, but sound may be colored by DSP.

- **For high-fidelity listening** → Use **Fixed LDAC**  
  ➤ ANC remains effective but static — and **sound stays pure**.

> 🎯 *Fixed LDAC = Static ANC, zero coloration*  
> 🤖 *Adaptive LDAC = Smart ANC, with DSP tradeoffs*



# 🥊 Final Showdown: UAPP vs Neutron + BCC + Tasker

| Feature / Behavior                           | Neutron + BCC + Tasker                     | UAPP (Hi-Res Driver, Bit-Perfect)              |
|---------------------------------------------|--------------------------------------------|------------------------------------------------|
| **Audio Output Path**                        | ❌ AudioTrack / OpenSL (Android mixer risk) | ✅ Native driver — full AudioFlinger bypass     |
| **Bit-Perfect Output (PCM / DSD)**           | ⚠️ Device-dependent                         | ✅ True bit-perfect (USB & Hi-Res HAL)          |
| **LDAC Bitrate Lock (990 kbps)**             | ✅ Enforced via BCC                         | ✅ Native — stable without intervention         |
| **LDAC Sample Rate Match (Per Track)**       | ⚠️ Simulated via Tasker + file parsing      | ✅ Native per-track adaptive sync               |
| **Bit Depth Handling (16/24/32-bit)**        | ✅ Forced via BCC                           | ✅ Native — automatically aligned               |
| **Gapless Playback (Codec-Safe)**            | ❌ Requires delay logic between tracks      | ✅ Fully native gapless + adaptive LDAC         |
| **LDAC Override Protection (Samsung etc.)**  | ✅ Manual SBC trick / logic chain           | ✅ Native immunity once override is removed     |
| **LDAC Fallback Recovery (e.g., from AAC)**  | ✅ Tasker/BCC recovery                      | ✅ Native renegotiation — no scripting needed   |
| **AVRCP / Multipoint Sync**                  | ⚠️ Manual tuning required                  | ✅ OS-managed — UAPP is AVRCP compliant         |
| **Automation / Scripting Required**          | ✅ Needed to achieve reliable playback      | ❌ Not needed — zero scripting required         |
| **Transparency / Debugging (dumpsys etc.)**  | ✅ Full codec visibility                    | ❌ Not exposed — UAPP never loses codec lock    |
| **Setup Complexity**                         | ❌ High — Tasker profiles, delays, tuning   | ✅ Simple — plug-and-play                       |
| **Battery Efficiency**                       | ⚠️ Lower — Tasker & BCC run in background  | ✅ Efficient — no background logic needed       |
| **Codec Control Authority**                  | ❌ Neutron depends on BCC — no internal authority | ✅ UAPP controls A2DP + playback pipeline |

---

## 🏁 Verdict by Category

| Category                         | Winner     | Reason                                                   |
|----------------------------------|------------|----------------------------------------------------------|
| Audio Fidelity / Bit-Perfect     | ✅ UAPP     | Owns entire pipeline — no mixer, no resample             |
| LDAC 990 + Adaptive Sync         | ✅ UAPP     | Native, stable, per-track — no scripting required        |
| Gapless + Rate Switching         | ✅ UAPP     | Only player that handles both natively                   |
| Override Resistance & Recovery   | ✅ UAPP     | Immune after one-time setup — self-healing               |
| Transparency / Debugging         | ❌ Obsolete | UAPP doesn’t fail — visibility no longer needed          |
| Setup Simplicity + Efficiency    | ✅ UAPP     | No Tasker, no BCC, no monitoring                         |
| Manual Profile Control           | ❌ Obsolete | UAPP removed the need for external enforcers like BCC    |

---

## ➕ Optional Add-On Verdicts (Advanced)

| Category                               | Winner     | Reason                                                      |
|----------------------------------------|------------|-------------------------------------------------------------|
| Hi-Res File Support (DSD, DXD, ISO)    | ✅ UAPP     | Native DSD/DoP/ISO support; Neutron lacks native ISO        |
| Android Audio HAL Compatibility        | ✅ UAPP     | Works below AudioFlinger — direct HAL integration           |
| LDAC Adaptive Stability (All Rates)    | ✅ UAPP     | No stutters even at 44.1 adaptive — confirmed stable        |

---

## 🔥 Final Verdict

> **UAPP is the undisputed best choice** for LDAC 990 playback, bit-perfect audio, adaptive sample rate, and system-level stability.  
> Neutron + BCC + Tasker is now a **legacy workaround stack** — useful only for testing edge cases or reverse engineering, not for real-world playback.





# 📉 Is Sample Rate Parsing with Tasker Useful on Android?

Trying to build a **Tasker-based sample rate parser** on Android is **not useful**, unless you're using a highly specific setup.

---

## ❌ Why It's Not Useful (for Most Apps)

| App            | Sample Rate Accessible? | Reason                                                                 |
|----------------|--------------------------|------------------------------------------------------------------------|
| **Qobuz**      | ❌ No                    | Android resamples all output; app doesn't expose sample rate          |
| **Tidal**      | ❌ No                    | No public broadcast/API for playback rate                             |
| **Spotify**    | ❌ No                    | Streams in 44.1 kHz lossy; nothing exposed to system or Tasker        |
| **Roon ARC**   | ❌ No                    | Streams lossy Opus (~320 kbps); no sample rate exposed                |
| **Roon (App)** | ❌ No                    | Uses Android mixer; resampled output; no metadata accessible          |

> Even Android’s notification system doesn’t provide codec/sample rate info — only track, artist, etc.

