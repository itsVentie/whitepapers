# Personal Research Archive

A centralized repository and knowledge base for academic and applied research in computer science, cryptography, and systems security. This repository functions as a technical mirror of the research log and community channel.

* **Author:** Ventie Ravelle
* **Contact:** [Telegram Profile](http://t.me/ventie)
* **Community Chat:** [Telegram Chat](https://t.me/+d7QGmOUQys9iZjk6)
* **Affiliation:** Independent Researcher

---

## Domains & Fields of Competence

* **Mathematics:** Higher Mathematics, Discrete Mathematics (Modular Arithmetic, Finite Fields, Matrix Theory).
* **Theoretical Computer Science:** Cryptography (including Lattice-based, Post-Quantum, FHE, Quantum Cryptography, and BQC).
* **Information Security:** Offensive & Defensive Security (Purple Teaming), Applied Security, Digital Forensics & Incident Response (DFIR), Vulnerability Research.
* **Systems Engineering:** Low-level, high-load, fault-tolerant, and secure systems architecture (Go, Rust, C++).

---

## Repository Structure & Classification

The archive is strictly modularized into functional segments to ensure maintainability, clear version control, and precise classification of research vectors:

### 1. CyberSecurity
Deep-dive applied security research, forensic timeline reconstructions, and cryptanalysis:
* **`dfir/` (Digital Forensics & Incident Response):**
  * `host/` — Low-level OS artifact carving (MFT, $USN Journal, RAM dumps, EVTX logs).
  * `media/` — Media forensics, steganalysis, and JPEG quantization table fingerprinting.
  * `firmware/` — Firmware reverse engineering (UEFI NVRAM parsing, SPI Flash dumps via JTAG/UART).
  * `cloud/` — Container infrastructure isolation breakthroughs, eBPF logging, and Docker/K8s escape vectors.
  * `malware/` — Malware analysis, advanced injection techniques (VAD structures), and custom YARA rule engineering.
* **`cryptography/` (Applied Cryptanalysis & Protocols):**
  * `fhe/` — Fully Homomorphic Encryption (BFV, CKKS schemes) and implementation-specific side-channel/timing attacks.
  * `pqc/` — Post-Quantum Cryptography implementations (ML-KEM, FIPS 203) and transition frameworks.
  * `protocols/` — Security analysis of complex network architectures (TLS 1.3 hybrid handshakes, Noise Protocol Framework).
  * `blockchain/` — Smart contract vulnerabilities, transaction deanonymization models, and modular nonce leakage attacks (Ed25519/secp256k1).
* **`osint/network/`:** Network intelligence, BGP/ASN routing anomaly detection, and automated C2 infrastructure mapping (Telegram API tracking).
* **`ai-forensics/`:** Investigation of AI incidents, model data poisoning forensics, and adversarial machine learning (LLM prompt/backdoor injection vectors).
* **`writeups/`:** Detailed technical writeups for hardcore CTF, Hack The Box, and TryHackMe advanced scenarios.

### 2. Development
Production-grade systems engineering, specialized software tooling, and architectural designs:
* **`go/`** — High-performance Go optimization, low-level memory handling (`unsafe`), `mmap` implementations, and custom protocol parsers.
* **`rust/`** — Memory-safe zero-allocation utilities, custom FFI bindings, and performance-critical security wrappers.
* **`architecture/`** — Formal specifications for distributed, fault-tolerant, and decentralized (p2p) secure communications platforms.

### 🔬 3. Research & Theory
The mathematical and theoretical backbone powering the applied implementations:
* **`ComputerScience/`** — Advanced algorithms (finite automata, bitwise operations) and complex structural data representations (B-Trees, Merkle Trees).
* **`Mathematics/`** — Theory of numbers, matrix representations, linear algebra lattices, and verification of modular reduction primitives (Barrett/Montgomery).
* **`Research/reviews/`** — Structural peer-reviews, analytical reports, and verification proofs of international academic papers (IEEE, IACR, ACM).

---

## Operation & Distribution Policy

* **Primary Purpose:** This archive is maintained primarily for internal verification, structured self-documentation, and tracking integration progress into the international scientific community. It is not optimized for a mainstream audience.
* **Licensing:** All theoretical frameworks, documentation, mathematical layouts, and code snippets are distributed under the **Creative Commons Attribution 4.0 International (CC-BY-4.0)** license.