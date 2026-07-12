---
title: "Timeline Reconstruction: Carving the Deleted $USN Journal and Analyzing NTFS Attribute Discrepancies"
date: 2026-06-07
category: "dfir/host"
tags: [ntfs, mft, timestomping, forensics, golang, mmap]
---

# Timeline Reconstruction: Carving the Deleted $USN Journal and Analyzing NTFS Attribute Discrepancies

During incident investigations, a common anti-forensics situation arises: an attacker drops a malicious payload, alters its timestamps via the WinAPI (`SetFileTime`) to blend into the environment among legitimate Windows system files, and subsequently obliterates the transaction log using the native administrative command:

```cmd
fsutil usn deletejournal /D C:
```

Standard forensic parsers (e.g., KAPE, MFTEcmd) fail to yield results in this scenario. They operate by reading the `$DATA` attribute of the journal file at the logical file system level; if the attribute is empty or unallocated, the parser assumes no relevant history exists.

## The Lower-Level Mechanics of Deletion

When the journal is deleted via `fsutil`:

1. The `$Extend\$UsnJrnl:$J` alternate data stream is truncated and cleared at the logical level.
2. Clusters previously allocated to the journal stream are marked as free inside the `$Bitmap` metadata file.
3. Crucially, the **physical transaction data is not overwritten or zeroed out immediately**. The records remain in unallocated space as raw data until subsequent OS write operations reuse those specific clusters.

## Detecting Timestomping via MFT Paradoxes

In the NTFS architecture, MACB (Modified, Accessed, Created, MFT Modified) timestamps are maintained in two completely independent attributes within a file's Master File Table (MFT) record:

1. **`$STANDARD_INFORMATION` (`0x10`)**: This attribute is exposed to and modified by user-mode applications. A call to `SetFileTime` directly alters the values stored here.
2. **`$FILE_NAME` (`0x30`)**: This attribute is tightly restricted and can only be updated by the kernel-level driver (`Ntfs.sys`) during explicit file operations such as initial creation, renaming, or hard link mapping.

Timestomping leaves a distinct chronological paradox: if the creation timestamp inside the `$FILE_NAME` attribute is **later** than the creation timestamp inside the `$STANDARD_INFORMATION` attribute, the file has unequivocally been manipulated.

To prove this manipulation and accurately extract the original, uncorrupted timeline of attacker activity, the investigator must recover the deleted USN records from unallocated space.

## Carving `$J` in Unallocated Space

Because the `$J` stream operates strictly in an append-only fashion, the structural integrity of `USN_RECORD_V2` remains highly predictable. The rigid binary layout allows for highly effective data carving directly from a raw physical disk image (`.dd`, `.raw`), completely bypassing broken file system logic.

### Carving Algorithm:

* **Signature Matching**: Scan for the explicit version constants: `MajorVersion = 2` and `MinorVersion = 0` at their designated offsets.
* **Length Validation**: Verify that the `RecordLength` field contains a structurally sane value (constrained between `60` and `4096` bytes) and adheres to 8-byte alignment rules.
* **Timestamp Sanity Boundary**: Filter out false positives by ensuring the `TimeStamp` field decodes to a valid `FILETIME` layout within a plausible historical window.

The `Reason` field within the extracted data structures contains precise transaction flags. Identifying the `USN_REASON_BASIC_INFO_CHANGE` (`0x00008000`) flag maps out the exact execution moment of the attacker's `SetFileTime` call with millisecond accuracy, exposing the actual window of compromise.

---

## High-Speed Automation Framework

To scale the carving process against multi-terabyte disk images without exhaustive memory consumption, the scanner implementation maps the image using memory-mapped files (`mmap`).

### Core Design Principles:

1. **Sliding Ring Buffer**: Data blocks are parsed through chunks that explicitly account for record structures spanning cluster and window boundaries.
2. **Integrity Validation**: Cross-reference the `FileReferenceNumber` against the parent directory record identifier to ensure structural relationship validity and eliminate ghost artifacts.

### Artifact Outputs

The automation pipeline generates structured JSON/CSV logs mapping the physical offset, exact UTC timestamp, transactional context flags (`USN_REASON`), and the target filename.

---

## Implementation Source

The optimized, highly concurrent scanning utility implemented in Go utilizes low-level `mmap` calls and worker pools to scan raw media images at maximum I/O throughput.

* **Production Tool Source Code**: [GitHub Gist: usn-carver-mmap](https://gist.github.com/itsVentie/e7753601770e7f9839326d269bbbf379)
