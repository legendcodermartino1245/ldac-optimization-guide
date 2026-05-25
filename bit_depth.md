# Inner workings of LDAC

LDAC supports sample rates ranging from **44.1 kHz to 96 kHz**, quality modes of **330**, **660**, **990**, **303**, **606**, **909** kbps, or **Adaptive**, and supports both **16-bit** and **24-bit PCM input**. LDAC’s configured input depth should generally reflect the effective output format of the playback chain, especially when avoiding unnecessary conversions or preserving high-precision DSP output.

| Playback Scenario                          | Player Output Depth | LDAC Bit-Depth Setting         | Rationale                                                                                        |
| ------------------------------------------ | ------------------- | ------------------------------ | ------------------------------------------------------------------------------------------------ |
| **Pure CD-quality (44.1 kHz / 16-bit)**      | 16-bit              | 16-bit (or "System Selection") | Matches playback chains carrying primarily 16-bit source information without implying artificial hi-res reconstruction. *System Selection* lets Android auto-choose bit-depth, which usually defaults to 16-bit unless overridden by a hi-res app. |
| **Native Hi-Res (>44.1 kHz / 24-bit)**       | 24-bit              | 24-bit                         | Allows LDAC to receive the high-resolution PCM output of the playback chain before compression.          |
| **Any source + DSP (EQ, gain, fades)**     | 24-bit              | 24-bit                         | Provides headroom for processing; avoids rounding errors during DSP before LDAC encoding.        |
| **Non–bit-perfect apps (mixed to 16-bit)** | 16-bit              | 16-bit (or "System Selection") | Reflects playback chains where the effective source information is limited to 16-bit resolution. |

>  **Clarification:**  
> LDAC does not restore missing precision or create additional detail. It encodes the PCM stream delivered by Android’s audio pipeline, typically as either 16-bit or 24-bit encoder input.
>
>  LDAC doesn’t transmit raw PCM — it uses a proprietary compression method combining MDCT and Huffman coding. While it’s lossy, LDAC is designed to retain detail up to 24-bit/96kHz with minimal perceptual degradation when properly configured.
>
>  LDAC’s 24-bit support refers to accepted PCM input resolution, not guaranteed transparent preservation of full 24-bit source detail after compression. The codec remains perceptually lossy at all bitrate modes.
>
> Android’s audio pipeline may internally resample, mix, or reformat audio before it reaches the LDAC encoder, depending on the device, Android version, and playback path.
>
> - If the playback chain only contains 16-bit source information, enabling 24-bit LDAC transport does not add meaningful fidelity, even if Android internally reformats the stream before encoding.
> - If the playback chain delivers genuine 24-bit PCM or high-precision DSP output, LDAC can accept and encode that higher-resolution input before compression.
> - Apps may process audio internally in 32-bit float, but Bluetooth codecs like LDAC ultimately receive quantized PCM input, typically up to 24-bit precision.
>
>  In practice, transport stability and pipeline consistency often matter more than theoretical hi-res capability. A stable fixed-rate LDAC chain with deterministic DSP behavior is generally preferable to a dynamically switching pipeline with inconsistent processing states.
