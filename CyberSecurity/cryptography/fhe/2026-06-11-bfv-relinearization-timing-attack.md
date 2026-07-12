---
title: "Chosen-Ciphertext Attack on BFV via Relinearization Key Leakage and NTT Timing Side-Channels"
date: 2026-06-11
category: "cryptography/fhe"
tags: [cryptography, fhe, bfv, cryptanalysis, ntt, side-channel, appsec]
---

# Chosen-Ciphertext Attack on BFV via Relinearization Key Leakage and NTT Timing Side-Channels

When auditing Fully Homomorphic Encryption (FHE) services, a critical architectural vulnerability often arises: while the developer may correctly implement the BFV (Brakerski-Fan-Vercauteren) scheme and properly select secure parameters ($n$, $q$, and $t$), the execution environment remains deeply flawed. Specifically, the **relinearization keys** are generated once at startup and persist statically in memory for the entire uptime of the service. 

If the application operates as an oracle—accepting arbitrary ciphertexts, performing homomorphic multiplications, and returning the result—it becomes vulnerable to a highly targeted timing side-channel attack. Standard cryptographic analysis tools completely miss this flaw because they focus exclusively on parameter validation rather than operational execution times at the hardware instruction level.

## The Mechanics of Relinearization and EVK Exposure

In the BFV scheme, multiplying two standard ciphertexts increases the dimensionality of the resulting ciphertext. If $ct_1$ and $ct_2$ are represented as polynomial pairs $(c_0, c_1)$, their homomorphic product expands into a polynomial triple $(c_0, c_1, c_2)$. To prevent exponential growth of the ciphertext size during subsequent operations, **relinearization** is performed to collapse the triple back into a standard pair $(c_0', c_1')$.

This dimensional reduction requires Evaluation Keys ($evk$, also known as relinearization keys). Mathematically, the $evk$ is an encryption of the secret key $s$ squared ($s^2$), scaled by a special decomposition basis to manage noise expansion:

$$evk \approx \text{Enc}_s(\text{Base} \cdot s^2)$$

Because the $evk$ and the secret key $s$ are directly related through this algebraic structure, obtaining granular information about the internal coefficients of the $evk$ allows an attacker to reconstruct the secret key $s$ entirely, completely breaking the confidentiality of the FHE scheme.

## The NTT Modular Reduction Side-Channel

The core performance of polynomial multiplication within modern FHE libraries relies heavily on the **Number Theoretic Transform (NTT)** to reduce the computational complexity from $\mathcal{O}(n^2)$ to $\mathcal{O}(n \log n)$. 

During the execution of the NTT, modular reduction routines (such as naive implementations of Barrett or Montgomery reduction) frequently introduce conditional execution paths. A classic vulnerability involves the conditional residue correction step:

```c
// Example of a non-constant-time conditional residue correction
if (intermediate_result >= q) {
    intermediate_result -= q;
}

```

This conditional branch introduces a subtle timing discrepancy: the execution time for a coefficient that triggers the subtraction differs from one that does not by a few nanoseconds or clock cycles.

## Attack Vector: Exploiting Static EVK Interaction

During the relinearization phase, the system executes the NTT directly on the components of the static $evk$. Because the $evk$ coefficients are fixed at startup, they act as a persistent target.

By submitting carefully crafted, chosen ciphertexts ($ct_{chosen}$), an attacker can deliberately manipulate the interaction between the coefficients of the input ciphertext and the static coefficients of the $evk$ during polynomial multiplication.

### The Attack Pipeline:

1. **Chosen-Ciphertext Injection**: The attacker sends a sequence of ciphertexts where specific polynomial coefficients are systematically isolated or altered.
2. **Differential Timing Measurement**: The attacker measures the total execution time of the homomorphic multiplication operation with microsecond or nanosecond precision.
3. **Coefficient Leaking**: By analyzing the statistical variance in processing times, the attacker determines exactly when the conditional reduction (`-= q`) was triggered.
4. **Secret Key Reconstruction**: The recovered algebraic relationships between the input coefficients and the leaked $evk$ boundaries allow the attacker to calculate the values of the static evaluation keys, ultimately exposing the underlying secret key $s$.

---

## Remediation Strategy

To mitigate this architectural side-channel leak, FHE service deployments must enforce the following cryptographic controls:

* **Constant-Time Primitives**: Replace all conditional modular reduction steps in the NTT implementation with bitwise, constant-time arithmetic (e.g., complete Montgomery reduction without conditional branching).
* **Key Rotation**: Avoid long-lived static evaluation keys ($evk$) for multi-tenant or publicly accessible oracles.
* **Blinding Operations**: Introduce random cryptographic blinding factors to the ciphertexts prior to relinearization to mask the deterministic timing characteristics of the underlying coefficients.

---