##  Complete LDAC Codec Reset Matrix (v2025)

---

###  HEADSET-SIDE RESETS (Sony WH-1000XM Series)

| #  | Method                                                       | Resets Codec Profile | Resets Pairing | Notes                                                                 |
|----|--------------------------------------------------------------|----------------------|----------------|-----------------------------------------------------------------------|
| 1  | Hardware Reset (Power + Custom 7s)                           |  Yes               |  Yes         | Deepest reset — wipes firmware-level codec and pairing data          |
| 2  | Software Reset via Headphones Connect (System > Initialize) |  Yes               |  No          | Clears LDAC quality, EQ, app prefs — pairing info remains            |
| 3  | Leave Powered Off 10+ sec After Clean Handshake             |  Conditional        |  No          | Stores most recent profile (SBC or LDAC) — used for exploits, not reset |
| 3b | Power Off Headphones After Clean SBC or LDAC 16-bit Handshake |  (Firmware)        |  No          | Preserves clean handshake profile — crucial for override bypass      |

---

###  PHONE-SIDE RESETS (Samsung)

| #  | Method                                                     | Resets Codec Profile | Resets Samsung Override | Notes                                                                 |
|----|------------------------------------------------------------|----------------------|--------------------------|-----------------------------------------------------------------------|
| 4  | Reset Network Settings (General Management > Reset)       |  Yes               |  Yes                   | Resets Bluetooth, Wi-Fi, and override stack                          |
| 5  | Forget Bluetooth Device                                   |  Partial           |  Partial               | Removes pairing, but override often persists                         |
| 6  | Set Codec to SBC in Developer Options                     |  Yes               |  Yes                   | "Default" not available — SBC forces override flush                  |
| 7  | Disable Dev Options After SBC Set                         |  Yes               |  Yes                   | Locks in override removal, blocks LDAC reapplication                |
| 7b | Intermediate Profile Switch (SBC → LDAC 16-bit → LDAC 990 via BCC) |  Yes        |  Yes                   | Bypasses Samsung override with staged codec negotiation              |
| 8  | Disable HD Audio (in BT settings)                         |  No                |  May disable session     | Only disables session — override logic survives                      |

---

###  APP-LEVEL RESETS (Sony, BCC, Google Services)

| #  | Method                                                      | Resets Codec Profile | Blocks Future Override | Notes                                                                  |
|----|-------------------------------------------------------------|----------------------|------------------------|------------------------------------------------------------------------|
| 9  | Clear Storage: Sony Music Center                           |  App-side           |  Yes                | Prevents stored LDAC quality (660/990) from being reapplied            |
| 10 | Clear Storage: Headphones Connect (Sound Connect)          |  App-side           |  Yes                | Wipes EQ and LDAC behavior control                                     |
| 11 | Clear Storage: Bluetooth Codec Changer (BCC)               |  Profile mappings   |  No                 | Clears profile switch logic and XML config                             |
| 12 | Uninstall Sony/BCC Apps (no data cleared)                  |  No                |  No                 | Prevents future app changes — existing override remains                |
| 13 | Clear Saved Devices (Settings > Google > Devices & Sharing) |  No               |  Fast Pair metadata | Removes cloud pairing + override profile sync                          |
| 14 | Disable Nearby & Scanning (Location Settings)              |  No                |  Reduces interference | Blocks Google Services override triggers                               |

---

###  TEMPORARY / VOLATILE SESSION CLEARS

| #  | Method                                               | Resets Codec Profile | Duration         | Notes                                               |
|----|------------------------------------------------------|----------------------|------------------|-----------------------------------------------------|
| 15 | adb shell am force-stop com.android.bluetooth       |  No                | Until reconnect  | Ends A2DP session — clears temp override            |
| 16 | adb shell am force-stop com.google.android.gms       |  No                | Until restart    | Interrupts Nearby + Fast Pair syncing              |
| 17 | Airplane Mode Toggle + Reboot                        |  No                | Temporary        | Clears memory stack, not override                  |
| 18 | Reconnect Using Fast Pair (not BT settings)          |  Variable           | Variable         | Avoids override if done post-reset                 |
| 19 | Bluetooth Input Toggle (gear icon > input device)    |  No                |  No            | No effect on LDAC override                         |
---
