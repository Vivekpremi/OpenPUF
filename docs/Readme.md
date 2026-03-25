<div align="center">

# 🔐 PQC-RISC-V: Post-Quantum Cryptography Hardware Accelerator for IoT

### *A Tightly-Coupled RISC-V SoC with Keccak-f[1600] + NTT ISA Extensions — World-First Open-Source PQC ASIC on SkyWater SKY130*

---

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Process: SKY130](https://img.shields.io/badge/Process-SkyWater%20130nm-green)](https://github.com/google/skywater-pdk)
[![Standard: FIPS 203](https://img.shields.io/badge/FIPS-203%20%7C%20204%20%7C%20205-orange)](https://csrc.nist.gov/publications/fips)
[![Architecture: RISC-V](https://img.shields.io/badge/ISA-RISC--V%20RV32IMC-red)](https://riscv.org/)

</div>

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [The Problem We Solve](#2-the-problem-we-solve)
3. [Market Opportunity](#3-market-opportunity)
4. [Regulatory Mandate Timeline](#4-regulatory-mandate-timeline)
5. [Threat Model — Harvest Now, Decrypt Later](#5-threat-model--harvest-now-decrypt-later)
6. [Why Hardware? The Performance Case](#6-why-hardware-the-performance-case)
7. [Algorithm Deep Dive — ML-KEM, ML-DSA, SLH-DSA](#7-algorithm-deep-dive--ml-kem-ml-dsa-slh-dsa)
8. [Core Architecture](#8-core-architecture)
9. [Keccak-f\[1600\] — The Universal Bottleneck](#9-keccak-f1600--the-universal-bottleneck)
10. [NTT Accelerator Design](#10-ntt-accelerator-design)
11. [Silicon Feasibility on SKY130](#11-silicon-feasibility-on-sky130)
12. [Competitive Landscape](#12-competitive-landscape)
13. [Open-Source Building Blocks](#13-open-source-building-blocks)
14. [Novelty and Why This Matters](#14-novelty-and-why-this-matters)
15. [Project Roadmap](#15-project-roadmap)
16. [References](#16-references)

---

## 1. Executive Summary

> *"Post-Quantum Secure IoT SoC using Caravel Open Silicon Platform"*

Quantum computers are advancing rapidly. When they arrive at scale, every asymmetric cryptographic system in use today — RSA, ECDH, ECDSA — will be broken by Shor's algorithm. [NIST finalized FIPS 203, 204, and 205](https://csrc.nist.gov/publications/fips) on **August 13, 2024**, delivering the world's first post-quantum cryptography (PQC) standards. [NSA's CNSA 2.0 suite](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF) is already in effect.

**This project aims to deliver an open-source PQC-secure SoC** — a RISC-V–based SoC with integrated Keccak-f[1600] and NTT accelerators — targeting IoT applications. It will be designed for the [SkyWater SKY130 130nm open process](https://github.com/google/skywater-pdk) on Carvel framework to ensure reproducibility and facilitate further development within the open-source community.


### At a Glance

| Attribute | Value |
|---|---|
| **Target Process** | SkyWater SKY130 (130nm open PDK) |
| **Platform** | Caravel SoC Framework |
| **Accelerated Primitives** | Keccak-f[1600] (SHAKE-128/256, SHA3-256/512) + NTT |
| **Covered Standards** | FIPS 203 (ML-KEM), FIPS 204 (ML-DSA) |
| **Speedup over Software** | 5–10× |
| **Novelty** | A PQC-Secure SoC targeting IoT applications |

---

## 2. The Problem We Solve

### Classical cryptography has an expiry date

Every TLS handshake, VPN tunnel, firmware update, and V2X message today is secured by RSA or ECC. Shor's algorithm, running on a sufficiently powerful quantum computer, can break these in polynomial time. The cryptographic foundation of the internet — and of embedded devices everywhere — will collapse.

### IoT devices face a unique crisis

Unlike servers that can be patched overnight, **IoT devices have field lifetimes of 10–20 years**:

- An automotive ECU designed today will still be operational in 2044
- A medical implant certified in 2025 may not receive a firmware update for its entire service life
- Smart grid controllers operate for decades with no practical path to key rotation

If these devices are not shipped with post-quantum cryptography, they will be cryptographically broken before they are decommissioned. 

### Challenges in Deploying ML-DSA and ML-KEM on Constrained Devices

**1. Compute overhead** — ML-DSA-44 signing takes ~60 ms on a Cortex-M4 at 168 MHz, compared to 9.4 ms for ECDSA. Without acceleration, PQC is too slow for latency-critical applications like V2X safety messaging.

**2. Size explosion** — ML-DSA-44 signatures are 2,420 bytes versus 64 bytes for ECDSA P-256 — a **38× increase**. This creates severe bandwidth problems on LoRa, NB-IoT, and constrained V2X channels, and causes certificate chain bloat in TLS.

**3. Memory pressure** — ML-KEM requires careful SRAM management on constrained microcontrollers.

### The open-source gap

No production-ready, open-source PQC hardware core exists for ASIC fabrication. The entire open-source PQC hardware ecosystem focuces more on **academic FPGA prototypes** — complete industry ready prototypes targeting IoT devices are still missing. This project fills that gap.

---

## 3. Market Opportunity

The PQC hardware market is still emerging but rapidly accelerating, with governments worldwide placing strong emphasis on the development and deployment of quantum-resistant systems.

### Market Sizing

| Source | 2025 Overall PQC Market | Projection |
|---|---|---|
| [ABI Research](https://www.abiresearch.com/) *(dedicated Quantum Safe Technologies practice)* | **$281M** | → $530M by 2028 |
| [MarketsandMarkets](https://www.marketsandmarkets.com/) | $420M | → $2.84B by 2030 at **46.2% CAGR** |
| [Grand View Research](https://www.grandviewresearch.com/) / [Mordor Intelligence](https://www.mordorintelligence.com/) | $880M–$1.58B | Includes adjacent quantum security technologies |

Hardware constitutes **26–40%** of the total PQC market. Triangulating the IoT/embedded/edge slice specifically:

```
PQC Hardware Market 2025:   $73M  – $168M
PQC Hardware Market 2030:   $740M – $1.14B

IoT / Edge Slice 2025:      $12M  – $42M   (15–25% of hardware)
IoT / Edge Slice 2030:      $168M – $930M
```

**Lattice-based cryptography** — the family covering ML-KEM and ML-DSA — dominates at ~48% of hardware revenue. North America leads regionally at 37–38% market share.

## 4. Regulatory Mandate Timeline

Three layers of mandates converge to create hard engineering deadlines:

### United States Federal Mandates

| Year | Mandate | Requirement |
|---|---|---|
| **Aug 2024** | [NIST FIPS 203/204/205](https://csrc.nist.gov/publications/fips) | ML-KEM, ML-DSA, SLH-DSA — **immediately effective** |
| **Nov 2024** | [NIST IR 8547](https://csrc.nist.gov/publications/detail/nistir/8547/final) | Deprecation roadmap formally established |
| **2025** | [NSA CNSA 2.0](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF) | Prefer CNSA 2.0 for software/firmware signing |
| **Jan 2027** | NSA CNSA 2.0 | All new NSS acquisitions **must** be CNSA 2.0 compliant |
| **2030** | [NIST IR 8547](https://csrc.nist.gov/publications/detail/nistir/8547/final) | RSA-2048, ECC P-256 **deprecated** |
| **2035** | NIST IR 8547 | Classical asymmetric algorithms **disallowed** |

NSA CNSA 2.0 mandates **ML-KEM-1024 and ML-DSA-87** exclusively — at the highest security level only. The phased transition timeline by domain:

```
Software/firmware signing:     Prefer by 2025  →  Exclusive by 2030
Web browsers/servers/cloud:    Support by 2025  →  Exclusive by 2033
VPNs / Routers / Networking:   Support by 2026  →  Exclusive by 2030
Operating systems:             Support by 2027  →  Exclusive by 2033
Constrained/niche equipment:   Support by 2030  →  Exclusive by 2033
```

Additional U.S. mandates: [OMB M-23-02](https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memorandum-on-Migrating-to-Post-Quantum-Cryptography.pdf) requires annual cryptographic inventories from all federal agencies. [Executive Order 14144](https://www.whitehouse.gov/) requires CISA/NSA to publish PQC-ready product categories. The [Quantum Computing Cybersecurity Preparedness Act (P.L. 117-260)](https://www.govtrack.us/congress/bills/117/hr7535) mandated OMB migration guidance after NIST standards publication.

### European Union Mandates

| Mandate | Requirement | Deadline |
|---|---|---|
| [EU PQC Transition Roadmap](https://digital-strategy.ec.europa.eu/) | All member states initiate national PQC strategies | End of **2026** |
| EU PQC Roadmap | High-risk critical infrastructure quantum-secure | **2030** |
| EU PQC Roadmap | Full migration complete | **2035** |
| [Cyber Resilience Act (CRA)](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act) | Products must use "state-of-the-art" encryption; quantum-safe firmware updates mandatory | Full obligations **Dec 2027** |
| [NIS2 Directive](https://www.enisa.europa.eu/topics/cybersecurity-policy/nis-directive-new) | Robust cryptographic controls for critical infrastructure | In effect **Oct 2024** |
| [DORA](https://www.eiopa.europa.eu/dora_en) | Robust cryptographic controls for financial services | In effect **Jan 2025** |

### Sector-Specific Pressure

- **G7 Cyber Expert Group** — Financial sector PQC roadmap (January 2026): critical systems migration by 2030–2032
- **Automotive** — [UN R155](https://unece.org/transport/vehicle-regulations/un-regulation-no-155-cyber-security-and-cyber-security-management) cybersecurity requirements alongside 15–20 year vehicle lifecycles
- **Medical Devices** — [FDA crypto-agility guidance](https://www.fda.gov/) with regulatory constraints making post-deployment updates extremely difficult

---

## 5. Threat Model — Harvest Now, Decrypt Later

> *"The threat is not coming. It is already here."*

The **Harvest Now, Decrypt Later (HNDL)** attack does not require a quantum computer today. Adversaries are systematically **archiving encrypted traffic right now** — firmware update streams, VPN payloads, sensitive device telemetry — with intent to decrypt once quantum hardware matures.

This is explicitly cited as **actively occurring** by [NSA](https://www.nsa.gov/), [DHS](https://www.dhs.gov/), [NCSC](https://www.ncsc.gov.uk/), and [ENISA](https://www.enisa.europa.eu/).

For data with multi-decade confidentiality requirements — classified defense communications, medical records, long-lived infrastructure credentials — **the threat window is already open today**. A device shipped with classical cryptography in 2025 may have its communications decrypted retroactively in 2035.

This is why IoT devices with 10–20 year field lifetimes cannot wait. The cryptographic decisions made at design time are permanent.

---

## 6. Why Hardware? The Performance Case

### Benchmark Comparison Across Platforms

The following table aggregates results from peer-reviewed FPGA and ASIC implementations:

| Platform | ML-KEM-768 Encaps | ML-DSA-44 Sign | Notes |
|---|---|---|---|
| ARM Cortex-M4 @ 168 MHz ([pqm4](https://github.com/mupq/pqm4)) | ~924K cycles / **5.5 ms** | ~10.1M cycles / **60 ms** | Baseline embedded reference |
| RISC-V RV32IMC (software only) | ~1.1M cycles | Similar ratio | ~1.5× slower than Cortex-M4 |
| RISC-V + loosely-coupled accelerator | ~365K cycles (**3.0×**) | — | Bus overhead caps gain at ~3× |
| RISC-V + tightly-coupled ISE ([RISQ-V](https://tches.iacr.org/index.php/TCHES/article/view/8739)) | ~96K cycles (**9.6×**) | Similar range | 29 custom instructions, 1.6× area |
| RISC-V + tightly-coupled ISE ([ATHOS](https://eprint.iacr.org/)) | **7.74×** speedup | **4.12×** speedup | CV-X-IF interface, 1.47× area |
| **Nguyen et al. ML-KEM ASIC (180nm, 93 MHz)** | **KeyGen: 1.8 µs** | — | 160 kGE, 7.37 kB SRAM, **6.4 µJ total** |
| **Truong et al. ML-DSA FPGA (VUS+, 300 MHz)** | — | **Sign avg: 97.4 µs (level 5)** | ATP 1.27–2.58× better than SOTA |
| ECC P-256 reference (Cortex-M4) | ECDH: ~11.8 ms | ECDSA: 9.4 ms | *ML-KEM in SW is already faster than ECDH* |

**Key finding from Nguyen et al. (UEC Tokyo, IEEE Access 2025):** Their 180nm ML-KEM ASIC achieves ATP improvements of **1.5× to 115×** over prior ASIC solutions using only **7.37 kB SRAM** — directly demonstrating that area-efficient PQC silicon is achievable within IoT constraints.

**Key finding from Truong et al. (Inha University, IEEE Access 2024):** Their ML-DSA FPGA achieves the lowest latency among all published FPGA implementations — **1.27–2.58× better area-time product** — using a pipelined NTT and a split dual-Keccak hash architecture for SHAKE-128 and SHAKE-256.

### When Hardware Becomes Essential

Software-only PQC is sufficient for infrequent operations. Hardware becomes critical when:

- **High-frequency operations** — TLS termination at the edge, real-time V2X safety messages, payment transactions under 300 ms budgets
- **Side-channel resistance** — Constant-time hardware execution and masking countermeasures are far more robust than software alternatives
- **Energy efficiency** — The Nguyen et al. ML-KEM ASIC consumes just **6.4 µJ** per full KeyGen + Encaps + Decaps at 1.8V, 93 MHz
- **Deterministic latency** — Safety-critical automotive and industrial applications require guaranteed worst-case timing bounds

The 5–10× speedup from tightly-coupled ISA extensions brings ML-DSA signing from 60 ms down to **6–12 ms** — directly competitive with classical ECDSA.

---

## 7. Algorithm Deep Dive — ML-KEM, ML-DSA

### 7.1 ML-KEM — FIPS 203 (Key Encapsulation Mechanism)

[ML-KEM](https://csrc.nist.gov/pubs/fips/203/final), standardized from CRYSTALS-Kyber, is the primary algorithm for general encryption and key establishment. The fundamental computational tasks within ML-KEM involve **generating randomness polynomials** and performing **polynomial arithmetic**. Randomness generation relies on uniform sampling or centered binomial distribution, driven by pseudorandom bytes derived from the **Secure Hash Algorithm 3 (SHA3) standard**, specifically the **Keccakf [1600] permutation**. Polynomial arithmetic is efficiently executed using the **Number Theoretic Transform (NTT)**, which reduces the complexity of polynomial multiplication to a quasi-linear O(n×log n), where n is the polynomial degree.

**Computational profile on Cortex-M4:**
```
Keccak / SHAKE (SHAKE-128, SHAKE-256, SHA3):  64–81% of total ML-KEM time
NTT / INTT:                                   15–25% of total ML-KEM time
```
### 7.2 ML-DSA — FIPS 204 (Digital Signature Algorithm)

[ML-DSA](https://csrc.nist.gov/pubs/fips/204/final), is a digital signature scheme based on CRYSTALS Dilithium, a member of the Cryptographic Suite for Algebraic Lattices (CRYSTALS) and a digital signature scheme based on the ‘‘Fiat-Shamir with Aborts’’ approach. Unlike Kyber and Falcon,  Dilithium uses uniform sampling rather than discrete Gaussian distribution for secret randomness generation. The major computation tasks here include sampling of short random polynomials, performing polynomial arithimatic and executing challenge response generation via hashing. Randomness generation rely heavily on SHA3 and the polynomial arithmetic can be done very efficiently by using NTT.

**Computational profile on Cortex-M4:**
```
Keccak / SHAKE:  ~43% of ML-DSA execution time
NTT / INTT:      ~35–40% of ML-DSA execution time
```

## 8. Core Architecture

### SoC Block Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                         PQC-RISC-V SoC                               │
│                                                                      │
│  ┌──────────────────┐    ┌─────────────────────────────────────────┐ │
│  │  RISC-V Core     │◄──►│      Tightly-Coupled Accelerator Unit   │ │
│  │                  │    │                                         │ │
│  │                  │    │  ┌───────────────────┐ ┌─────────────┐  │ │
│  │                  │    │  │  Keccak-f[1600]   │ │  NTT Unit   │  │ │
│  │                  │    │  │                   │ │             │  │ │
│  │                  │    │  │  Rate-configurable│ │  q=3329     │  │ │
│  └──────────────────┘    │  │  SHAKE-128/256    │ │  q=8380417  │  │ │
│                          │  │  SHA3-256/512     │ │  (param.)   │  │ │
│  ┌──────────────────┐    │  │  20–25 kGE        │ │  15–30 kGE  │  │ │
│  │  SRAM Macros     │    │  └───────────────────┘ └─────────────┘  │ │
│  │  (VLSIDA SKY130) │    │                                         │ │
│  │  4–8 KB          │    │  Controller + Interconnect (~5–10 kGE)  │ │
│  └──────────────────┘    └─────────────────────────────────────────┘ │
│                                                                      │
│  ┌──────────────────┐    Total Logic: ~50–75 kGE                    │
│  │  GPIO / SPI /    │    SRAM: 1.5–3.0 mm²                         │
│  │  UART peripherals│    Estimated Die: 3.5–6.0 mm² @ SKY130       │
│  └──────────────────┘                                               │
└──────────────────────────────────────────────────────────────────────┘

```
// this need to be changed
## Accelerator Integration Strategy

This project proposes the integration of dedicated post-quantum cryptographic (PQC) accelerators within the Caravel framework to enable a quantum-secure SoC while addressing the computational limitations of embedded processors.

The design will incorporate hardware accelerators for Keccak-f[1600] and Number Theoretic Transform (NTT) within the Caravel user project area. These accelerators will be connected to the RISC-V core via the Wishbone bus as memory-mapped peripherals, enabling seamless interaction with the processor.

The primary objective is to offload computationally intensive operations—such as hashing and polynomial arithmetic—from the CPU to specialized hardware units. These operations form the core bottlenecks in PQC algorithms like ML-KEM and ML-DSA.

By adopting this approach, the system is expected to:

Significantly reduce execution latency of PQC workloads
Improve overall system throughput
Lower energy consumption on resource-constrained platforms

This integration will transform the baseline Caravel platform into a PQC-secure SoC, capable of efficiently supporting quantum-resistant cryptographic primitives, and will demonstrate a practical pathway for deploying PQC in IoT and edge devices.


## 9. Keccak-f[1600] — The Universal Bottleneck

### Why One Core Handles Everything

The Keccak-f[1600] permutation is the single shared primitive underlying all hash and XOF operations in both ML-KEM and ML-DSA. Every matrix expansion, every seed derivation, every challenge hash — all go through Keccak-f[1600] with different rate/padding configurations.

**A hardware Keccak accelerator reduces the permutation from tens of thousands of cycles to a few thousand cycles.** Combined with NTT acceleration, total speedup reaches 7–8× for ML-KEM and 4–6× for ML-DSA.

All four functions (SHA3-256/SHA3-512/SHAKE-128/SHAKE-256) share a **single Keccak-f[1600] permutation core** — only rate, padding rule, and output length differ. One well-designed hardware core handles the complete hash requirement for all three finalized NIST standards.

### Keccak State and Round Structure

The Keccak state is a **5×5 array of 64-bit lanes** = 1,600-bit total state. The permutation applies 24 rounds of five step mappings in sequence: **θ** (column parity mixing), **ρ** (bitwise rotation), **π** (lane permutation), **χ** (nonlinear substitution), **ι** (round constant XOR). The pseudocode for keccak can be studied from [here](https://example.com).

<figure>
  <img src="https://opentitan.org/book/hw/ip/kmac/doc/sha3-padding.svg](https://opentitan.org/book/hw/ip/kmac/doc/keccak-round.svg" alt="diagram">
  <figcaption></figcaption>
</figure>
<figure>
  <a href="https://opentitan.org/book/hw/ip/kmac/doc/sha3-padding.svg" target="_blank">
    <img src="https://opentitan.org/book/hw/ip/kmac/doc/sha3-padding.svg" alt="diagram">
  </a>
</figure>

Keccak round logic has two phases inside. Theta, Rho, Pi functions are executed at the 1st phase. Chi and Iota functions run at the 2nd phase. The Keccak round logic stores the intermediate state after processing the 1st phase. The stored values are then fed into the 2nd phase computing the Chi and Iota functions. The Chi function leverages first-order [Domain-Oriented Masking](https://eprint.iacr.org/2017/395.pdf) (DOM) to deter SCA attacks. Processing a Keccak_f (1600 bit state) takes a total of 96 cycles (24 rounds X 4 cycles/round) including the 1st and 2nd phases. The 1st phase completes in one cycle but the second phase takes 3 cycles to complete, since no. of DOM multipliers were compromised to save area and hardware at the cost of clock cycles. 

### Padding
Padding logic supports **SHA3/SHAKE algorithms**. All these share similiar datapath except the last part added next to the end of the message. SHA3 adds **2'b10** and SHAKE adds **4'b1111** and then follows the padding. This module talks to Keccak round logic with a more memory-like interface. The interface has an additional address signal on top of the valid, ready, and data signals.

<figure>
  <img src="https://opentitan.org/book/hw/ip/kmac/doc/sha3-padding.svg" alt="diagram">
  <figcaption></figcaption>
</figure>

The hashing process begins when the software issues the start command to CMD register . 
<!-- 
If cSHAKE is enabled, the padding logic expands the prefix value into a block size. The block size is determined by a register whose value can be updated by the software when the hashing engine is in idle state. The block on the basis of register value is shown in the table below.

<img width="590" height="205" alt="image" src="https://github.com/user-attachments/assets/8b8fe064-2255-4ed8-8997-05f585771eff" />

The expanded prefix value is transmitted to the Keccak round logic. After sending the block size, the padding logic triggers the Keccak round logic to run a full 24 rounds.
If the mode is not cSHAKE, the padding logic accepts the incoming message bitstream and forward the data to the Keccak round logic in a block granularity. The padding logic controls the data flow and makes the Keccak logic to run after sending a block size. 
--> The padding logic, after receiving the Process command, appends proper ending bits with respect to the mode SHA3/SHAKE. The logic writes 0 up to the block size to the Keccak round logic then ends with 1 at the end of the block .

<figure>
  <img src="https://opentitan.org/book/hw/ip/kmac/doc/sha3-padding-fsm.svg" alt="diagram">
  <figcaption></figcaption>
</figure>

After the Keccak round completes the last block, the padding logic asserts a signal to notify the software. The signal generates the keccak_done interrupt. The software is now able to read the digest(hash output) in Keccak State memory(1600 bit memory) region. The software completes the operation by issuing Done command after reading the digest. The padding logic clears internal variables and goes back to Idle state.

### Keccak State Access

After the Keccak round completes the hashing operation, the contents of the Keccak state contain the digest value. The software can access the 1600 bit of the Keccak state directly through the window of the output SHA3/SHAKE register. 
The Keccak state is valid only after the sponge absorbing process is completed, 0 otherwise. This ensures that the logic does not expose the secret key XORed with the keccak_f results of the prefix to the software. 


### How does a sponge look?

The sponge construction is a cryptographic framework that transforms a fixed-size permutation, such as Keccak-f[1600], into a variable-input, variable-output function suitable for hashing and pseudorandom generation.
The internal state has a fixed width b=1600 bits and is divided into two regions:
  1. The rate r, which is the portion of the state exposed to input and output.
  2. The capacity c, which remains hidden and determines the security level.

The block diagram below explains how a sponge works. It shows the complete flow, the input message is first padded then XORed with the current state and then sent to keccak round block.
<figure>
  <img src="https://arxiv.org/html/2508.20653v1/x1.png" alt="diagram">
  <figcaption></figcaption>
</figure>

### OpenTitan Reference Implementation

The [OpenTitan KMAC block](https://github.com/lowRISC/opentitan/tree/master/hw/ip/kmac) (taped out on TSMC 40nm via Google) provides a silicon-proven Keccak-f[1600] with **two-share DOM masking** for side-channel resistance. The core module [`keccak_round.sv`](https://github.com/lowRISC/opentitan/blob/master/hw/ip/kmac/rtl/keccak_round.sv) is an excellent architectural reference.

---

## 10. NTT Accelerator Design

### The Number Theoretic Transform

The NTT reduces polynomial multiplication from O(n²) to O(n log n) by exploiting the ring structure:

```
c = INTT( NTT(a) ∘ NTT(b) )
```

Both ML-KEM and ML-DSA use the ring **R_q = Z_q[x] / (x^256 + 1)** with n = 256, but require **different modular arithmetic**:

| Algorithm | Modulus q | 2n-th Root of Unity | Reduction Method |
|---|---|---|---|
| ML-KEM (FIPS 203) | **3,329** | ζ = 17 | Barrett reduction |
| ML-DSA (FIPS 204) | **8,380,417** | φ₂ₙ = 1,753 | Specialized reduction (2²³ ≡ 2¹³ − 1) |

The NTT uses **negative-wrapped convolution (NWC)**, representing each degree-256 polynomial as 128 degree-one residues modulo quadratic factors. This structure is inherently parallel and suited for hardware pipelining.

### Butterfly Unit

The fundamental primitive is the butterfly unit (BU), supporting both Cooley-Tukey (CT) forward NTT and Gentleman-Sande (GS) inverse NTT through a configurable multiplexer:

```
CT butterfly:  (A, B) ← (A + Bω,  A − Bω)   mod q
GS butterfly:  (A, B) ← (A + B,   (A − B)ω)  mod q
```

One configurable BU handles both NTT and INTT — and can be reconfigured for pointwise multiplication (PWM) — reducing total hardware cost.

**Key optimization:** The technique by Zhang et al. eliminates the n⁻¹ (mod q) post-INTT multiplication by pre-processing twiddle factors and incorporating divide-by-2 directly into the addition unit.

### Modular Reduction for ML-DSA

For ML-DSA's modulus q = 8,380,417 = 2²³ − 2¹³ + 1, a specialized reduction exploits this relation:

```
s[45:0] ≡ s̄[45:23] + {s[42:33], 10'b0, s̄[45:43]}
         + {s[32:23], s̄[45:33]} + 2¹³s[45:43] + z + m
```

Where m = 50,331,648. Result falls in (−q, 3q), which simplifies final correction hardware significantly. This avoids general Montgomery or Barrett reduction and is more area-efficient for this specific modulus.

### Two-Mode Flexible Arithmetic Module

From Truong et al.'s ML-DSA design, the arithmetic module supports two operation modes controlled by a mode signal:

**Mode 1 — Fully Pipelined (8 PEs, for KeyGen and Verify):**
```
8 butterfly units (BU1–BU8), one per NTT layer for n=256
Stall delay:     153 clock cycles (CC) from input to first output
Throughput:      128 CC per polynomial NTT/INTT
Parallelism:     8 coefficients per cycle
```

**Mode 2 — Folding Transform (4+4 PEs, for Sign):**
```
Module split into two independent 4-PE units
Each PE handles two adjacent NTT layers via time-slicing (folding transform)
Delay:           281 CC, throughput 256 CC per polynomial
One unit:        computes vector y and w (lines 6–8 of Sign algorithm)
Other unit:      all remaining Sign computations
```

This dual-mode design allows the Sign operation (which needs two parallel polynomial streams) to execute efficiently without doubling total hardware.

### Twiddle Factor Storage Optimization

Three twiddle factor arrays are normally required — ζⁱ (NTT), ζ⁻ⁱ (INTT), ζ^(2·BitRev7(i)+1) (PWM) — totaling 1.5×n storage. From Nguyen et al. (2025), both derived arrays can be computed from ζⁱ on-the-fly:

```
ζ⁻ⁱ         ≡ q − ζ^(n−i)                               (one subtractor)
ζ^(2·BitRev7(2k+1)+1) ≡ q − ζ^(2·BitRev7(2k)+1)         (same subtractor)
```

This **reduces twiddle factor ROM by 3×** — a direct saving of n/2 × 12-bit words for ML-KEM-512.

### Measured NTT Performance

From Nguyen et al. ML-KEM ASIC (180nm, 93 MHz, 1.8V):

```
NTT/INTT per polynomial:    224 clock cycles
PWM between two polynomials: 128 clock cycles
ML-KEM-512  KeyGen/Encaps/Decaps: 10.6 / 13.6 / 21.3 µs  (FPGA @ 169 MHz)
ML-KEM-768  KeyGen/Encaps/Decaps: 18.3 / 21.3 / 33.2 µs
ML-KEM-1024 KeyGen/Encaps/Decaps: 26.6 / 30.2 / 45.0 µs
```

---

## 11. Silicon Feasibility on SKY130

### Area Budget Analysis

Physical design estimates scaled from published ASIC results to [SkyWater SKY130](https://github.com/google/skywater-pdk) 130nm:

| Component | Gate Count | Area @ SKY130 (130nm) |
|---|---|---|
| Compact Keccak-f[1600] (serialized) | 6–10 kGE | 0.3–0.5 mm² |
| Full-speed Keccak-f[1600] (24-cycle pipelined) | 20–25 kGE | 1.0–1.5 mm² |
| Parameterizable NTT (1–2 butterfly units) | 15–30 kGE | 0.75–1.5 mm² |
| Controller + Interconnect | 5–10 kGE | 0.3–0.5 mm² |
| SRAM 4–8 KB ([VLSIDA macros](https://github.com/VLSIDA/sky130_sram_macros)) | — | 1.5–3.0 mm² |
| **Total MVP (NTT + Keccak + SRAM)** | **~50–75 kGE** | **3.5–6.0 mm²** |

A combined NTT + SHA-3 core comfortably fits within a **10 mm² budget**, with margin for RISC-V core, control logic, and GPIO.

### Calibration Against Fabricated Designs

| Design | Process | Logic | SRAM | Area | Frequency |
|---|---|---|---|---|---|
| **Nguyen et al. ML-KEM (2025)** | **180nm CMOS** | **160 kGE** | **7.37 kB** | **3.47 mm²** | **93 MHz** |
| Truong et al. ML-DSA (FPGA→ASIC est.) | 28nm ref | ~50k LUT-eq | ~2 BRAM | 0.263 mm² | 300 MHz |
| Bisheh-Niasar KyberASIC | TSMC 65nm | 95 kGE | 10 KB | ~0.5 mm² | 200 MHz |
| MIT Sapphire crypto-processor | TSMC 40nm | 106 kGE | 40 KB | 0.28 mm² | — |
| Asinghani AES+SHA256 | **SKY130 130nm** | ~15 kGE | — | ~1.5 mm² | ~50 MHz |

Scaling rule: 65nm → 130nm applies ~4× area penalty, consistent with the 3.5–6.0 mm² projection. The Nguyen et al. 180nm design at 3.47 mm² for 160 kGE is the closest direct reference — a similar gate count on SKY130 targets a comparable footprint.

### Accelerator Tiers

| Tier | Contents | Area @ SKY130 | Speedup | Recommendation |
|---|---|---|---|---|
| **Tier 1** | Keccak-f[1600] only | 0.3–1.5 mm² | 2–3× | Highest impact/mm² — handles 64–81% of ML-KEM bottleneck alone |
| **Tier 2** | Keccak + parameterizable NTT | 3.5–6.0 mm² | 5–10× | **Recommended MVP** — covers ML-KEM and ML-DSA |
| **Tier 3** | Full PQC co-processor + SCA countermeasures | 5–12 mm² | Maximum | Fixed-function secure elements |

> The NTT unit **must** support parameterizable modulus — q = 3,329 for ML-KEM and q = 8,380,417 for ML-DSA. These require different modular reduction hardware.

### Proven SKY130 Toolchain

The [asinghani/crypto-accelerator-chip](https://github.com/asinghani/crypto-accelerator-chip) — AES+SHA256 on SKY130 with PicoRV32, taped out December 2020 — proves the complete [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane)/[OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD) RTL-to-GDS flow for cryptographic accelerators. This is the direct template for our tapeout.

---

## 12. Competitive Landscape

### Commercial Leaders in PQC Silicon

| Company | Product | Key Achievement | Status |
|---|---|---|---|
| [Infineon](https://www.infineon.com/cms/en/product/security-smart-card-solutions/security-controllers/tegrion-security-controllers/) | TEGRION SLC27 | World's first **CC EAL6+ certified PQC** (ML-KEM), 28nm | ✅ Shipping (Dec 2024) |
| [NXP](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-9-processors/i-mx-94-applications-processor-family:iMX94) | i.MX 94 | Co-authored CRYSTALS-Kyber with IBM; PQC secure enclave | ✅ Sampling (Q1 2025) |
| [PQShield](https://pqshield.com/pqplatform-trustsys/) | PQPlatform-TrustSys | First FIPS 140-3 CMVP PQC library; first PQC silicon test chip (Sep 2024) | ✅ Available |
| [Lattice Semiconductor](https://www.latticesemi.com/Products/FPGAandCPLD/MachXO5-NX) | MachXO5-NX TDQ | First FPGA with complete **CNSA 2.0 compliance**; in-field updatability | ✅ Shipping |
| [Microchip Technology](https://www.microchip.com/) | MEC175xB | "Immutable hardware" PQC — unmodifiable post-manufacture | ✅ Shipping |

### Notable Startups

| Company | Product | Differentiation |
|---|---|---|
| [SEALSQ](https://www.sealsq.com/) (NASDAQ: LAES) | QS7001 | 32-bit RISC-V @ 80 MHz, ML-KEM + ML-DSA HW accel, CC EAL5+ |
| [Xiphera](https://xiphera.com/) | PQC IP Cores | Pure RTL, no software components — maximum SCA resistance |
| [PQSecure Technologies](https://www.pqsecurity.com/) | Configurable HW/SW | Four tiers: Tiny / Compact / Balanced / High-performance |
| [KiviCore](https://kivicore.com/) | ML-KEM Core | Free evaluation licenses; aggressively low IP pricing |

### The Open-Source Vacuum

No open-source PQC ASIC exists. The [QUASAR-CREATE project](https://www.tum.de/) (TUM/NTU, announced March 2026) targets GlobalFoundries 180nm but is pre-fabrication and pre-submission. A SKY130 tapeout would be a **world-first** — immediately available to the 600+ member [Efabless chipIgnite](https://efabless.com/chipignite) community.

---

## 15. Project Roadmap

### Phase 1 — Integration of Open-Source Accelerators

- Integrate existing open-source **NTT** and **Keccak-f[1600]** RTL implementations into the design framework  
- Adapt interfaces for compatibility with the system architecture (e.g., Wishbone-based integration)  

---

### Phase 2 — Optimization of Accelerators

- Re-optimize integrated NTT and Keccak engines for the target use case  
- Focus on:
  - Latency reduction  
  - Area efficiency  
  - Improved dataflow and memory usage  
  - Better coupling with the processor  

---

### Phase 3 — RTL Verification

- Perform functional verification using:
  - **cocotb** for testbench-driven validation  
  - **Verilator** for cycle-accurate simulation  
- Apply **formal verification** using **SymbiYosys** for correctness and corner-case coverage  

---

### Phase 4 — System-Level Validation

- Validate the optimized design at the system level  
- Ensure correct interaction between CPU and accelerators  
- Verify end-to-end operation of target cryptographic workloads  

---

### Phase 5 — RTL-to-GDS Flow

- Convert RTL to synthesis-ready format  
- Execute full ASIC flow using **OpenROAD/OpenLane**, including:
  - Synthesis  
  - Floorplanning  
  - Placement and routing  
  - Timing closure  
  - DRC/LVS verification  

---

---

## 16. References

### NIST Standards

- [FIPS 203 — ML-KEM](https://csrc.nist.gov/pubs/fips/203/final)
- [FIPS 204 — ML-DSA](https://csrc.nist.gov/pubs/fips/204/final)
- [FIPS PUB 202 — SHA-3 / Keccak](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf)
- [NIST IR 8547 — PQC Transition Roadmap](https://csrc.nist.gov/publications/detail/nistir/8547/final)

### Key Academic Papers (Primary Sources for This Proposal)

- **Truong, Duong, Lee (2024)** — "Efficient Low-Latency Hardware Architecture for Module-Lattice-Based Digital Signature Standard." *IEEE Access*, Vol. 12. DOI: [10.1109/ACCESS.2024.3370470](https://doi.org/10.1109/ACCESS.2024.3370470). Inha University, South Korea. *(ML-DSA FPGA — pipelined NTT + dual Keccak, best ATP 1.27–2.58× over SOTA)*

- **Nguyen et al. (2025)** — "An Area-Time Efficient Hardware Architecture for ML-KEM Post-Quantum Cryptography Standard." *IEEE Access*, Vol. 13. DOI: [10.1109/ACCESS.2025.3579379](https://doi.org/10.1109/ACCESS.2025.3579379). University of Electro-Communications, Tokyo. *(ML-KEM ASIC @ 180nm — 160 kGE, 7.37 kB SRAM, 6.4 µJ, ATP 1.5–115× better than prior ASICs)*

- **Fritzmann et al. (2020)** — "RISQ-V: Tightly Coupled RISC-V Accelerators for Post-Quantum Cryptography." *IACR TCHES* Vol. 2020 No. 4. [Link](https://tches.iacr.org/index.php/TCHES/article/view/8739)

- **Beckwith et al. (2021)** — "High-Performance Hardware Implementation of CRYSTALS-Dilithium." *ICFPT 2021*

- **Aikata et al. (2023)** — "KaLi: A Crystal for Post-Quantum Security Using Kyber and Dilithium." *IEEE TCAS-I* Vol. 70. DOI: [10.1109/TCSI.2022.3218641](https://doi.org/10.1109/TCSI.2022.3218641)

- **Banerjee et al. (2019)** — "Sapphire: A Configurable Crypto-Processor for Post-Quantum Lattice-Based Protocols." *IACR TCHES*

### Policy and Regulatory

- [NSA CNSA 2.0](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF)
- [OMB Memorandum M-23-02](https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memorandum-on-Migrating-to-Post-Quantum-Cryptography.pdf)
- [Quantum Computing Cybersecurity Preparedness Act (P.L. 117-260)](https://www.govtrack.us/congress/bills/117/hr7535)
- [EU Cyber Resilience Act](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act)
- [CRYSTALS-Kyber Algorithm Spec v3.1](https://pq-crystals.org/kyber/data/kyber-specification-round3-20210131.pdf)
- [CRYSTALS-Dilithium Algorithm Spec v3.1](https://pq-crystals.org/dilithium/data/dilithium-specification-round3-20210208.pdf)

### Open-Source Hardware and Toolchain

- [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk)
- [OpenLane RTL-to-GDS](https://github.com/The-OpenROAD-Project/OpenLane)
- [OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD)
- [Efabless chipIgnite](https://efabless.com/chipignite)
- [asinghani/crypto-accelerator-chip (SKY130 AES+SHA256)](https://github.com/asinghani/crypto-accelerator-chip)
- [secworks/sha3](https://github.com/secworks/sha3)
- [caslab-code/cshake-core](https://github.com/caslab-code/cshake-core)
- [xingyf14/CRYSTALS-Kyber](https://github.com/xingyf14/CRYSTALS-Kyber)
- [acmert/kyber-polmul-hw](https://github.com/acmert/kyber-polmul-hw)
- [GMUCERG/Dilithium](https://github.com/GMUCERG/Dilithium)
- [VLSIDA SKY130 SRAM Macros](https://github.com/VLSIDA/sky130_sram_macros)
- [OpenTitan KMAC Block](https://github.com/lowRISC/opentitan/tree/master/hw/ip/kmac)
- [CV32E40P RISC-V Core](https://github.com/openhwgroup/cv32e40p)
- [Open Quantum Safe / liboqs](https://openquantumsafe.org/)
- [pqm4 — PQC for ARM Cortex-M4](https://github.com/mupq/pqm4)
- [Keccak Team VHDL Reference](https://keccak.team/hardware.html)

---

## License

This project is released under the [Apache 2.0 License](LICENSE) for all RTL and software components, consistent with the open-silicon community standard. Silicon deliverables follow the [SkyWater SKY130 PDK terms](https://github.com/google/skywater-pdk/blob/main/LICENSE).

---

<div align="center">

---

*The harvest is already happening. The deadline is 2030.*
*The silicon fits in 6 mm². The toolchain is proven. The standards are final.*

**The only missing piece is an open-source tapeout. This project delivers it.**

---

</div>
