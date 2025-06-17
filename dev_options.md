#  Developer Options Are Safe — If You Clean Up Properly

>  Just disabling Developer Options is **not enough** if LDAC was ever manually selected.

Samsung may silently continue applying the **last used LDAC override profile** even after Developer Options are turned off — especially if it was previously set to LDAC 660, Adaptive, or 990.  
LDAC supports both 16-bit and 24-bit input. But the encoder only uses what it receives — not what you select in Developer Options.

###  Correct Reset Procedure

1. Enable **Developer Options** — only if you’ve used them before  
2. Set **Bluetooth Audio Codec** to **SBC**
   - ℹ On Samsung, there is **no “Default” option** — selecting **SBC** is the only way to fully clear override behavior  
3. Exit the Developer Options menu  
4. Go back and **disable Developer Options**  
5. *(Optional but safest)* Reset network settings:  
   - `Settings → General Management → Reset → Reset network settings`

 This fully clears Samsung’s override memory, ensuring a clean LDAC handshake window for hijack and BCC profile injection.


##  Developer Options: Bit Depth Misconceptions

Changing the **bit depth** in Developer Options (e.g., from 32-bit to 24-bit or 16-bit):

-  **Does NOT actually change** the real output bit depth.  
-  The system almost always stays locked at **32-bit**, even if 24-bit is selected.

However:

-  **Toggling the bit depth or sample rate** does **reapply the LDAC quality mode setting** (like forcing a renegotiation of 990 kbps or Adaptive).
-  This makes Developer Options useful for *triggering codec behavior changes*, but not for controlling the bit depth directly.
