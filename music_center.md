>  **Codec Behavior Note**  
> The **Sony | Music Center** app can only switch between:
>
> - **SBC**
> - **LDAC 96 kHz 32-bit Adaptive**
> - **LDAC 96 kHz 32-bit 909/990 kbps (Fixed)**
>
> Once LDAC is locked by Music Center, **external tools like Bluetooth Codec Changer (BCC) and USB Audio Player PRO (UAPP) cannot override it** unless the codec is renegotiated via disconnection or an SBC handshake.
>
> Opening Music Center **after a profile has been set by another app** will immediately trigger a renegotiation to the **currently selected LDAC mode inside the app**, overriding any prior configuration—even if BCC or UAPP had previously succeeded.


>  **Default Behavior:**  
> If you install and open Sony Music Center **without changing any codec settings**, it defaults to:
>
> - **LDAC Adaptive** mode  
> - **96 kHz** sample rate  
> - **Bitrate** dynamically adjusts between **330 / 660 / 990 kbps**, depending on link quality
>
>  This means Music Center does **not** apply a fixed bitrate by default — it applies a **96 kHz Adaptive profile**, and lets LDAC decide between 330, 660, or 990 kbps in real time.

##  Additional Notes on Codec Storage and LDAC Behavior

- **LDAC quality settings written by Music Center are not applied immediately.**  
  The selected codec (e.g., 990 kbps) is stored in the headset firmware, but it only takes effect **on the next Bluetooth connection**.  
  Even then, due to the LDAC bug, the codec may **appear correct in dumpsys or the GUI** but still stream at the wrong quality.  
   Always verify actual playback quality — never trust visuals alone.

        




 is needed to store a codec profile.**  
  After applying SBC or a 16-bit LDAC intermediate profile (via Music Center or BCC), you can **power off the headphones immediately**.  
  The setting is written to firmware instantly — no need to wait 10+ seconds.  
   This enables faster and more reliable handshake training for override bypass strategies.

###  LDAC Priority Setting Impact

>  Music Center's LDAC priority setting directly affects whether BCC can override the codec.

- **Priority on Sound Quality**  
  Forces LDAC (usually 990 kbps or 96 kHz Adaptive) via GATT.  
   This setting **locks LDAC**, and BCC **cannot override** it — even if Auto Switch or Intermediate profiles are enabled.

- **Priority on Stable Connection**  
  Defaults to SBC or fallback modes.  
   This setting **releases LDAC control**, allowing BCC or app-based renegotiation (UAPP, Neutron, etc.) to fully succeed after handshake.

 **To regain override control:**  
Change Music Center to **Stable Connection**, then:
1. Disconnect the headset.
2. Optionally apply SBC via Developer Options or Music Center.
3. Reconnect using BCC or Fast Pair (with override bypass).

###  Background Behavior — Music Center Codec Reassertion

Sony Music Center doesn’t just apply codec settings once — it registers a background **Bluetooth service** that monitors codec state and may silently reassert its LDAC mode when:

-  Headphones reconnect  
-  LDAC toggle is changed in system settings  
-  A playback app (like UAPP or Neutron) triggers a new LDAC session

 Even if you **swipe the app away**, these background receivers **remain active**.

---

###  How to Stop Music Center from Overriding LDAC

1. **Force stop** the app  
   `Settings → Apps → Sony Music Center → Force Stop`

2. *(Optional but recommended)*  
   **Clear app storage** to remove any stored LDAC quality settings

3. **Reconnect** or **power cycle** the headphones  
   → Ensures your BCC or UAPP profile isn’t silently overwritten

---

##  Headphone Firmware Storage Behavior (Sony WH-1000XM5)

Sony’s WH-1000XM5 can **store only a limited set of codec settings** in firmware between power cycles.

| Parameter                      | Stored in Firmware | How It's Set                      | Persistent? | Notes                                                                 |
|-------------------------------|--------------------|-----------------------------------|-------------|-----------------------------------------------------------------------|
| **Codec** (SBC / LDAC)        |  Yes             | Last active codec at power-off    |           | The most recently used codec is remembered.                          |
| **LDAC Quality Mode**         |  Yes             | Only via Sony Music Center        |           | "Priority on Sound Quality" = 990kbps<br>"Stable Connection" = Adaptive |
| **Sample Rate**               |  No              | Set by Android host at runtime    |           | Always needs to be re-applied on connect (e.g., via BCC or UAPP)     |
| **Bit Depth**                 |  No              | Set by Android host at runtime    |           | Cannot be stored in firmware                                         |

###  Firmware Persistence Table

| Setting                                 | Stored in Headphones? | Survives Power Cycle? | Notes                                           |
|-----------------------------------------|------------------------|------------------------|-------------------------------------------------|
| **LDAC/SBC mode (Sound Quality / Stable)** |  Yes                 |  Yes                 | Stored via Sony Music Center with AV ON         |
| **Sample Rate (e.g., 96kHz)**           |  No                  |  No                  | Always renegotiated per stream                  |
| **Bit Depth (e.g., 24-bit)**            |  No                  |  No                  | Decided by app/player, not stored in firmware   |
| **Developer Options codec**            |  No                  |  No                  | Reset on disconnect/reconnect                   |
| **BCC profile (990 kbps etc.)**        |  No                  |  No                  | Session-only unless re-applied each reconnect   |




 **Important:**  
- **Sony Music Center** is the **only app** that can store the LDAC **quality mode** (not the bitrate itself).
- **Sample rate and bit depth must always be forced** by the phone — either via:
  - Bluetooth Codec Changer (BCC),
  - Hi-res aware app (e.g., UAPP or Neutron),
  - Or codec handshake tricks.


>  **Warning: Sony Music Center silently reasserts LDAC settings**  
> If you've previously used Music Center to select a specific LDAC profile (e.g., 990 kbps), it may **automatically reapply that setting** on the next reconnect — *even if the app is no longer open*.  
>  
> To prevent this, you must either:  
> - **Set LDAC to 660 kbps** in the app before uninstalling (resets override state), or  
> - **Clear app data** or uninstall Music Center **before the next pairing**  
>  
> Simply uninstalling **after** a 990 kbps override won't erase the stored configuration from the headphone firmware.

