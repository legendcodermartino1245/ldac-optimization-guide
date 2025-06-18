![image](https://github.com/user-attachments/assets/c078c8eb-bb69-45ec-9707-dbdc5d90ed34)# LDAC Done Right
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
###  LDAC Configuration Matrix Fixed

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


###  LDAC Configuration Matrix Adaptive
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






## 🧠 Samsung Override Is Always Armed Behind HD Audio

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

- 🎭 Use **profile switch tricks** (like SBC → LDAC 16-bit → LDAC 990kbps) to bypass override logic.
- 🤖 Use **Bluetooth Codec Changer (BCC)** to automate negotiation **before override is injected**.






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
## 🎚️ Sound Connect EQ Has No Impact on LDAC Quality (WH-1000XM5)

When using the **Sony WH-1000XM5** with LDAC active, enabling or disabling **EQ in the Sound Connect app** does **not** affect LDAC quality, bitrate, or codec parameters.

### ✅ Confirmed Behavior:
- LDAC **remains active** at the **same bitrate and sample rate** regardless of EQ state.
- The **Developer Options “LDAC Audio Quality”** setting stays unchanged (e.g. 990 kbps) when EQ is toggled.
- No codec renegotiation or fallback (e.g. to AAC) occurs when EQ is applied.

### 🧠 Why It Matters:
- On older Sony headphones (e.g. WH-1000XM3), enabling EQ **can** lower the effective LDAC bitrate or force fallback to AAC.
- On WH-1000XM5, **LDAC codec quality is preserved** even with audio processing features like EQ enabled — as long as override conditions aren't triggered (e.g. Samsung LDAC override or HD Audio toggling).

### 📌 Tip:
- You can safely use **EQ in Sound Connect** on XM5 **without compromising LDAC 990 kbps** or causing bitrate drops — unless **other system-level override conditions** interfere.





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



