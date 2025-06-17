#  Settings That Dont Interfere with LDAC 990kbps
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








##  Settings That Interfere with LDAC 990kbps

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

    Disable **Nearby device scanning** under:  
   - `Settings > Google > Devices & Sharing > Devices > Scan for nearby devices`  
     → **Turn this OFF**

    Disable **Saved Devices auto-sync** under:  
   - `Settings > Google > Devices & Sharing > Saved Devices`  
     → Tap ** (3-dot menu)** and select **“Turn off Automatically Save Devices”**

>  **Explanation:**  
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

13. **Spotify Nearby Devices Permission**  
    - Disable Spotify’s **Nearby Devices** permission under:  
      `Settings > Apps > Spotify > Permissions > Nearby Devices → Deny`  
    - Explanation:  
      - Spotify periodically performs **background device discovery** to detect Cast devices, Spotify Connect speakers, and other targets.
      - These discovery scans occur over BLE and 2.4GHz, which directly interfere with **LDAC 990kbps transmission stability**.
      - Disabling this permission minimizes background scanning load during LDAC playback.

---


##  Settings That help with LDAC 990kbps
Change scan interval is set to rarely in  connectivity labs 
filter option is set to show less in  connectivity labs



# 🔬 Adaptive LDAC Stability Tiers — Fully Validated Stability Model

Through full empirical testing across Samsung Android devices, WH-1000XM firmware, and Bluetooth Codec Changer (BCC), the following stability tiers have been universally verified for Adaptive LDAC playback.

---

## 🧠 Governing Stability Principle

> Adaptive LDAC stability depends primarily on sample rate, independent of bitrate or bit depth.

- Bitrate scaling (330 / 660 / 990 kbps) under Adaptive has no meaningful impact on stability.
- Bit depth (16-bit vs 24-bit) has only minor secondary influence.
- Sample rate fully governs adaptive link stability behavior.

---

## 🔬 Adaptive Sample Rate Stability Table

| Sample Rate | Stability Class | Comment |
|--------------|------------------|---------|
| 48.0 kHz     | ✅ Fully Stable  | Universally stable across devices |
| 96.0 kHz     | ✅ Fully Stable  | Stable with full link margin |
| 88.2 kHz     | ⚠ Partially Stable | Minor stability variance observed |
| 44.1 kHz     | ❌ Unstable      | Frequent bitrate drops and link renegotiations |

---

## 🔬 Summary Rule Statement

- ✅ Adaptive 48kHz and 96kHz operate with full long-term stability across all tested conditions.
- ⚠ Adaptive 88.2kHz may work but is not recommended for critical listening.
- ❌ Adaptive 44.1kHz cannot maintain full bitrate stability — frequent negotiation dropouts observed.
- ✅ Bit depth (16/24-bit) is not the cause of adaptive instability — sample rate governs link behavior.

---

## 🔧 Best Practice for Adaptive Mode

| Scenario | Recommendation |
|----------|-----------------|
| Adaptive Streaming | Prefer 48kHz or 96kHz |
| 44.1kHz Content (CD Quality) | Use Fixed LDAC Mode at 44.1kHz |
| High-Res Content (>= 48kHz) | Adaptive fully viable |
| Critical Listening Stability | Use Fixed Mode for maximum link reliability |

---

✅ **This model has been fully validated across multiple firmware generations, devices, Samsung Bluetooth stacks, and both WH-1000XM3 and WH-1000XM5 models.**



## 🎯 LDAC 909/990 Semi-GUI Verification Method

Once the desired LDAC profile has been correctly applied (via BCC, Music Center, or UAPP), full bitrate verification can be performed as follows:

### 🔧 Verification Procedure

1. Start playback of actual 44.1 kHz content.
2. Open one of:
    - **Bluetooth Settings (full system menu)**
    - **WiFi Settings (full system menu)**
    - **Bluetooth Quick Settings panel**
3. While playback is active, observe playback behavior:
    - ✅ If **stutters occur**, 909 or 990 kbps is fully active.
    - ❌ If **stable under heavy conditions**, likely fallback bitrate active.

### 🚫 Do not use:
- **WiFi Quick Settings tile** (wont generate stutter).

---

> ⚠ This method leverages the fact that certain system settings panels briefly interact with Bluetooth stack refresh timing, making existing high-bitrate LDAC states slightly more sensitive to RF congestion during active playback.

##  Troubleshooting Tricks I Used
These low-level techniques helped uncover hidden sources of LDAC interference.
---
###  Inspect Permission Usage to Detect LDAC Interference
You can identify apps or system components that silently interfere with Bluetooth, Nearby Devices, or scanning features by checking **recent permission usage logs**.
####  Step-by-Step
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
###  What to Watch Out For
| Permission        | Unexpected Offenders                    | Action to Take                         |
|-------------------|------------------------------------------|----------------------------------------|
| **Nearby Devices**| Google Play Services, Assistant          | Deny permission or use ADB `appops`    |
| **Bluetooth**     | Music Center, Galaxy Wearable, GMS       | Force-stop or uninstall                |
| **Location**      | SmartThings, Zepp, Health tracking apps  | Disable or deny permission             |
---
 *This method reveals hidden reconnections, override attempts, or scanning triggers — even after toggles have been turned off.*
>  Combine this technique with `dumpsys bluetooth_manager` or ADB log monitoring for full visibility.


#### Google Play Services Interference Timing

You do **not** need to disable Nearby Devices or revoke Find My Device permissions immediately.

- During initial setup (SBC handshake, profile injection, LDAC training), Google’s override services do not interfere — as long as:
  - Developer Options are fully cleared.
  - Sony Music Center is inactive.

- Only after the LDAC 990 profile is confirmed and stored in firmware should you disable:
  - Nearby Devices permission for `com.google.android.gms` (via ADB or system settings)
  - Assistant and Find My Device background access (optional)

✅ This allows full device tracking, Find My Device, and Assistant functionality to remain active during the initial training phase.
