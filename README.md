# LDAC Done Right

 There is a lot of misconception about LDAC and how to properly configure it on different operating systems (Windows and Android for example.) This guide will focus on both and will include strategies which help you get the best sound possible. I tried to make it as practical as possible to replicate. To make troubleshooting easier I have clearly documented the steps needed to set everything back to defaults for full transparency.


 My setup and the hardware I used during the making of this guide:
 
| Device         | OS / Firmware             | Supported Codecs                       |
|----------------|---------------------------|----------------------------------------|
| **Samsung S22 Plus** | One UI 6.1 (Android 14)    | LDAC, SSC, AptX, AAC, SBC               |
| **Samsung S24**      | One UI 6.1 (Android 14)    | LDAC, SSC, AptX, AAC, SBC               |
| **Windows 11**       | Version 24H2               | AAC, SBC *(LDAC not supported natively)* |
| **Windows 10**       | Version 22H2               | AAC, SBC *(LDAC not supported natively)* |
| **Sony WH-1000XM5**  | 2.4.1                      | LDAC, AAC, SBC                          |
| **Sony WH-1000XM3**  | 4.5.2                      | LDAC, AptX, AAC, SBC                    |

> 📌 *Note: On Windows, LDAC support requires specific Bluetooth drivers or third-party implementations (e.g. CSR Harmony stack or alternative USB dongles). This guide focuses on standard OS behavior unless otherwise noted.*
               
## Inner working of LDAC



LDAC supports sample rates ranging from **44.1 kHz to 96 kHz**, bit depths of **16-bit or 24-bit**, and quality modes of **330**, **660**, **990 kbps**, or **Adaptive**. Internally, LDAC encodes at 24-bit even when 16-bit is requested — making 24-bit the most stable and consistent choice.
The **Adaptive** mode dynamically switches between 330–990 kbps depending on available bandwidth and signal strength.



The table below lists all valid LDAC configurations tested and confirmed across both Android and Windows platforms.

### 🎛️ LDAC Configuration Matrix

| Sample Rate | Bit Depth | Bitrate (kbps) | Mode             |
|-------------|-----------|----------------|------------------|
| 44.1 kHz    | 16-bit    | 330            | Adaptive         |
| 44.1 kHz    | 16-bit    | 660            | Adaptive / Fixed |
| 44.1 kHz    | 16-bit    | 990            | Adaptive / Fixed |
| 44.1 kHz    | 24-bit    | 990            | Fixed            |
| 44.1 kHz    | 32-bit    | 990            | Adaptive         |
| 48 kHz      | 16-bit    | 330            | Adaptive         |
| 48 kHz      | 24-bit    | 660            | Adaptive / Fixed |
| 48 kHz      | 24-bit    | 990            | Adaptive / Fixed |
| 48 kHz      | 32-bit    | 990            | Adaptive         |
| 88.2 kHz    | 24-bit    | 990            | Fixed            |
| 88.2 kHz    | 32-bit    | 990            | Adaptive         |
| 96 kHz      | 24-bit    | 990            | Fixed            |
| 96 kHz      | 32-bit    | 990            | Adaptive         |

 Now that my setup and the software and hardware I use is clear let's dive into what bug in android I solved.

## The Real LDAC Bug: Quality Settings Don’t Apply on Their Own

There’s a hidden behavior in Android’s LDAC implementation that causes almost everyone to configure it wrong — even advanced users.

Here’s the issue:

> **Changing the LDAC quality setting (330 / 660 / 990 / Adaptive) by itself does nothing.**  
>  
> Unless the system renegotiates the entire codec connection, **your change won’t be applied** — even if the UI says it was.

### What Triggers a Real Codec Reset?
LDAC settings like bitrate, sample rate, and bit depth are **only renegotiated** when one of the following is changed:

- A different **codec** is selected (e.g. switching to SBC or AAC, then back to LDAC)
- The **sample rate** changes (e.g. 48kHz → 44.1kHz → back)
- The **bit depth** changes (e.g. 32-bit → 24-bit → back)

> **Only then will the LDAC handshake restart**, and Android will apply the new quality setting (330, 660, 990, or Adaptive).


> ⚠️ **This is a bug in Android's Bluetooth stack — not just a limitation.**  
> When you change LDAC bitrate, sample rate, or bit depth, Android often fails to renegotiate the codec connection.  
> The system UI may show the new values, but the actual audio stream continues using the old configuration.  
> **Your changes are silently ignored unless you force a full codec reset.**
> The bitrate setting always nneed to be reapplied after reconection cant be saved anywhere

## Samsung LDAC Override Stack
Samsung always enforces its own LDAC codec profile on Bluetooth handshake.
This override is hardcoded in the firmware-level negotiation and cannot be bypassed, removed, or overwritten

 What the Samsung Override Forces Every Time:
LDAC 96 kHz

32-bit

Adaptive bitrate (330–990 kbps, dynamic)

These values are pushed at the system level
In dev options "Default" is not a selectable bitrate — it indicates the override has taken effect.


