---
title: "Hardware & Firmware Forensics: UEFI SPI Flash Carving and Detecting Clandestine NVRAM Implants"
date: 2026-07-13
category: "dfir/firmware"
tags: [dfir, hardware, firmware, uefi, apt, nvram, rootkit]
---

# Hardware & Firmware Forensics: UEFI SPI Flash Carving and Detecting Clandestine NVRAM Implants

When analyzing platform compromise at the UEFI layer, forensic analysts traditionally search for structural modifications within the firmware executable code—specifically focusing on the patching or injection of malicious Driver Execution Environment (DXE) or Pre-EFI Initialization (PEI) drivers inside the SPI Flash layout. 

However, modern hardware-enforced security boundaries, such as Intel Boot Guard, AMD Hardware Validated Boot, and cryptographically anchored hardware roots of trust (Verified Boot), have drastically reduced the operational feasibility of direct binary patching. A tampered firmware image will inevitably fail cryptographic verification during the early processor execution phases, halting the boot sequence.

Consequently, advanced persistent threat (APT) actors (including BlackLotus, CosmicStrand, and MoonBounce) have shifted their persistence vectors toward legitimate configuration mechanics: **Non-Volatile RAM (NVRAM)**. This non-volatile storage region within the SPI Flash chip houses platform configuration states, including `Setup`, `BootOrder`, and critical Secure Boot key databases.

## The Mechanism of NVRAM Exploitation

Rather than modifying cryptographic binaries within write-protected or verified SPI regions, attackers exploit weak data isolation controls within the NVRAM storage layout. They inject malicious PE32 drivers, independent shellcodes, or deployment scripts directly into the data payload of custom or hijacked variables. 

During the DXE phase, the legitimate, verified firmware execution thread programmatically reads these NVRAM variables. If the vendor-specific variable parser or data handler contains programmatic vulnerabilities (such as integer overflows during buffer allocation calculations), the malformed variable triggers arbitrary code execution. This execution boundary occurs long before the operating system kernel initializes, completely bypassing OS-level defenses, Hypervisor-Protected Code Integrity (HVCI), and Endpoint Detection and Response (EDR) agents.

### Real-World APT Implementations

* **MoonBounce**: This implant achieved persistence by hooking the legitimate `GetVariable` function within the UEFI Runtime Services table. When the Windows kernel subsequently queried specific NVRAM parameters during initialization, the hooked function intercepted the request, executing an inline data replacement that mapped a kernel-mode payload directly into the address space of the loading OS.
* **BlackLotus**: This rootkit targeted the architectural foundation of Secure Boot by exploiting the Baton Drop vulnerability (`CVE-2022-21894`). By manipulating systemic NVRAM variables, it bypassed hardware restrictions to modify the authorized signature database (`db`). Injecting a custom certificate into the platform's trust chain enabled the legitimate loading of an unsigned, modified bootloader (e.g., custom GRUB or `bootmgr`). This bootloader disabled BitLocker, bypassed HVCI, and patched `ntoskrnl.exe` directly in memory.

---

## Low-Level NVRAM Binary Topology

On a raw SPI Flash binary dump, the NVRAM allocation region comprises a sequential array of distinct storage blocks known as *Variable Stores*. Regardless of the specific vendor implementation (Intel, AMD, or standard reference UEFI/PI specifications), the underlying baseline memory layout anchors to a `VARIABLE_STORE_HEADER` structure, immediately followed by sequential, raw binary variable records.

Each discrete variable entry is defined by a rigid header constraint (`VARIABLE_HEADER`):

| Offset | Type | Field Name | Description |
| :--- | :--- | :--- | :--- |
| `0x00` | `uint16_t` | `StartId` | Variable entry signature (typically `0x55AA`) |
| `0x02` | `uint8_t` | `State` | Transactional status state (`VALID`, `DELETED`, `IN_TRANSITION`) |
| `0x03` | `uint8_t` | `Reserved` | Reserved byte |
| `0x04` | `uint32_t` | `Attributes` | Access runtime flags (`NV`, `BS`, `RT`) |
| `0x08` | `uint32_t` | `NameSize` | Explicit byte length of the UTF-16 variable name string |
| `0x0C` | `uint32_t` | `DataSize` | Explicit byte length of the variable data payload |
| `0x10` | `Guid` | `VendorGuid` | 16-byte unique namespace identifier (GUID) |

Legitimate configuration elements enforce typical attribute masks combining `EFI_VARIABLE_NON_VOLATILE` (`0x01`), `EFI_VARIABLE_BOOTSERVICE_ACCESS` (`0x02`), and occasionally `EFI_VARIABLE_RUNTIME_ACCESS` (`0x04`). Attackers deliberately force a combined attribute mask of `0x07`. This mask ensures their injected payload remains persistent across both initial boot services phases and runtime operating system phases, allowing user-mode or kernel-mode malware to communicate directly with the underlying UEFI implant during OS runtime.

---

## Low-Level NVRAM Carving Framework

Standard firmware manipulation and analysis frameworks (such as `UEFITool` or `FirmwareModKit`) process NVRAM by following active file system indices and allocation tables. Consequently, they trust the internal integrity of the Variable Store indexes and will routinely miss orphaned, corrupted, or hidden variables. Comprehensive forensic verification requires building a custom signature-based carver to scan raw flash media.

### Advanced Carving & Verification Pipeline:

1. **Signature Alignment Scanning**: Scan the raw binary file for the `StartId` constant (`0x55AA`) strictly along predictable memory alignment boundaries (typically 4-byte or 8-byte boundaries).
2. **Structural Boundary Validation**: Eliminate false-positive hits by applying length validation bounds. The `NameSize` metric must not exceed rational constraints (typically capped at 512–1024 bytes), and the `DataSize` must remain within the physical boundaries of the NVRAM region (individual variables rarely scale beyond 64 KB).
3. **State Bitmask Analysis**: Evaluate the `State` byte configuration. If the byte resolves to a deleted status configuration (e.g., `VAR_DELETED` mapping to `0x3F` or `0x7F` depending on the flash controller logic), the active OS and standard parsers treat the space as empty. However, the raw payload remains physically intact on the chip until the firmware controller triggers a garbage collection cycle. Deleted space frequently retains historical implant iterations or temporary stager fragments.
4. **Entropy and Payload Signature Extraction**: Authentic configuration variables contain concise structs, plaintext string fragments, or cryptographic hashes (e.g., Secure Boot `db`/`dbx` fingerprints). If an extracted variable exposes an abnormally large data footprint, its payload must be evaluated for:
   * **Executable Magic Signatures**: Evaluating bytes for raw executable header magic signatures (`MZ`, `PE32`, or `TE`).
   * **High Structural Entropy**: Flagging dense, uniform byte distributions indicating packed or encrypted shellcode arrays.
5. **GUID Namespace Cross-Referencing**: Every vendor maps variables to strict, known GUID namespaces. The appearance of an unmapped, anomalous, or randomly generated `VendorGuid` bound to an exceptionally large data allocation is a high-fidelity indicator of platform compromise.

Reconstructing the operational timeline of NVRAM modifications by carving unallocated residual data fragments inside the raw SPI Flash dump represents the only definitive method to confirm UEFI-layer vulnerability exploitation and expose persistent hardware-level implants.

---

## Automation Tooling

The high-performance scanning utility designed to parse raw SPI dumps, validate variable header structures, and extract high-entropy payloads from deleted blocks is maintained as an external utility:

* **Production Tool Source Code**: [GitHub Gist: uefi-nvram-carver](https://gist.github.com/itsVentie/69870c6e9bc514e18875c6b54ab0e44e) 