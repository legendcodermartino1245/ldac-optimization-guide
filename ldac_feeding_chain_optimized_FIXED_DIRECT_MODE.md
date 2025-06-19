# LDAC Feeding Chain Optimization — Fully Consolidated

This chapter documents the fully reverse-engineered LDAC feeding chain optimization across Android, Windows, and Linux.  
Bit depth dominates codec behavior. Sample rate only affects frame density at any given bitrate.

---
## Practical UAPP Rule

Always manually set both UAPP PCM output and BCC target profile to 24-bit integer to guarantee clean feeding into LDAC’s 24-bit quantizer.

- BCC profile enforcement  
- Tasker automation  
- Exclusive Mode usage  
- Adaptive switching stability

✅ Bit-perfect feeding.  
✅ No mid-stream codec drops.  
✅ Clean adaptive sample rate switching.

---

## Additional Stability Rule

Never switch BCC profiles while A2DP session is active.  
❗ Never switch BCC profiles mid-stream.  
All codec switching must finish *before* A2DP handshake completes, or override risk and codec desync may occur.

---

## UAPP PCM Control Fragility Warning

UAPP does not directly control Bluetooth codec state — it only feeds PCM. Thus:

- Any A2DP session renegotiation (calls, UI interactions, unlock events) can cause LDAC fallback if BCC isn’t locked first.
- Tasker/BCC codec adjustments must precede UAPP playback start.

🔬 BCC locks codec stack.  
🎛 UAPP optimizes PCM feeding.

---

## Global Recommendation

ℹ️ **Firmware Note:** On Sony WH-1000XM headphones, only a codec selected via a **firmware-writing method** (e.g., **Sony Music Center**, **Developer Options codec toggle**, or **SBC→LDAC handshake**) is stored in the internal codec profile.

> ⚠️ On **Samsung devices**, the system-injected LDAC override always applies first at A2DP connect — regardless of which codec becomes active afterward.

To persist **LDAC 990** or **SBC** as the stored profile, use **Music Center** or **BCC automation** to forcibly apply a new codec *after* connect.  
UAPP **cannot** override Samsung’s default — it must follow a codec already set by **BCC** or any other method capable of codec injection.




Always use 24-bit for both BCC profile locking and UAPP PCM output.  
16-bit profiles are reserved solely for intermediate handshake defeat (override reset).

---

## Bit Depth Priority

- 24-bit always preferred for LDAC.
- Feeding 16-bit into 24-bit LDAC is safe — zero-padding occurs; no loss of fidelity.
- Feeding 24-bit into 16-bit causes irreversible quantization loss — lower 8 bits are discarded, reducing true dynamic range.

---

## BCC Bit Depth Enforcement (Android Only)

- Only BCC directly enforces LDAC bit depth at Bluetooth stack level.
- Operates deeper than UAPP, Neutron, or Developer Options.
- Allows full codec-level automation via Tasker.

---

## Bit Depth Upsampling Strategy

- Bit depth upsampling (16 → 24-bit) maximizes LDAC quantization precision.
- Pre-convert sources to 24-bit PCM to avoid Android resamplers.
- Avoid 32-bit float containers; always feed 24-bit integer PCM into LDAC chain.
- Guarantees platform-consistent behavior (Android, Windows, Linux).

---

## Sample Rate Strategy

| Source → Target     | Optimal? | LDAC Bitrate Benefit? | Notes                         |
|---------------------|----------|------------------------|-------------------------------|
| 44.1 → 44.1 kHz     | ✅ Yes   | ✅ Yes                | Native; preserves timing      |
| 44.1 → 88.2 kHz     | ✅ Yes   | 🔸 Slight             | Valid 2× upsample             |
| 44.1 → 96 kHz       | ❌ No    | ❌ None               | No gain; filler only          |
| 48 → 48 kHz         | ✅ Yes   | ✅ Yes                | Native Android-safe           |
| 96 → 96 kHz         | ✅ Yes   | ✅ Yes                | Ideal for hi-res sources      |
| 176.4 / 192 → 96    | ✅ Yes   | ✅ Indirect           | Clean downsample to preserve  |
| 88.2 → 96 kHz       | ❌ No    | ❌ None               | Non-integer resample, avoid   |

### Additional Target Notes

| Target | Notes |
|--------|-------|
| 48 kHz / 24-bit | Safest for Android pipeline |
| 88.2 kHz / 24-bit | Clean 2× integer upsample |

> Upsampling 44.1 → 96 adds no LDAC bitrate packing benefit; only native rate maintains full temporal accuracy at given bitrate.  
> Upsampled CD masters remain valid if stored clean as 24-bit.

---

## High-Resolution Handling (176.4 / 192 kHz)

- Downsample 176.4/192 → 96 kHz / 24-bit for LDAC feeding.
- Fake hi-res (CD upsampled) remains valid as long as 24-bit PCM integrity is preserved.
  Note: While upsampling doesn’t add true resolution, 24-bit containers avoid float-to-int artifacts and maintain LDAC compatibility.

---

## App Resampling Logic

- App resampling influences the PCM input into Bluetooth stack encoder only; does not affect LDAC codec profile selection once stack is locked via BCC.
- Apps control PCM input feeding into Bluetooth stack encoder.

---

## Platform Parity

- Android (BCC + Tasker)
- Windows (Default & Alternative A2DP Drivers)
- Linux (PipeWire, BlueZ, PulseAudio)

LDAC rule remains universal:  
**Bit depth first — sample rate second.**

---

## Automation (Android Exclusive)

- BCC + Tasker enable fully deterministic codec negotiation.
- All codec profile locks must execute **before** A2DP session initialization.
- UAPP playback must always start after BCC profile application.

---

## Absolute Summary Rule

> For LDAC, clean 24-bit PCM containers always dominate sample rate — whether native 44.1 or upsampled 96 kHz.

---

## UAPP vs BCC — Codec Control Summary

| Capability | UAPP | BCC |
|-------------|------|-----|
| Codec Parameter Control | PCM feeding only | Full codec lock (stack level) |
| Bit Depth Enforcement | PCM feeding only | Full Bluetooth codec enforcement |
| Adaptive Bitrate Control | No | Full chaining support |
| Tasker Automation | ❌ (not supported) | ✅ Full system integration |
| Multipoint Handling | Fragile | Fully stable |
| Mid-Stream Stability | Susceptible to renegotiation | Fully persistent |

---

## UAPP Direct(LDAC) Mode — Feeding Chain Behavior

UAPP achieves maximum LDAC stability when Direct(LDAC) mode is fully engaged.

### Why Direct(LDAC) Mode Matters

| Advantage | Explanation |
|-----------|-------------|
| Mixer Bypass | Fully bypasses AudioFlinger (no system resampling) |
| PCM Feeding | Feeds 24-bit PCM directly into Bluetooth stack |
| Codec Lock Synergy | Preserves BCC's LDAC profile lock underneath |
| Renegotiation Protection | Prevents most Android stack interference |
| Frame Packing Integrity | Ensures maximum LDAC frame density and prevents quantization mismatch |

> ⚠️ **Warning:** Without Direct LDAC active, UAPP defaults to Android mixer path.  
> All feeding purity and LDAC lock guarantees are lost.

### Mandatory Operational Rule

Always confirm Direct(LDAC) is active inside UAPP when used with BCC.  
If not active → system mixer (AudioFlinger) re-engages → stability is compromised.

> ⚠️ **Warning:** Bit-perfect mode preserves source bit depth.  
> If feeding 16-bit sources, manually upsample to 24-bit for optimal LDAC frame packing.

### Feeding Chain Sequence

1. BCC applies codec profile lock (bit depth + bitrate).  
2. UAPP feeds clean PCM directly via Direct(LDAC).  
3. LDAC frame packing remains fully stable.

---
