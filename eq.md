#  EQ Optimization (Wavelet Best Practices)
For users who want to apply EQ while preserving LDAC 990 kbps playback, Wavelet is the safest option.
###  Recommended EQ Setup
Use **Wavelet** as your only EQ. Disable all other music app EQs:
-  Neutron DSP (if not using high-res bypass)
-  UAPP parametric EQ
-  Poweramp tone controls or presets

Dont use eq in sound connect this will make ldac 303 or 330 based on sample rate

###  Wavelet Settings for LDAC Stability
| Setting         | Recommended Value                        |
|----------------|-------------------------------------------|
| Buffer Size     | **MAX** – Prevents dropouts and glitches |
| Legacy Mode     | **OFF** (unless your phone requires it)  |
| AutoEQ Profiles | **Enabled** – Clean tuning for most models |
Grant DUMP permission for Wavelet to access the audio session:
```bash
adb shell pm grant com.pittvandewitt.wavelet android.permission.DUMP
```



## Sound Connect EQ Has No Impact on LDAC Quality (WH-1000XM5)

When using the **Sony WH-1000XM5** with LDAC active, enabling or disabling **EQ in the Sound Connect app** does **not** affect LDAC quality, bitrate, or codec parameters.

### Confirmed Behavior:
- LDAC **remains active** at the **same bitrate and sample rate** regardless of EQ state.
- The **Developer Options “LDAC Audio Quality”** setting stays unchanged (e.g. 990 kbps) when EQ is toggled.
- No codec renegotiation or fallback (e.g. to AAC) occurs when EQ is applied.

### Why It Matters:
- On older Sony headphones (e.g. WH-1000XM3), enabling EQ **can** lower the effective LDAC bitrate or force fallback to AAC.
- On WH-1000XM5, **LDAC codec quality is preserved** even with audio processing features like EQ enabled — as long as override conditions aren't triggered (e.g. Samsung LDAC override or HD Audio toggling).

### Tip:
- You can safely use **EQ in Sound Connect** on XM5 **without compromising LDAC 990 kbps** or causing bitrate drops — unless **other system-level override conditions** interfere.
