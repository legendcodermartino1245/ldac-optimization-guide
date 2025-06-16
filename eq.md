##  EQ Optimization (Wavelet Best Practices)
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

