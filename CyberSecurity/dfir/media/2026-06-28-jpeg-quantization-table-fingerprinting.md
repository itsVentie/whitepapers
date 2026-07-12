---
title: "Deanonymizing Media Sources via JPEG Quantization Table Fingerprinting in Telegram Repost Chains"
date: 2026-06-28
category: "dfir/media"
tags: [osint, forensics, jpeg, steganalysis, double-compression]
---

# Deanonymizing Media Sources via JPEG Quantization Table Fingerprinting in Telegram Repost Chains

During media forensics and open-source intelligence (OSINT) operations, investigators frequently encounter a complex tracking issue: a target image has been forwarded across 3 to 4 independent channels, the EXIF metadata is entirely stripped, and each channel has applied distinct visual alterations (such as cropping or watermarking). 

Standard verification methods—including reverse image search engines and metadata parsers—fail completely because metadata is generated from scratch upon each unique platform upload, and physical cropping masks obvious visual commonalities. However, tracking the source remains mathematically viable by extracting structural compression artifacts embedded within the JPEG format itself.

## Telegram Encoding Mechanics

When a user transmits an image through Telegram as a standard media payload (rather than an uncompressed document "file"), the platform forces a server-side re-encoding pass. The image is compressed into a JPEG configuration targeting a fixed Quality Factor (QF) of approximately 82. Additionally, downscaling algorithms are enforced if the spatial resolution exceeds predefined platform thresholds.

Consequently, regardless of the original capturing device (e.g., iOS/Android SDKs) or prior desktop editing suites (e.g., Photoshop, libjpeg, mozjpeg), the output image invariably inherits the unique quantization tables utilized by the Telegram encoder.

### The Physics of JPEG Quantization

JPEG compression relies on Discrete Cosine Transform (DCT) block processing. The resulting frequency coefficients are rounded using two distinct structural arrays:
1. **Luminance Matrix**: Handles brightness data (64 distinct coefficients mapped in an $8 \times 8$ grid).
2. **Chrominance Matrix**: Handles color-difference channels ($8 \times 8$ grid).

A nominal "Quality 82" designation is not standard across software suites. Different compression engines use proprietary formulas to scale raw quantization coefficients to achieve that target factor. Thus, the exact 128 coefficients extracted from a file form a highly predictive fingerprint of the specific encoder binary.

---

## Double-Compression Anomaly Detection

If a photo is downloaded from a channel and uploaded manually to another channel as a standard image (instead of using the platform's native forward feature or document routine), it undergoes **Double JPEG Compression**. Telegram decodes the stream back into raw pixel matrices and passes it through the encoder again.

If the primary quantization table (the matrix from the source image) diverges from the secondary table (Telegram's stable QF $\approx$ 82 matrix), the distribution of the final DCT coefficients retains a distinct, non-random statistical trace. Single-pass compression yields a relatively smooth distribution of coefficients. Double compression causes artificial clustering.

This phenomenon is quantified via **DCT Coefficient Histogram Periodicity Analysis** (leveraging methodologies outlined by Friedrich & Lukas). 

When analyzing the histogram of specific high-frequency or mid-frequency DCT coefficients, double quantization introduces predictable periodic gaps (characteristic comb-like peaks and valleys). Mathematically modeling the wavelength of this periodicity allows an investigator to reverse-calculate the exact properties of the primary quantization matrix used *prior* to the final upload.

---

## Practical Investigative Scenarios

### Scenario A: Verifying Original Uploads Between Two Sources
An investigator is evaluating identical images published by Channel A and Channel B to determine which account obtained the asset first. By analyzing the quantization properties:
* If Channel A's file exhibits a single-pass quantization layout matching a mobile device profile or known camera signature, while Channel B's file demonstrates clear double-compression artifacts matching the Telegram profile, **Channel A is programmatically verified as the earlier node in the chain**.

### Scenario B: Resampling Artifact Detection
Telegram enforces a fixed, deterministic resampling filter when shrinking high-resolution images. The interpolation footprint leaves specific rounding patterns within high-frequency DCT coefficients. These spatial anomalies differ substantially from resizing filters applied by commercial image editors (like GIMP or Photoshop). This distinction enables forensic examiners to differentiate between an authentic Telegram media file and an adversarial screenshot or external resave.

---

## Structural Method Limitations

Forensic investigators must account for several boundary conditions where this tracking model degrades:
* **Third-Party Editor Resaves**: If a threat actor passes the image through a local photo editor between channel uploads, a non-Telegram quantization table is injected at each interval. The tracking logic degenerates into a generalized double-compression analysis, losing platform-specific tracing capabilities.
* **Document Mode Transmissions**: If any link in the distribution chain transmits the image as a "File" (Document payload), Telegram completely bypasses the transcode pipeline. The original binary stream is preserved bit-for-bit, terminating the generation of new compression fingerprints.