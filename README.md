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


## LDAC Codec Negotiation & Profile Generation — Finalized Edition

> Everything that determines which codec (SBC, LDAC 330/660/990) gets selected during Bluetooth connection.  
> This list is 100% focused on **connection-time behaviors** — not post-connection bitrate changes or audio stability.

---

### 🎧 Headphone & Device Factors

- **Power cycling headphones**  
  → Clears stored codec profile in the headphone’s memory.  
  → Allows a new profile (e.g., LDAC 990) to be stored on next clean connection.

- **Multipoint pairing active**

- **AVRCP version mismatch**  
  → May block proper Absolute Volume detection.  
  → Can disrupt handshake logic or GUI sync.

- **Absolute Volume ON vs OFF**  
  - **AV ON**: Android controls headphone volume directly. Can block SBC → LDAC profile switching.  
  - **AV OFF**: Required for proper manual profile chaining, BCC override, and stored profile training.  
    → Disables Android volume sync interference, enabling clean codec negotiation.

---

### 📱 Phone Settings That Affect Codec Negotiation

- **LDAC toggle in Developer Options**  
  → Activates Samsung’s LDAC override stack.  
  → Must be followed by SBC reset and Developer Options OFF to stop override.

- **Developer Options open during connection**  
  → Re-applies override logic immediately if LDAC is selected.  
  → Avoid opening Dev Options during or right before pairing.

- **HD Audio toggle in Bluetooth device settings**  
  → Triggers full codec renegotiation.  
  → May allow or re-trigger override stack.  
  → Controls whether LDAC is allowed at all in A2DP profile exchange (independent of Developer Options LDAC toggle).

- **Nearby Devices permission (Music Center, GMS)**  
  → Enables silent override via GATT.  
  → Reapplies stored codec profiles without user interaction.  
  → Must be revoked or app force-stopped to disable.

- **Connection method: Quick Settings vs Power-On**  
  → Reconnecting via **Quick Settings** toggle: more likely to honor stored (trained) profile.  
  → Reconnecting via **powering on headphones**: often re-triggers Samsung override.

- **Disabling Developer Options while disconnected**  
  → Leaves override state intact — no reset occurs.

- **Disabling Developer Options while connected**  
  → Clears override state immediately, allowing your codec profile to apply.

---

### ⚙ System Stack Behavior & Profile Storage

- **Samsung LDAC override stack**  
  → Automatically activates if LDAC is used in Developer Options.  
  → Always forces Samsung’s preferred LDAC mode unless bypassed.  
  → On Samsung, override stack can independently toggle HD Audio ON after pairing.  
  → However, override stack **cannot toggle LDAC codec toggle directly** — only HD Audio layer.

- **AAC default fallback post-pairing (Samsung stack)**  
  → After first pairing, AAC is no longer neutral.  
  → Becomes part of Samsung’s override chain if LDAC isn't fully locked.  
  → Even with AV OFF, Samsung may initially negotiate AAC, then force LDAC override within seconds unless SBC profile chaining blocks it.

- **Absolute Volume status**  
  - **AV ON**: Volume sync events can re-trigger override or block codec switching.  
  - **AV OFF**: Required for successful intermediate profile chaining and GUI desync repair.  
    → Prevents Android-side volume control from interfering with profile logic.

- **Fast Pair timing**  
  → Determines which profile wins: Samsung override or user-defined profile.  
  → Override usually applies within 1–2 seconds unless interrupted by SBC chaining.

- **Intermediate profile chaining**  
  → Example: SBC → LDAC 16-bit → LDAC 24-bit 990  
  → Bypasses override stack when done early and with AV OFF.  
  → Essential to force LDAC 990 without triggering Samsung override.

- **Waiting 10+ seconds post-handshake (no override)**  
  → Locks negotiated profile into headset firmware (WH-1000XM5/XM3).  
  → Overrides won’t reapply unless retriggered.

- **GUI desync between Developer Options and BCC**  
  → Happens if override or stack race condition occurs.  
  → Solved by double-applying the BCC profile and using AV OFF.

- **Codec override persists across reboots**  
  → Only cleared via SBC handshake followed by Developer Options OFF during active connection.

---

### 📲 App Behavior That Influences Codec Negotiation

- **Sony | Music Center**  
  → With Nearby Devices permission: silently re-applies LDAC profile at connection.  
  → Override happens even if you only changed volume.  
  → Must be force-stopped or stripped of permission to prevent interference.

- **Bluetooth Codec Changer (BCC)**  
  → Defeats Samsung override using profile chaining:  
    - SBC → LDAC 16-bit → LDAC 24-bit 990  
  → Must apply within 1–2 seconds of connection.  
  → Double-apply profile to fix GUI mismatch.

- **USB Audio Player PRO (UAPP)**  
  → May re-trigger codec negotiation at playback start.  
  → Can override or conflict with BCC if launched too early.  
  → Best practice: allow BCC to finish first, then launch UAPP.

- **Google Play Services (GMS)**  
  → With Nearby Devices permission: silently applies stored override.  
  → Often triggered during Fast Pair.  
  → Disable permission to stop this.

- **Tasker (Bluetooth connect triggers)**  
  → Can switch to SBC or intermediate LDAC profiles instantly at connect.  
  → Must run before override logic executes (within ~1–2s).  
  → Used to automate profile chaining for override bypass.

- **“Automatically save devices” in Fast Pair**  
  → If enabled, GMS syncs override profiles to the cloud.  
  → Reapplies LDAC override silently after reset or on new device.  
  → Must be turned OFF to prevent Samsung override returning.

---

## 🧬 Samsung Codec Behavior — Override Negotiation Logic

Samsung's codec handling is **never neutral** after pairing — all codec transitions participate in its override system.

---

### ✅ AAC Override Logic (When LDAC Is Disabled)

- If **LDAC is disabled globally** (HD Audio toggle OFF):
  - Samsung will forcibly activate **AAC** whenever possible.
  - This occurs regardless of **Absolute Volume ON or OFF**.
- AAC becomes the "preferred high-quality codec" under Samsung's stack if LDAC is not allowed.
- AAC selection here is not user-driven — it is part of Samsung’s override logic asserting itself even in absence of LDAC.
- Disabling LDAC does not fall back to SBC by default; AAC is treated as the next-in-line override.
- **Absolute Volume state has no effect on AAC override behavior**:  
  → AV OFF does not prevent AAC default activation when LDAC is disabled.

---

### ✅ SBC Is Never Neutral — Samsung Will Never Default to SBC

- Samsung’s pairing and reconnect behavior **never defaults to SBC** after initial pairing.
- SBC only appears under very specific conditions:
  - Explicit SBC selection in Developer Options.
  - Manual override defeat workflows (SBC handshake exploit).
  - Complete device capability fallback (rare edge case if all high-quality codecs fail).
- Even during poor RF conditions or reconnections, Samsung prefers Adaptive LDAC degradation over SBC fallback.
- SBC is therefore not a "graceful fallback" in Samsung's override stack — it is an explicit override defeat tool.

---

### ✅ AAC ≠ Neutral on Reconnect — It’s Part of Override Negotiation

- AAC is not a fallback or neutral reconnect state.
- On reconnect:
  - Samsung may briefly show **AAC** for several seconds.
  - The override system then auto-transitions to **LDAC Adaptive** unless override defeat logic is actively blocking it.
  - AAC is simply a transient override phase in Samsung's handshake logic.
- This is why AAC sometimes appears momentarily before BCC or SBC handshake defeats fully assert.

---

### 🔧 Samsung Override Control Priority

| Priority | Control Layer |
|----------|----------------|
| 1️⃣ | HD Audio Toggle (global switch Samsung controls) |
| 2️⃣ | Override stack injects AAC or LDAC depending on allowed codec set |
| 3️⃣ | Developer Options LDAC settings mostly ignored unless override is fully defeated |

---

### 🔬 Summary Table

| Scenario | Codec | Override Behavior |
|----------|-------|--------------------|
| LDAC Disabled | AAC | Forced by Samsung (independent of AV state) |
| LDAC Enabled (Override Active) | AAC → LDAC Adaptive | Sequential override progression |
| Override Defeated (BCC/Tasker) | Anything | Full manual handshake control |








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


