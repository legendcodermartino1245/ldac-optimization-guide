
#  EQ Optimization (Wavelet Best Practices)

For users who want to apply EQ while preserving LDAC 990 kbps playback, Wavelet is the safest option on most setups — especially on older Sony headphones.

###  Recommended EQ Setup
Use **Wavelet** as your only EQ. Disable all other music app EQs:
-  Neutron DSP (if not using high-res bypass)
-  UAPP parametric EQ
-  Poweramp tone controls or presets

###  Wavelet Settings for LDAC Stability
| Setting         | Recommended Value                        |
|----------------|-------------------------------------------|
| Buffer Size     | **MAX** – Prevents dropouts and glitches |
| Legacy Mode     | **OFF** (unless your phone requires it)  |
| AutoEQ Profiles | **Enabled** – Clean tuning for most models |

Grant DUMP permission (only once via ADB on PC) for Wavelet to access the audio session:
```bash
adb shell pm grant com.pittvandewitt.wavelet android.permission.DUMP
```

> *Required only once after installing Wavelet.*

> ⚠️ *If Wavelet fails to apply EQ consistently, try enabling **Legacy Mode** — especially on Android 10/11 or MIUI-based systems. Note: Legacy Mode may reduce compatibility with session-aware apps like UAPP.*  

### 🔄 Sound Connect EQ vs. Wavelet — Comparison

| Feature                     | **Sound Connect EQ**                           | **Wavelet EQ**                                 |
|----------------------------|--------------------------------------------------|------------------------------------------------|
| **LDAC 990kbps Safe (XM5)**| ✅ Yes — no fallback on XM5                    | ✅ Yes (with DUMP + max buffer)                |
| **LDAC 990kbps Safe (XM3)**| ⚠️ No — may force 330/303 or AAC               | ✅ Yes (more stable across all devices)        |
| **Latency**                | ✅ Low — system-level, minimal delay           | ⚠️ Moderate — added buffer increases latency   |
| **Sound Quality**          | ⚠️ Mid-tier — Sony preset EQ curves            | ✅ Higher — especially with AutoEQ or parametric|
| **Custom Profiles**        | ❌ Limited (manual sliders)                    | ✅ Deep control, AutoEQ, parametric, FIR/IIR   |
| **System Resource Usage**  | ✅ Low                                          | ⚠️ Higher — uses its own audio session         |
| **Compatibility**          | ✅ Optimized for Sony headphones                | ⚠️ Some quirks on phones without DUMP access   |
| **Preservation Across Apps** | ✅ Always active system-wide                   | ⚠️ Not system-wide — Wavelet hooks into active playback sessions and may be bypassed by apps like UAPP or Neutron in Hi-Res mode unless configured to allow it |

❗ **Important:** Never run multiple EQs simultaneously (e.g., Sound Connect + Wavelet + app EQ). This causes DSP conflicts and may lower LDAC bitrate or disable EQ entirely.

#### 🧠 Summary — When to Use Which EQ

| Use Case                                | Best EQ Choice       | Why                                                                 |
|-----------------------------------------|----------------------|----------------------------------------------------------------------|
| WH-1000XM5 — stability + simplicity     | ✅ **Sound Connect** | Zero latency, no impact on LDAC 990, fully integrated with Sony DSP |
| WH-1000XM3 or older Sony headphones     | ✅ **Wavelet**        | Prevents LDAC fallback; Sound Connect EQ may drop bitrate           |
| Precise tuning or AutoEQ corrections    | ✅ **Wavelet**        | Offers AutoEQ, parametric/FIR filters for detailed sound shaping    |
| Gaming or real-time use (low latency)   | ✅ **Sound Connect** | Minimal delay, safer for video/gaming with no DSP overhead          |
| Want full control over EQ profiles      | ✅ **Wavelet**        | Manual parametric + FIR/IIR support                                 |

---

## 🔍 Sound Connect EQ — Stable LDAC 990 on WH-1000XM5

✅ Verified: On **WH-1000XM5**, toggling EQ in the Sound Connect app **does not reduce LDAC bitrate or cause fallback.** You will stay locked at **990 kbps LDAC**, even with EQ on.

### ⚠️ Important Note:
- This applies only to **XM5 and newer** models.
> ❗ *Using EQ in Sound Connect reduces LDAC bitrate on older headphones (like WH-1000XM3), but does **not** affect LDAC 990 kbps on the WH-1000XM5.*
