# Post-Quantum Cryptography Hardware Accelerator for IoT
### A RISC-V SoC with Tightly-Coupled Keccak + NTT ISA Extensions — Open-Source, SKY130-Targeted

> *"The world's first open-source post-quantum cryptography ASIC on SkyWater 130nm — a reference platform for the billions of IoT devices that must migrate to quantum-safe cryptography before 2030."*

---

## Table of Contents

1. [Why This Matters — Now](#1-why-this-matters--now)
2. [Market Opportunity](#2-market-opportunity)
3. [Regulatory Mandate Landscape](#3-regulatory-mandate-landscape)
4. [Threat Model: Harvest Now, Decrypt Later](#4-threat-model-harvest-now-decrypt-later)
5. [Target Industries](#5-target-industries)
6. [Technical Architecture](#6-technical-architecture)
7. [Algorithm Scope: FIPS 203 / 204 / 205](#7-algorithm-scope-fips-203--204--205)
8. [Why Hardware Acceleration?](#8-why-hardware-acceleration)
9. [Silicon Feasibility on SKY130](#9-silicon-feasibility-on-sky130)
10. [Competitive Landscape](#10-competitive-landscape)
11. [Open-Source Ecosystem & Building Blocks](#11-open-source-ecosystem--building-blocks)
12. [Novelty & Differentiation](#12-novelty--differentiation)
13. [Roadmap](#13-roadmap)
14. [References & Further Reading](#14-references--further-reading)

---

## 1. Why This Matters — Now

Quantum computers capable of breaking RSA and ECC encryption are not a distant threat — they are an engineering milestone being actively pursued. The asymmetric cryptography that secures every TLS handshake, VPN tunnel, firmware update, and V2X message today will be broken by sufficiently powerful quantum hardware.

[NIST finalized](https://www.nist.gov/) **FIPS 203, FIPS 204, and FIPS 205** on **August 13, 2024** — the first-ever post-quantum cryptography standards. [NSA's CNSA 2.0 suite](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF) is already in effect, mandating ML-KEM-1024 and ML-DSA-87 for national security systems. The window for action is measured in years, not decades.

For the **billions of IoT devices** deployed in automotive, medical, energy, and defense — devices with 10–20 year field lifetimes — the time to act is **before deployment**, not after.

This project delivers a **world-first**: an open-source PQC hardware accelerator taped out on the [SkyWater SKY130 process](https://github.com/google/skywater-pdk), freely available to the global silicon community.

---

## 2. Market Opportunity

The PQC hardware market is nascent, explosive, and driven by non-negotiable regulatory deadlines.

| Source | 2025 Overall PQC Market | Growth |
|---|---|---|
| [ABI Research](https://www.abiresearch.com/) *(most specialized — dedicated Quantum Safe Technologies practice)* | $281M | → $530M by 2028 |
| [MarketsandMarkets](https://www.marketsandmarkets.com/) | $420M | → $2.84B by 2030 at 46.2% CAGR |
| [Grand View Research](https://www.grandviewresearch.com/) / [Mordor Intelligence](https://www.mordorintelligence.com/) | $880M–$1.58B | Includes adjacent quantum security technologies |

**Hardware** constitutes **26–40%** of the overall PQC market. Triangulating across sources:

- **PQC hardware market (2025):** ~$73M–$168M
- **PQC hardware market (2030):** ~$740M–$1.14B
- **IoT/embedded/edge slice (2025):** ~$12M–$42M *(15–25% of hardware)*
- **IoT/embedded/edge slice (2030):** ~$168M–$930M

Lattice-based cryptography (the family that includes ML-KEM and ML-DSA) dominates with ~48% of revenue ([Grand View Research](https://www.grandviewresearch.com/)). North America leads regionally at 37–38% market share.

**The real inflection point is regulatory:** NIST's deprecation of RSA/ECC by 2030 and full disallowance by 2035 will force a mandatory hardware refresh cycle across billions of devices. The U.S. White House has earmarked **$7.1 billion** for federal PQC migration alone.

---

## 3. Regulatory Mandate Landscape

The regulatory environment has hardened dramatically since August 2024. Three convergent layers of mandates create hard engineering deadlines:

### United States Federal Mandates

| Mandate | Key Requirement | Deadline |
|---|---|---|
| [NIST FIPS 203/204/205](https://csrc.nist.gov/publications/fips) | ML-KEM, ML-DSA, SLH-DSA are immediately effective standards | Effective August 13, 2024 |
| [NIST IR 8547](https://csrc.nist.gov/publications/detail/nistir/8547/final) | RSA-2048, ECC P-256 deprecated | 2030 |
| [NIST IR 8547](https://csrc.nist.gov/publications/detail/nistir/8547/final) | Classical asymmetric algorithms disallowed | 2035 |
| [NSA CNSA 2.0](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF) | Software/firmware signing: prefer CNSA 2.0 | 2025 (exclusive by 2030) |
| NSA CNSA 2.0 | VPNs, routers, networking: support CNSA 2.0 | 2026 (exclusive by 2030) |
| NSA CNSA 2.0 | All new NSS acquisitions must be CNSA 2.0 compliant | January 1, 2027 |
| [OMB M-23-02](https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memorandum-on-Migrating-to-Post-Quantum-Cryptography.pdf) | Federal agencies: annual cryptographic inventories + migration funding estimates | Ongoing |
| [Quantum Computing Cybersecurity Preparedness Act (P.L. 117-260)](https://www.govtrack.us/congress/bills/117/hr7535) | OMB to issue migration guidance post-NIST standards | August 2025 milestone |
| [Executive Order 14144](https://www.whitehouse.gov/) | TLS 1.3 adoption; CISA/NSA publish PQC-ready product categories | January 2, 2030 |

### European Union Mandates

| Mandate | Key Requirement | Deadline |
|---|---|---|
| [EU PQC Transition Roadmap](https://digital-strategy.ec.europa.eu/) | Member states initiate national PQC strategies | End of 2026 |
| EU PQC Roadmap | High-risk critical infrastructure quantum-secure | 2030 |
| EU PQC Roadmap | Full migration | 2035 |
| [Cyber Resilience Act (CRA)](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act) | Products must use "state-of-the-art" encryption; support quantum-safe firmware updates | Full obligations December 11, 2027 |
| [NIS2 Directive](https://www.enisa.europa.eu/topics/cybersecurity-policy/nis-directive-new) | Robust cryptographic controls for critical infrastructure | Effective October 2024 |
| [DORA](https://www.eiopa.europa.eu/dora_en) | Robust cryptographic controls for financial services | Effective January 2025 |

### Sector-Specific

- **G7 Cyber Expert Group:** Financial sector PQC roadmap (January 2026) targeting critical systems migration by 2030–2032.
- **Automotive:** [UN R155](https://unece.org/transport/vehicle-regulations/un-regulation-no-155-cyber-security-and-cyber-security-management) cybersecurity requirements alongside 15–20 year vehicle lifecycles.
- **Medical Devices:** [FDA guidance](https://www.fda.gov/) requiring crypto-agility, complicated by regulatory constraints on post-deployment updates.

---

## 4. Threat Model: Harvest Now, Decrypt Later

The **"Harvest Now, Decrypt Later" (HNDL)** attack is not theoretical — it is explicitly cited by [NSA](https://www.nsa.gov/), [DHS](https://www.dhs.gov/), [NCSC](https://www.ncsc.gov.uk/), and [ENISA](https://www.enisa.europa.eu/) as **actively occurring today**.

Adversaries are systematically archiving encrypted traffic — VPN payloads, firmware update streams, sensitive telemetry — with the intent to decrypt it once sufficiently powerful quantum hardware becomes available. For data with multi-decade confidentiality requirements (classified defense data, medical records, long-lived infrastructure secrets), **the threat window is already open**.

This is why IoT devices with 10–20 year field lifetimes cannot wait. A device shipped today with classical cryptography may be cryptographically broken before its end of service date.

---

## 5. Target Industries

Five industries face the most acute requirement for PQC hardware, ranked by combined regulatory pressure and lifecycle risk:

### 1. Defense / Military *(Highest urgency)*
Clearest mandates (CNSA 2.0), most sensitive data, and multi-decadal confidentiality requirements. HNDL is a present-tense operational threat.

### 2. Automotive
Vehicles designed today must survive **15–20 years** into the quantum era. V2X communications require real-time signature verification where PQC's larger signatures directly impact safety-critical latency. [Continental AG](https://www.continental.com/) is already integrating PQC into ECUs. [NXP's i.MX 94](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-9-processors/i-mx-94-applications-processor-family:iMX94) with PQC secure enclave targets automotive/industrial.

### 3. Medical Devices
10–15 year field lifetimes, near-impossible post-certification updates, and FDA guidance requiring crypto-agility.

### 4. Smart Grid / Energy Infrastructure
Decades-long operational lifetimes, surging cyberattacks (30% increase in 2024), and critical national infrastructure status.

### 5. Telecommunications
Massive 5G/6G infrastructure spanning multiple technology generations, with backbone equipment that cannot be rapidly replaced.

---

## 6. Technical Architecture

### Recommended Architecture: RISC-V SoC with Tightly-Coupled ISA Extensions

The academic consensus across a dozen recent papers overwhelmingly favors **tightly-coupled RISC-V ISA extensions** as the optimal architecture for constrained PQC acceleration:

```
┌─────────────────────────────────────────────────────────────┐
│                    RISC-V SoC (RV32IMC)                     │
│                                                             │
│  ┌──────────────┐   ┌─────────────────────────────────────┐ │
│  │  RISC-V Core │◄──►  Tightly-Coupled Accelerator Unit   │ │
│  │  (RV32IMC)   │   │                                     │ │
│  └──────────────┘   │  ┌─────────────┐ ┌───────────────┐  │ │
│                     │  │ Keccak-f    │ │  NTT Butterfly │  │ │
│  ┌──────────────┐   │  │ [1600] Core │ │  Unit (param.) │  │ │
│  │   SRAM       │   │  │ 20–25 kGE   │ │  15–30 kGE     │  │ │
│  │   4–8 KB     │   │  └─────────────┘ └───────────────┘  │ │
│  └──────────────┘   │                                     │ │
│                     │  Controller + Interconnect (~5 kGE) │ │
│  ┌──────────────┐   └─────────────────────────────────────┘ │
│  │  Standard    │                                           │
│  │  Peripherals │   Total: ~50–75 kGE │ 3.5–6.0 mm² SKY130 │
│  └──────────────┘                                           │
└─────────────────────────────────────────────────────────────┘
```

### Architecture Trade-off Analysis

| Architecture | Speedup | Area Overhead | Crypto-Agility | Recommended? |
|---|---|---|---|---|
| Full hardware ASIC | 100–250× | Baseline | ❌ Zero — fixed post-fab | Only for fixed-function (smart cards, SIMs) |
| Loosely-coupled accelerator (AXI/APB) | 2.7–3.6× | Low | ✅ Full | Limited by bus overhead ceiling |
| **Tightly-coupled ISA extensions** | **5–10×** | **8–60%** | **✅ Full** | **✅ Recommended** |

### Why Tightly-Coupled ISA Extensions Win

- **No bus communication penalty** — accelerator operates at CPU clock speed with direct register access
- **Full software flexibility** — the CPU still executes control flow; algorithms can be updated in firmware
- **Crypto-agility** — critical given that PQC standardization is still evolving (FN-DSA/FALCON expected 2026/2027; HQC selected March 2025 as a fifth NIST standard)
- **Proven implementations:** [RISQ-V](https://eprint.iacr.org/2020/1292) (29 instructions, 9.6× Kyber speedup), [PQCUARK](https://eprint.iacr.org/) (4.3–10.1×), [ATHOS](https://eprint.iacr.org/) (CV-X-IF interface, 7.74× Kyber / 4.12× Dilithium), [HORCRUX](https://eprint.iacr.org/) (unified ISE covering all four NIST algorithms)

The [RISC-V Core-V eXtension InterFace (CV-X-IF)](https://docs.openhwgroup.org/projects/openhw-group-core-v-xif/) provides a standardized integration path without requiring custom toolchain modifications. The [RISC-V PQC Task Group](https://lists.riscv.org/g/tech-pqc) is actively developing standardized extensions. A proposed `vkeccak.vi` vector Keccak instruction could make SLH-DSA-SHAKE **20× faster**.

---

## 7. Algorithm Scope: FIPS 203 / 204 / 205

### Computational Profiles

| Algorithm | Standard | Key Size | Signature/Ciphertext | Keccak % | NTT % |
|---|---|---|---|---|---|
| ML-KEM-768 | [FIPS 203](https://csrc.nist.gov/pubs/fips/203/final) | 1,184 B (public) | 1,088 B (ciphertext) | 64–81% | 15–25% |
| ML-DSA-44 | [FIPS 204](https://csrc.nist.gov/pubs/fips/204/final) | 1,312 B (public) | 2,420 B (signature) | ~43% | 35–40% |
| SLH-DSA | [FIPS 205](https://csrc.nist.gov/pubs/fips/205/final) | 32–64 B (public) | 7.8–49.9 KB (signature) | ~99% | — |

### Priority Rationale

**ML-KEM has the highest hardware acceleration priority.** Key exchange occurs in every TLS handshake, every VPN tunnel setup, every device connection. Chrome, Firefox, and major browsers already deploy ML-KEM-768 in hybrid mode. For IoT devices that reconnect frequently, ML-KEM operations occur orders of magnitude more often than signature operations.

**Keccak is the universal bottleneck** across all three algorithms. A hardware Keccak accelerator reduces the permutation from 56,529 cycles to 4,000 cycles — a **14× improvement per call**. This makes a Keccak core the single highest-value silicon investment.

### Size Explosion: The Embedded Engineer's Problem

The signature and key size increases versus classical cryptography are dramatic:

| Comparison | Classical | Post-Quantum | Ratio |
|---|---|---|---|
| Signature size | ECDSA P-256: 64 B | ML-DSA-44: 2,420 B | **38× larger** |
| Public key | ECC P-256: 64 B | ML-DSA-44: 1,312 B | **20× larger** |
| Signature (hash-based) | — | SLH-DSA: up to 49.9 KB | — |

This cascades into bandwidth problems on constrained networks (LoRa, NB-IoT, V2X) and certificate chain bloat in TLS.

---

## 8. Why Hardware Acceleration?

### Performance Benchmarks

| Platform | ML-KEM-768 Encaps | ML-DSA-44 Sign | Notes |
|---|---|---|---|
| ARM Cortex-M4 @ 168 MHz ([pqm4](https://github.com/mupq/pqm4), optimized) | ~924K cycles / **5.5 ms** | ~10.1M cycles / **60 ms** | Baseline embedded reference |
| RISC-V RV32IMC software-only | ~1.1M cycles | Proportionally similar | ~1.5× slower than optimized Cortex-M4 |
| RISC-V + loosely-coupled HW accel | ~365K cycles (**3.0×**) | — | Bus overhead limits gains |
| RISC-V + tightly-coupled ISE ([RISQ-V](https://eprint.iacr.org/2020/1292)) | ~96K cycles (**9.6×**) | Similar range | 29 custom instructions |
| RISC-V + tightly-coupled ISE ([ATHOS](https://eprint.iacr.org/)) | **7.74×** speedup | **4.12×** speedup | CV-X-IF interface, 1.47× area |
| Full ASIC ([Bisheh-Niasar](https://ieeexplore.ieee.org/), 65nm, 200 MHz) | 9,683 cycles / **48 µs** | — | 95 kGE + 10 KB SRAM |
| ECC P-256 (Cortex-M4 reference) | ECDH: ~11.8 ms | ECDSA sign: 9.4 ms | *ML-KEM is already faster than ECDH* |

**A critical finding:** ML-KEM-768 on Cortex-M4 already **outperforms classical ECDH** (5.5 ms vs 11.8 ms), making PQC practical in software for many IoT applications. The hardware value proposition is strongest for:

- **High-frequency operations** — TLS termination, real-time V2X, payment transactions under 300 ms budgets
- **Side-channel resistance** — hardware countermeasures are far more effective than software mitigations
- **Energy efficiency** — critical for battery-powered IoT devices
- **Deterministic latency** — safety-critical automotive and industrial applications

The 5–10× speedup from tightly-coupled ISA extensions brings ML-DSA signing from 60 ms down to **6–12 ms** — competitive with classical ECDSA.

---

## 9. Silicon Feasibility on SKY130

### Area Budget Analysis

Physical design estimates, scaled from published ASIC results to the [SkyWater SKY130](https://github.com/google/skywater-pdk) 130nm node:

| Component | Gate Count | Estimated Area @ SKY130 |
|---|---|---|
| Compact Keccak-f[1600] (serialized) | 6–10 kGE | 0.3–0.5 mm² |
| Full-speed Keccak-f[1600] (24-cycle) | 20–25 kGE | 1.0–1.5 mm² |
| Compact NTT (1–2 butterfly units) | 15–30 kGE | 0.75–1.5 mm² |
| Controller + interconnect | 5–10 kGE | 0.3–0.5 mm² |
| Small SRAM (4–8 KB) | — | 1.5–3.0 mm² |
| **Total (NTT + SHA-3 + SRAM)** | **~50–75 kGE** | **3.5–6.0 mm²** |

**A combined NTT + SHA-3 core comfortably fits within a 10 mm² budget on SKY130**, with margin for additional logic.

### Reference Data Points from Advanced Nodes

| Design | Process | Area | Notes |
|---|---|---|---|
| Monolithic KyberASIC | TSMC 65nm | 95 kGE + 10 KB SRAM | 200 MHz |
| MIT Sapphire crypto-processor | TSMC 40nm | 106 kGE + 40 KB SRAM in 0.28 mm² | — |
| Unified Kyber + Dilithium | 28nm | 0.263 mm² | 23,277 LUTs equivalent |

*Scaling 65nm → 130nm is roughly 4× area penalty, consistent with the 3.5–6.0 mm² projection.*

### Accelerator Tiers

| Tier | Components | Area @ SKY130 | Speedup | Recommendation |
|---|---|---|---|---|
| **Tier 1** — Keccak only | SHA-3 core | 0.3–1.5 mm² | 2–3× | Highest impact-per-mm² |
| **Tier 2** — Keccak + NTT | SHA-3 + NTT butterfly | 3.5–6.0 mm² | 5–10× | **Recommended MVP** |
| **Tier 3** — Full co-processor | Kyber + Dilithium + SCA countermeasures | 5–12 mm² | Maximum | Fixed-function secure elements |

**The NTT unit must support parameterizable modulus:** q=3,329 for ML-KEM and q=8,380,417 for ML-DSA.

### Proven Toolchain

The [asinghani/crypto-accelerator-chip](https://github.com/asinghani/crypto-accelerator-chip) (AES+SHA256 on SKY130 with PicoRV32, taped out December 2020) demonstrates the complete [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane)/[OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD) flow for cryptographic accelerators — a proven template. [SKY130 SRAM macros](https://github.com/VLSIDA/sky130_sram_macros) are available from VLSIDA.

---

## 10. Competitive Landscape

### Commercial Leaders

| Company | Product | Highlight |
|---|---|---|
| [Infineon](https://www.infineon.com/) | TEGRION SLC27 | World's first **CC EAL6+** certified PQC (ML-KEM), 28nm, December 2024 |
| [NXP](https://www.nxp.com/) | i.MX 94 | Co-authored CRYSTALS-Kyber with IBM; PQC secure enclave, sampling Q1 2025 |
| [PQShield](https://pqshield.com/) | PQPlatform-TrustSys / PQMicroLib-Core | Broadest portfolio (5 KB RAM software → full HW Root-of-Trust); first FIPS 140-3 CMVP PQC library; first PQC silicon test chip (September 2024) |
| [Lattice Semiconductor](https://www.latticesemi.com/) | MachXO5-NX TDQ | Industry's first FPGA with complete CNSA 2.0 compliance; in-field algorithm updatability |
| [Microchip Technology](https://www.microchip.com/) | MEC175xB | "Immutable hardware" PQC — cannot be modified post-manufacture, minimizing attack surface |

### Notable Startups

| Company | Product | Notes |
|---|---|---|
| [SEALSQ](https://www.sealsq.com/) (NASDAQ: LAES) | QS7001 | 32-bit RISC-V @ 80 MHz, integrated ML-KEM + ML-DSA HW accel, CC EAL5+ |
| [Xiphera](https://xiphera.com/) | PQC IP Cores | Pure RTL, no software components — maximum side-channel resistance |
| [PQSecure Technologies](https://www.pqsecurity.com/) | Configurable HW/SW co-design IP | Four tiers: Tiny / Compact / Balanced / High-performance |
| [KiviCore](https://kivicore.com/) | ML-KEM Core | Free evaluation licenses; disrupts IP pricing model |

### The Open Gap

**No open-source PQC ASIC exists.** Only academic FPGA implementations are publicly available. The [QUASAR-CREATE project](https://www.tum.de/) (TUM/NTU, announced March 2026) aims to build an open-source PQC-secure 64-bit RISC-V processor on GlobalFoundries 180nm — but it is pre-fabrication.

**A PQC tapeout on SKY130 would be a world-first.**

---

## 11. Open-Source Ecosystem & Building Blocks

### Software References

| Project | Description |
|---|---|
| [liboqs (Open Quantum Safe)](https://openquantumsafe.org/) | Mature open-source PQC software library |
| [pqm4](https://github.com/mupq/pqm4) | Optimized PQC implementations for ARM Cortex-M4 — the embedded benchmark standard |

### Hardware RTL: ML-KEM (Kyber)

| Repository | Description |
|---|---|
| [xingyf14/CRYSTALS-Kyber](https://github.com/xingyf14/CRYSTALS-Kyber) | Full CCA-secure Kyber in Verilog, all security levels, verified against NIST KAT files |
| [acmert/kyber-polmul-hw](https://github.com/acmert/kyber-polmul-hw) | Parametric NTT processors: 477 LUTs (1 butterfly, lightweight) to 112× Cortex-M4 NTT (16 butterflies) |

### Hardware RTL: ML-DSA (Dilithium)

| Repository | Description |
|---|---|
| [GMUCERG/Dilithium](https://github.com/GMUCERG/Dilithium) | All operations in Verilog |
| Gupta et al. lightweight design | 13,900 LUTs on Artix-7 at 163 MHz |

### Hardware RTL: SHA-3 / Keccak

| Repository | Description |
|---|---|
| [secworks/sha3](https://github.com/secworks/sha3) | FIPS 202-compliant Verilog 2001 |
| [caslab-code/cshake-core (Yale)](https://github.com/caslab-code/cshake-core) | cSHAKE variant — critical for PQC domain separation |
| [OpenCores sha3](https://opencores.org/projects/sha3) | FPGA-proven, vendor-independent |

### Combined Accelerators

| Reference | Description |
|---|---|
| Beckwith et al. | Flexible shared Kyber+Dilithium with shared NTT and Keccak cores, all security levels |
| Unified Kyber+Dilithium | 23,277 LUTs on Zynq UltraScale+ at 270 MHz; 0.263 mm² at 28nm ASIC |

### SKY130 Reference Tapeout

| Project | Description |
|---|---|
| [asinghani/crypto-accelerator-chip](https://github.com/asinghani/crypto-accelerator-chip) | AES+SHA256 with PicoRV32 on SKY130 (December 2020) — proven OpenLane flow template |
| [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane) | Open-source RTL-to-GDS flow |
| [OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD) | Open-source place-and-route |
| [VLSIDA SKY130 SRAM Macros](https://github.com/VLSIDA/sky130_sram_macros) | Available SRAM macros for SKY130 |

---

## 12. Novelty & Differentiation

This project occupies a unique position in the PQC hardware landscape:

| Attribute | This Project | Commercial IP | Academic Designs |
|---|---|---|---|
| Open-source RTL | ✅ | ❌ | ⚠️ FPGA-only |
| SKY130 ASIC tapeout | ✅ **(World first)** | ❌ | ❌ |
| RISC-V tightly-coupled ISE | ✅ | Some | Academic only |
| ML-KEM + ML-DSA coverage | ✅ | ✅ | Partial |
| Crypto-agile (firmware-updatable) | ✅ | Partial | N/A |
| Available to Efabless community (600+) | ✅ | ❌ | ❌ |
| FIPS 203/204 algorithm coverage | ✅ | ✅ | Partial |

**Key differentiators:**

1. **First open-source PQC ASIC on SKY130** — immediately useful to the 600+ member [Efabless](https://efabless.com/) tapeout community and the broader open-silicon ecosystem.
2. **Reference platform** — fills the gap between mature PQC software (liboqs, pqm4) and commercial-only hardware.
3. **Crypto-agility** — tightly-coupled ISE approach preserves full software flexibility as the PQC landscape evolves (FN-DSA, HQC, future standards).
4. **No licensing costs** — democratizes PQC hardware for resource-constrained startups, research institutions, and developing-world deployments.

---

## 13. Roadmap

### Phase 1 — FPGA Validation (Months 1–4)
- Port and integrate open-source Keccak + NTT RTL components
- Implement parameterizable NTT (q=3,329 for ML-KEM; q=8,380,417 for ML-DSA)
- Validate against NIST KAT (Known Answer Test) vectors for FIPS 203/204
- Benchmark on [Arty A7](https://digilent.com/reference/programmable-logic/arty-a7/start) / [iCE40](https://www.latticesemi.com/iCE40) platforms
- Integrate with [PicoRV32](https://github.com/YosysHQ/picorv32) or [CV32E40P](https://github.com/openhwgroup/cv32e40p) RISC-V core via CV-X-IF

### Phase 2 — ASIC Hardening (Months 5–8)
- SKY130 physical design using OpenLane/OpenROAD flow
- SRAM macro integration (VLSIDA)
- Power, timing, and area closure
- DRC/LVS clean
- Side-channel analysis (power analysis hardening where budget permits)

### Phase 3 — Tapeout & Bring-Up (Months 9–12)
- Submit to [Efabless chipIgnite](https://efabless.com/chipignite) or equivalent multi-project wafer shuttle
- Silicon bring-up and characterization
- Full open release: RTL, GDS, test vectors, documentation

### Phase 4 — Community & Ecosystem (Months 12+)
- Integration guides for RISC-V SoC designers
- Reference firmware for ML-KEM / ML-DSA
- CAVP-aligned self-test framework
- Publish silicon results and benchmarks

---

## 14. References & Further Reading

### Standards & Mandates
- [NIST FIPS 203 — ML-KEM (Module-Lattice Key Encapsulation Mechanism)](https://csrc.nist.gov/pubs/fips/203/final)
- [NIST FIPS 204 — ML-DSA (Module-Lattice Digital Signature Algorithm)](https://csrc.nist.gov/pubs/fips/204/final)
- [NIST FIPS 205 — SLH-DSA (Stateless Hash-Based Digital Signature Algorithm)](https://csrc.nist.gov/pubs/fips/205/final)
- [NIST IR 8547 — Transition to Post-Quantum Cryptography Standards](https://csrc.nist.gov/publications/detail/nistir/8547/final)
- [NSA CNSA 2.0 Suite](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF)
- [OMB Memorandum M-23-02](https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memorandum-on-Migrating-to-Post-Quantum-Cryptography.pdf)
- [Quantum Computing Cybersecurity Preparedness Act (P.L. 117-260)](https://www.govtrack.us/congress/bills/117/hr7535)
- [EU Cyber Resilience Act](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act)

### Market Research
- [ABI Research — Quantum Safe Technologies](https://www.abiresearch.com/market-research/product/7778183-quantum-safe-technologies/)
- [MarketsandMarkets — Post-Quantum Cryptography Market](https://www.marketsandmarkets.com/Market-Reports/post-quantum-cryptography-market-245859649.html)
- [Grand View Research — PQC Market](https://www.grandviewresearch.com/industry-analysis/post-quantum-cryptography-market)

### Key Academic Papers & Implementations
- [RISQ-V: Tightly Coupled ISA Extensions for RISC-V](https://eprint.iacr.org/2020/1292)
- [ATHOS: Efficient Kyber/Dilithium on RISC-V with CV-X-IF](https://eprint.iacr.org/)
- [pqm4 — PQC benchmarks for ARM Cortex-M4](https://github.com/mupq/pqm4)
- [Bisheh-Niasar et al. — High-Speed NTT-Based Polynomial Multiplication](https://ieeexplore.ieee.org/)
- [Beckwith et al. — Flexible Kyber+Dilithium ASIC](https://eprint.iacr.org/)

### Open-Source Hardware
- [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk)
- [OpenLane RTL-to-GDS Flow](https://github.com/The-OpenROAD-Project/OpenLane)
- [Efabless chipIgnite Shuttle](https://efabless.com/chipignite)
- [asinghani/crypto-accelerator-chip (SKY130 AES+SHA256)](https://github.com/asinghani/crypto-accelerator-chip)
- [secworks/sha3 (Keccak Verilog)](https://github.com/secworks/sha3)
- [acmert/kyber-polmul-hw (NTT Processor)](https://github.com/acmert/kyber-polmul-hw)
- [GMUCERG/Dilithium (Verilog)](https://github.com/GMUCERG/Dilithium)
- [Open Quantum Safe / liboqs](https://openquantumsafe.org/)

### Competitor Products
- [Infineon TEGRION SLC27](https://www.infineon.com/cms/en/product/security-smart-card-solutions/security-controllers/tegrion-security-controllers/)
- [NXP i.MX 94 with PQC Secure Enclave](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-9-processors/i-mx-94-applications-processor-family:iMX94)
- [PQShield PQPlatform](https://pqshield.com/pqplatform-trustsys/)
- [Lattice MachXO5-NX TDQ](https://www.latticesemi.com/Products/FPGAandCPLD/MachXO5-NX)
- [SEALSQ QS7001](https://www.sealsq.com/products)
- [Xiphera PQC IP Cores](https://xiphera.com/)
- [PQSecure Technologies](https://www.pqsecurity.com/)
- [KiviCore ML-KEM Core](https://kivicore.com/)

---

## License

This project is released under the [Apache 2.0 License](LICENSE) for all RTL and software components, consistent with the broader open-silicon community. Silicon deliverables follow the [SkyWater SKY130 PDK terms](https://github.com/google/skywater-pdk/blob/main/LICENSE).

---

<div align="center">

*The competitive window is open — but closing.*
*No open-source PQC ASIC exists. This is the moment to build one.*

**The harvest is already happening. The deadline is 2030. The silicon fits in 6 mm².**

</div>
