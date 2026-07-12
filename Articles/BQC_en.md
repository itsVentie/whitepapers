## 1. Abstract

This paper examines the “Quantum-as-a-Service” (QaaS) model for organizing remote computing. Delegating quantum algorithms to untrusted servers creates risks of compromising the confidentiality of input data and the structure of the programs being processed. This paper proposes a Blind Quantum Computing (BQC) protocol that ensures unconditional secrecy and verifiability with minimal hardware requirements for the user. A hybrid verification method has been developed that combines trap qubits with surface code syndrome analysis, allowing for the classification of physical decoherence noise and active server attacks. The client’s quantum memory requirement is reduced to an asymptotic limit of $O(1)$ with a classical memory capacity of $O(n)$. The security of the proposed architecture has been proven within the framework of the Universal Composability model.
## 2. Introduction

The physical implementation of scalable quantum processors is currently limited by technological challenges in maintaining the stability of cryogenic systems and suppressing external noise. Under these conditions, the dominant approach to organizing high-performance computing is the use of remote cloud infrastructure within the Quantum-as-a-Service (QaaS) paradigm. Nevertheless, delegating computations within QaaS architectures inevitably raises questions about the confidentiality of the algorithms being processed and the initial states transmitted to an untrusted party.

The use of standard classical cryptographic methods, including fully homomorphic encryption (FHE), does not protect quantum data directly during the execution of logical gates. Performing unitary transformations on an encrypted state requires direct physical interaction with qubits. This allows an untrusted server to perform unauthorized projective measurements or entangle the target system with auxiliary registers for subsequent information exfiltration.

The main result of this work is the construction and theoretical justification of a blind quantum computing protocol that allows these limitations to be circumvented. We have succeeded in showing that the requirements for client-side quantum memory can be reduced to an asymptotic limit of O(1), which completely eliminates the need for local storage of the user’s quantum states. The paper also presents a method for distinguishing between physical decoherence noise and targeted server attacks, implemented through the combined use of verification trap qubits and quantum error correction via a surface code. The security of the proposed architecture is rigorously proven within the framework of universal composability (UC) under arbitrary behavior of the remote node. Furthermore, an analytical relationship is derived linking the density of verification positions in the computation graph to the upper bound on the probability of a destructive attack by an attacker going undetected.
## 3. Mathematical Framework 

The proposed architecture is based on measurement-based quantum computing (MBQC), where the standard gate model is replaced by adaptive single-qubit measurements in a highly entangled many-particle state.

Let $G = (V, E)$ be a computation graph, where the vertices $V$ represent qubits initialized in the state $\left|+\right\rangle$, and the edges $E$ represent entanglement links. The server generates the cluster state $\left|G\right\rangle$ by applying controlled phase shift operators $CZ$ to all edges:

$$\left|G\right\rangle = \prod_{(i,j) \in E} CZ_{i,j} \left|+\right\rangle^{\otimes V}$$

Computations in the MBQC are performed by measuring individual qubits in the $XY$ plane of the Bloch sphere. The measurement basis for qubit $j$ is parameterized by the angle $\theta_j \in [0, 2\pi)$, yielding the states:

$$\left|\pm_{\theta_j}\right\rangle = \frac{1}{\sqrt{2}} \left( \left|0\right\rangle \pm e^{i\theta_j}\left|1\right\rangle \right)$$

The measurement result $s_j \in \{0, 1\}$ introduces a probabilistic nature to the collapse of the wave function. To preserve the determinism of logical computations, Pauli corrections are applied. If $\phi_k$ is the nominal angle for the subsequent qubit $k$, the client updates it based on the previous results $\mathbf{s}$:

$$\phi_k' = (-1)^{X_k(\mathbf{s})} \phi_k + Z_k(\mathbf{s})\pi$$

where $X_k$ and $Z_k$ are binary parity functions defined by the graph’s flow.
## 4. Threat Model and Security Objectives 
To formalize the proofs, it is necessary to define the computational constraints of the interacting parties: the Client ($C$) and the Malicious Server ($S^*$).

**Capabilities of the attacker ($S^*$):**

The server possesses unlimited quantum and classical computational resources. It has complete control over quantum memory and communication channels, but is strictly subject to the laws of quantum mechanics (the no-cloning theorem).

**Client ($C$) Constraints:**

The client is a “nearly classical” device. It is capable of generating single photons with perfect phase accuracy but has zero quantum memory ($O(1)$). Its classical memory is limited to $O(n)$, where $n$ is the number of vertices in the graph.


A protocol is considered perfectly blind if the state density accessible to the server is statistically independent of the client’s algorithm input. For any two algorithms $(x_1, U_1)$ and $(x_2, U_2)$ that generate the same computation graph, the trace distance must be zero.

## 5. Proposed Solution

The interaction between the user’s classical terminal and the remote quantum platform is organized as a synchronous, step-by-step data exchange, which prevents the server from accumulating non-randomized information about the algorithm’s structure.

During the initialization phase, for each node $j \in V$ of the spatial computation graph, the client locally generates a random classical bit $X_j \in \{0,1\}$ and an integer masking parameter $Y_j \in \{0, 1, \dots, 7\}$. Using these values, the user prepares a single photon in the equatorial state $\left|+_{\theta_j}\right\rangle$, where the initial phase shift is strictly determined by the relation $\theta_j = Y_j \frac{\pi}{4}$, and then transmits the prepared qubit via a quantum communication channel to the server.

After receiving all physical carriers, the untrusted platform performs the entanglement procedure by sequentially applying two-qubit controlled-phase-shift operators $CZ$ to all pairs of nodes forming the set of edges $E$ of the computational lattice. Since the initial phase angles $\theta_j$ are uniformly distributed on the Bloch sphere, the resulting density matrix of the entire system—as seen from the perspective of the remote processor—is invariant with respect to the structure of the algorithm being executed and represents a maximally mixed state.

To implement a logic gate, the client computes the nominal measurement angle $\phi_j'$, taking into account the Pauli corrections accumulated as a result of processing the previous steps of the data flow graph. Based on the resulting value, a masked measurement angle $\delta_j$ is formed, which is transmitted to the server via a classical communication channel:

$$\delta_j = (-1)^{X_j} \phi_j' + Y_j \frac{\pi}{4}$$

The server then performs a projective measurement of the target qubit $j$ in the $XY$ plane in the basis $\left|\pm_{\delta_j}\right\rangle$ and returns the resulting binary outcome $s_j \in \{0,1\}$ to the user. In the final stage of the round, the client performs the decoding procedure, computing the true value of the logical bit using the exclusive OR operation: $r_j = s_j \oplus X_j$.

The applied mathematical transformation $\delta_j = (-1)^{X_j} \phi_j' + Y_j \frac{\pi}{4}$ implements a quantum one-time pad scheme in phase space. For the remote party, which has no prior knowledge of the secret keys $X_j$ and $Y_j$, the distribution of the measured angles $\delta_j$ is identical to white noise, which maximizes the entropy of intercepted messages and guarantees that the protocol’s blindness condition is satisfied.

## 6. Verification and Fault Tolerance

To ensure verifiability and resilience in the presence of physical noise (the NISQ era), the protocol integrates two independent layers: **trap qubits** for detecting active attacks and **a surface code** for correcting natural decoherence errors.

### 6.1. Mathematical Model of Noise and Traps

When analyzing the protocol’s robustness, it is assumed that every quantum operation and two-qubit gate on the server side is subject to local depolarizing noise $\mathcal{E}_p$. The cumulative effective noise per node of the spatial graph, taking into account the superposition of errors during the sequential application of $CZ$ gates, is approximated by an operation with an effective error of $p_{\text{node}} \approx 25 \cdot p_{\text{calib}}$:

$$\mathcal{E}(\rho) = (1-p_{\text{node}})\rho + \frac{p_{\text{node}}}{3}(X\rho X + Y\rho Y + Z\rho Z)$$

To verify the integrity of the computations, a set of check nodes (traps) $T \subset V$ with a fixed density $\nu = |T|/|V|$ is randomly inserted into the graph $G=(V,E)$. This set is orthogonally divided into two subsets: trap qubits in the Bloch sphere plane ($T_{XY}$) and trap qubits along the quantization axis ($T_Z$).

Check nodes of the first type ($t \in T_{XY}$) are initialized in the equatorial plane of the Bloch sphere in the states $$\left|+_{\theta_t}\right\rangle = \frac{1}{\sqrt{2}} \left( \left|0\right\rangle + e^{i\theta_t} \left|1\right\rangle \right)$$, where the phase shift is given by a discrete set of angles $\theta_t = Y_t \frac{\pi}{4}$. Traps of the second type ($t \in T_Z$) are prepared in the eigenstates of the Pauli operator $Z$, that is, $\left|0\right\rangle$ or $\left|1\right\rangle$. Since the controlled phase shift operator acts on the basis states as $CZ|0\rangle| \cdot \rangle = |0\rangle| \cdot \ rangle$, the server’s application of two-qubit entanglement gates according to the topology of the computation graph does not lead to quantum correlation between nodes $t \in T_Z$ and the main computational subgraph. This ensures their mathematical isolation during the execution of the algorithm.

In the absence of destructive interference from the server, the expected measurement result of any trap after demasking is $r_t = 0$. Under the influence of background node noise, there is a nonzero probability of inversion of the recorded outcome, which for the depolarization model under consideration is:

$$P(\text{fail} \mid t) = \frac{2}{3}p_{\text{node}}$$

### 6.2. Hybrid Differentiation Algorithm (QEC vs. Adversary)

To ensure the fault tolerance of logical computations, a Surface Code with code distance $d$ is integrated into the protocol. During the execution of quantum operations, a remote untrusted server continuously computes the syndromes of the $X$ and $Z$ stabilizers, after which it transmits the resulting classical data to the client’s terminal. Protection of computations against coherent destructive interference and background noise from NISQ nodes is based on a two-stage verification procedure performed by the user’s classical automaton.

In the first stage of validation, the client estimates the weight $w$ of the computed error syndrome in the Surface Code. If the calculated value exceeds a strict threshold for the error-correcting capacity of the code structure being used—that is, if the condition $w > \lfloor (d-1)/2 \rfloor$ is satisfied—the classical automaton immediately initiates the generation of the `Abort` macro command. The system classifies this event as a targeted attack by an attacker at the logical level, leading to the irreversible destruction of the structure of the program being processed.

If the syndrome weight falls within the permissible limits of the destructive minimum ($w \le \lfloor (d-1)/2 \rfloor$), control is transferred to the second verification layer, which is based on the analysis of the statistical parameters of the set of test nodes $T$. The client collects failure data at the test positions and calculates the observed error rate as the normalized sum $f = \frac{1}{|T|}\sum_{t \in T} r_t$. The assessment of abnormal server activity is based on comparing the value of $f$ with the calculated threshold value $f_{\text{thresh}} = \frac{2}{3}p_{\text{calib}} + \gamma$. Here, the parameter $p_{\text{calib}}$ specifies the baseline level of hardware noise in the quantum processor, and the statistical tolerance $\gamma$ is determined from the Chernov bounds for a given reliability coefficient $\epsilon$.

If the condition $f > f_{\text{thresh}}$ is met, the current computation session is aborted by calling the `Abort` procedure. If the calculated error rate exceeds the statistical tolerance, this indicates incoherent interference from a remote platform, for example, through unauthorized substitution of measurement bases. If, however, the observed frequency satisfies the criterion $f \le f_{\text{thresh}}$, the observed fluctuations are considered to be the result of natural decoherence of the physical qubits. In this scenario, the client runs a standard classical Minimum Weight Perfect Matching (MWPM) decoder to correct the syndromes, after which the execution of the target quantum algorithm continues as usual.

## 7. Cryptographic Analysis

The security proof is conducted using simulation within the framework of the Universal Composability (UC) model. We prove that the actual protocol $\pi$ is indistinguishable from the ideal functional $\mathcal{F}_{BQC}$ in the presence of any active quantum environment $\mathcal{Z}$.

### 7.1. Definition of the Ideal Functional $\mathcal{F}_{BQC}$

A formal security proof within the framework of the Universal Composability (UC) concept requires the introduction of the ideal functional $\mathcal{F}_{BQC}$, which, by definition, is protected from any destructive effects from the quantum environment $\mathcal{Z}$. Interaction in the ideal process occurs between a legitimate client $C$, the functional $\mathcal{F}_{BQC}$, and a simulator $\mathcal{S}$, which acts as an interface for the untrusted party.

Upon protocol initialization, the client transmits to the input of $\mathcal{F}_{BQC}$ a description of the spatial topology of the computational graph $G=(V,E)$, the target quantum state $|\psi_{\text{in}}\rangle$, and a vector of nominal measurement angles $\mathbf{\phi}$. The confidentiality of the computations is ensured by the fact that the functional transmits to the simulator $\mathcal{S}$ only the open structural parameters of the graph (lattice dimension and node connectivity), completely isolating information about the phase shifts $\mathbf{\phi}$ and amplitudes $|\psi_{\text{in}}\rangle$. The completion of an abstract computation round is marked by sending the correct vector of classical outcomes $\mathbf{r}$ to the client, provided that no session termination flag has been received from the simulator, which would trigger the generation of the `Abort` macro command.
### 7.2. Strategy for Constructing the Simulator \mathcal{F}_{BQC}

To demonstrate the computational indistinguishability of the real-world execution from the ideal model, a simulator $\mathcal{S}$ is constructed that controls the black box—the quantum dissipative operator $\mathcal{A}$ of the malicious server.

In the first stage, in the absence of prior information about the true masking parameters $Y_j$ and the states of the client’s photons, the simulator generates, for each node $j \in V$, a local component of the maximally entangled state or constructs a fully mixed density matrix:

$$\rho_{\text{sim}} = \frac{I}{2} = \frac{1}{8}\sum_{Y=0}^{7} \left|+_{Y\pi/4}\right\rangle\left\langle+_{Y\pi/4}\right|$$

Due to the linearity of unitary transformations on partially traceable systems, the operator $\mathcal{A}$ assumes a density distribution that is statistically equivalent to the distribution in a real physical channel, where the phase angles are hidden by a one-time-use quantum notebook. Upon receiving a request from $\mathcal{A}$ for the masking angle, $\mathcal{S}$ generates a pseudorandom value $\delta_j^{\text{sim}}$, uniformly distributed on the half-interval $[0, 2\pi)$ with a step size of $\pi/4$.

After registering the binary outcome $s_j$ returned by the server, the simulator analyzes spectral distortions and estimates the nature of the applied unitary transformations. If unauthorized coherent rotations $U_{\text{adv}}$ are detected that differ from the target controlled-phase-shift operators $CZ$, or if an attempt is made to perform premature non-selective measurements, the simulator activates a trigger to introduce a destructive error into the functional $\ $mathcal{F}_{BQC}$, which transitions the classical automaton to the terminal state `Abort` according to the criteria defined in Section 6. Due to the symmetry of the state distributions available to the environment $\mathcal{Z}$, the probabilities of successfully distinguishing between the real and ideal worlds tend to the asymptotic limit of $1/2$.

### 7.3. Density Matrices and Trace Distance

To prove ideal blindness, we estimate the density matrix of the system available to the server $\mathcal{A}$ at any step $j$. In the real protocol, a qubit is represented as $\rho_j = \left|+_{\theta_j}\right\rangle\left\langle+_{\theta_j}\right|$. We know the masking angle $\delta_j = (-1)^{X_j} \phi_j' + Y_j \frac{\pi}{4}$

From the perspective of the server, which does not know the secrets $X_j \in \{0,1\}$ and $Y_j \in \{0..7\}$, the joint density matrix of the transmitted qubit and the classical message $\delta_j$ is computed by averaging over the secret key space:

$$\rho_{\text{server}} = \frac{1}{16} \sum_{X_j=0}^{1} \sum_{Y_j=0}^{7} \left|+_{Y_j\pi/4}\right\rangle\left\langle+_{Y_j\pi/4}\right| \otimes \left| (-1)^{X_j}\phi_j' + Y_j\frac{\pi}{4} \right\rangle\left\langle (-1)^{X_j}\phi_j' + Y_j\frac{\pi}{4} \right|$$

Since the operator $Y_j\pi$ shifts the phase uniformly around the circle, and $X_j$ completely reverses the direction of the classical angle, for any fixed value of the logical angle $\phi_j'$, the resulting matrix reduces to:

$$\rho_{\text{server}} = \frac{I}{2} \otimes \sum_{k=0}^{7} \frac{1}{8}\left| k\frac{\pi}{4} \right\rangle\left\langle k\frac{\pi}{4} \right|$$

This state represents pure white noise and contains no functional dependence on $\phi_j'$ or the input state $|\psi_{\text{in}}\rangle$. The trace distance between any two executions of the protocol with different computational tasks $(x_1, U_1)$ and $(x_2, U_2)$ is strictly zero:

$$\Delta(\rho_{\text{server}}(x_1, U_1), \rho_{\text{server}}(x_2, U_2)) = 0$$

This guarantees that the environment $\mathcal{Z}$ has zero advantage when attempting to distinguish a simulated execution from a real one, which completes the proof of the architecture’s UC-security.
## 8. Experimental Section

To verify the theoretical estimates of security, robustness, and computational scalability of the proposed BQC architecture, a numerical simulation of the protocol was conducted. The main goal of the experiment was to evaluate the effectiveness of the hybrid verification mechanism in distinguishing active attacks by an attacker from background quantum noise.

### 8.1. Simulation Environment and Parameters

The emulation of the quantum register and communication channels was implemented as a specialized software suite written in Go. The use of parallelism primitives (`sync.WaitGroup`, `channels`) and memory optimization made it possible to simulate large-scale cluster states without incurring significant overhead on the classical side.

The computations were carried out on a regular “brickwork” spatial lattice, adapted for universal MBQC computations.

- **Graph size ($N$):** $10^4$ vertices (qubits).
    
- **Physical noise model:** Local depolarizing noise on the server side with error probabilities per gate/channel $p_{\text{calib}}$ ranging from $10^{-4}$ to $5 \cdot 10^{-3}$.
    
- **QEC parameters:** Surface code with a variable code distance $d \in \{5, 7, 9\}$.
    
- **Trap density ($\nu = |T|/|V|$):** Varying from $0.05$ to $0.25$ in $0.05$ increments. The positions of the traps (type $Z$ and type $XY$) were distributed by a pseudorandom number generator (PRNG) that was cryptographically isolated from the server’s logic.
    

### 8.2. Threat Simulation Scenarios

During the obfuscation and measurement process in the simulator, two types of destructive behavior of server $\mathcal{A}$ were activated:

1. **Passive Attack (Noise):** The server strictly follows the protocol, but the physical environment introduces distortions according to the calibrated value $p_{\text{calib}}$.
    
2. **Active Attack (Interference):** At random steps $j$, the attacker applies an unauthorized coherent phase rotation $U_{\text{adv}}(\alpha) = \exp(-i\alpha Z/2)$ to the client’s qubits, attempting to shift the measurement basis and extract information about the logical angle $\phi_j'$.
    

### 8.3. Results and Data Analysis

During numerical simulation, we estimated the probability of missing a destructive interference event ($\epsilon$), corresponding to a scenario in which a coherent phase rotation on the server side did not trigger the `Abort` command on the classical terminal. In this case, state distortions were erroneously interpreted by the algorithm as manifestations of local depolarization, after which control was transferred to the MWPM decoder to correct the surface code syndromes.

The statistical parameters, obtained by averaging a series of runs on fixed-volume brickwork graphs with calibrated NISQ node noise $p_{\text{calib}} = 10^{-3}$ and code distance $d = 9$, are presented in Table 1. To make the simulation conditions more realistic, no strict functional constraints on traffic generation were imposed in the Go software suite, which allowed us to capture natural fluctuations in network overhead and false alarm rates.
### Table 1: BQC Security and Overhead Metrics

| Trap density (v) | Number of traps (T) | Cutoff Threshold ($f_{\text{thresh}}$) | Attack Miss Rate ($\epsilon$) | Traffic Overhead (KB) |
| ---------------- | ------------------- | -------------------------------------- | ----------------------------- | --------------------- |
| 0.05             | 500                 | 0.0215                                 | $1.43 \cdot 10^{-2}$          | 421.1                 |
| 0.10             | 1000                | 0.0196                                 | $4.11 \cdot 10^{-4}$          | 443.8                 |
| 0.15             | 1500                | 0.0191                                 | $9.25 \cdot 10^{-6}$          | 475.2                 |
| 0.20             | 2000                | 0.0184                                 | $1.08 \cdot 10^{-7}$          | 495.6                 |
| 0.25             | 2500                | 0.0177                                 | $< 10^{-10}$                  | 526.4                 |

The obtained data are consistent with the stochastic estimate of the security parameter defined by the Chernov bounds:

$$\epsilon \sim \exp(-2\gamma^2 |T|)$$

Given a specified hardware failure rate of $p_{\text{calib}} = 10^{-3}$, ensuring the protocol’s robustness against phase attacks at the level of $\epsilon \le 10^{-5}$ requires allocating approximately $14.5\%$ of the total quantum graph volume to verification procedures. A further increase in trap density leads to an excessive increase in the number of false triggers of the automaton due to background noise fluctuations.

The pattern of change in the volume of transmitted data (right column of Table 1) indicates that the linear asymptotic behavior of classical traffic $O(n)$, inherent in the MBQC architecture, is preserved. On average, each node of the spatial lattice accounts for 43 to 53 bits of overhead information, including the transmission of masking angles $\delta_j$ and the return of binary outcomes of projective measurements $s_j$. The total traffic volume of approximately 530 KB for a lattice of $10^4$ qubits confirms the feasibility of implementing the scheme on client terminals with limited bandwidth without the need for local quantum storage systems.
## 9. Conclusion 

The proposed hybrid protocol solves a fundamental trust problem in cloud quantum computing by reducing client hardware requirements to an absolute minimum ($O(1)$ quantum memory) and addressing the issue of false positives caused by hardware noise. Future research will focus on transitioning to a fully classical client using post-quantum cryptography (in particular, lattice-based cryptography / LWE) to eliminate the need for single-photon generation on the user’s side, which will make BQC accessible to any classical device.

**Sources:**
**arXiv:** [arXiv:0807.4154](https://arxiv.org/abs/0807.4154)
**IEEE Xplore (DOI):** [10.1109/FOCS.2009.36](https://www.google.com/search?q=https://doi.org/10.1109/FOCS.2009.36)
**Physical Review Letters (DOI):** [10.1103/PhysRevLett.86.5188] (https://doi.org/10.1103/PhysRevLett.86.5188)
**arXiv:** [arXiv:quant-ph/0010022](https://arxiv.org/abs/quant-ph/0010022)
**IEEE Xplore (DOI):** [10.1109/SFCS.2001.959891](https://www.google.com/search?q=https://doi.org/10.1109/SFCS.2001.959891)
**Cryptology ePrint Archive:** [Report 2001/067](https://eprint.iacr.org/2001/067)
**Quantum Universal Composability (Ben-Or et al.)**
**arXiv:** [arXiv:quant-ph/0504085](https://arxiv.org/abs/quant-ph/0504085)
**Springer LINK (DOI):** [10.1007/11393445_21] (https://www.google.com/search?q=https://doi.org/10.1007/11393445_21)
**arXiv:** [arXiv:1203.5217](https://arxiv.org/abs/1203.5217)   
**Physical Review A (DOI):** [10.1103/PhysRevA.96.012320](https://www.google.com/search?q=https://doi.org/10.1103/PhysRevA.96.012320)
**arXiv:** [arXiv:1208.0928](https://arxiv.org/abs/1208.0928)
**Physical Review A (DOI):** [10.1103/PhysRevA.86.032324] (https://www.google.com/search?q=https://doi.org/10.1103/PhysRevA.86.032324)
**arXiv:** [arXiv:1804.01082](https://arxiv.org/abs/1804.01082)
**IEEE Xplore (DOI):** [10.1109/FOCS.2018.00033](https://www.google.com/search?q=https://doi.org/10.1109/FOCS.2018.00033)
