<h3>
  <a href="README.md">ENG</a> | 
  <a href="README-RU.md">RUS</a>
</h3>

# Technical Specifications: Project "Hydra-Core"
**Version:** 1.0 (Stable Concept / Adaptive Implementation)  
**Status:** Active Counter-Censorship Architecture Design  
**Stack:** Rust (Core/Logic), Kotlin (Android), Swift (iOS), JavaFX (Desktop), TOML/JSON (Dynamic Configs)  
**License:** MIT

---

## 1. General Concept & Philosophy
**Hydra-Core** is an intelligent, cross-platform framework designed to bypass **State-level DPI (Deep Packet Inspection)** and **ISP-level censorship nodes (e.g., National Firewall hardware)**.

**Key Innovation:** A paradigm shift from passive evasion to a strategy of **"Computational Asymmetry."** Instead of simple masking, Hydra-Core exploits the finite nature of hardware resources (SRAM/TCAM) and CPU cycles within DPI nodes. Utilizing a high-performance Rust core, the application identifies vulnerabilities in the censor’s packet reassembly and defragmentation algorithms. It forces the DPI to expend 100–1000x more computational power to analyze a session than the client spends generating it. The ultimate goal is to force the DPI into **Fail-Open mode** (where it ceases analysis and permits all traffic due to resource exhaustion).

---

## 2. System Architecture

### 2.1. Multi-Layer Abstraction
1.  **L3/L4 Adaptive Core (Rust):** Direct manipulation of raw packets via Raw Sockets/TUN.
    * **Jitter-Buffer Module:** Injects micro-delays to bypass hardware defragmentation timers.
    * Custom TCP/UDP stack implementation to generate invalid but "enticing" states for the DPI engine.
2.  **Strategy Hunter Engine (Rust):** Dynamic fuzzing of parameters (TTL, Split-position, Window Size).
    * Feedback Loop driven by RTT analysis and RST packet fingerprinting.
3.  **Orchestrator & FFI:** Lifecycle management of evasion strategies.
    * Anonymized synchronization of "winning" presets via the Hydra-Net gossip protocol.
4.  **Platform Wrappers:**
    * **Android:** VpnService + JNI (zero-copy performance).
    * **iOS:** NetworkExtension + PacketTunnelProvider.
    * **Desktop:** Wintun (Windows) / UTUN (macOS) / TUN (Linux).

---

## 3. Core Technical Specification (Rust)

### 3.1. Surgical Manipulation Module (DPI-Evasion)
Exploiting the logic of **TCP Reassembly**:
* **Lazy Fragmentation:** Splitting the SNI into tiny segments (e.g., 1–2 bytes) with artificial delays ($delay > 100ms$). This forces the DPI to either drop legitimate traffic (causing public outcry) or permit segments without full reassembly.
* **Overlapping Segments:** Sending TCP segments with overlapping Sequence Numbers; the first contains "garbage," while the second contains the payload. Flawed DPI implementations will analyze the garbage while the destination server accepts the valid data.
* **Multi-TTL Desync:** Injecting fake HTTP/TLS packets with precise TTL values that allow them to reach the DPI node but "die" (expire) before reaching the target server.

### 3.2. Advanced Stress Engine (Resource Exploitation)
Targeting hardware bottlenecks rather than raw bandwidth:
* **SRAM State Exhaustion:** Generating thousands of `uTLS` (Client Hello) sessions with valid modern browser fingerprints. This saturates the DPI's State Tables, triggering aggressive LIFO-drops that degrade the censor's overall filtering quality.
* **QUIC-Spin & Entropy Attack:**
    * Manipulating the `Spin Bit` in QUIC to disorient network quality monitoring algorithms.
    * **Adaptive Padding:** Dynamically resizing QUIC UDP packets (1200–1450 bytes) to mimic video streaming and bypass packet-length heuristics.
* **Zero-Window Attack:** Simulating client congestion via TCP Window Size = 0, forcing the DPI to hold the session context in memory for the maximum possible duration.

### 3.3. "Phantom Router" Model
Simulating a multi-device NAT environment to mask signatures:
* **Fingerprint Rotation:** Each parallel session uses a unique set of TCP options (MSS, SACK_PERM, TSopt), mimicking various OS environments (Windows, iPhone, Android).
* **TTL Randomization:** Randomizing TTL within a $\pm 3$ range to simulate complex internal network topologies.

---

## 4. "Strategy Hunter" Algorithm (Intellectual Evasion)

An adaptive module based on evolutionary search principles.

### 4.1. Feedback Loop & Detection
Differentiating between block types:
1.  **Active RST:** If the `Sequence Number` in an incoming RST does not match the expected window—it is a DPI injection. The Hunter measures the Delta RTT; if Delta < 10ms, the censor is physically "close" (at the ISP level).
2.  **Silent Drop:** Detected via a lack of ACKs after a series of retransmits.
3.  **Throttling:** Detected by comparing L4 RTT (SYN-ACK) vs. L7 RTT (TLS Finished).

### 4.2. "Gap" Discovery Method
1.  **Probe Phase:** Attempting a connection with `split_pos = 2`. On failure, the algorithm iterates through positions.
2.  **Desync Calibration:** Running a traceroute to the target host to determine the exact hop where the **State-level DPI box** resides. The system sets `fake_ttl = current_hop - 1`.
3.  **QUIC Migration:** If UDP packet loss exceeds 15%, the Hunter initiates `Connection Migration`, changing the port and CID to reset the DPI's analysis context.

### 4.3. Hydra-Net (Anonymous Exchange)
Discovered strategies are exchanged via **Probabilistic Gossiping**:
* Nodes share hash tables of the form `{ASN_ID: Strategy_Hash}`.
* Data is transmitted via steganographic channels (e.g., in request headers to trusted Beacon hosts) or public JSON repositories.

---

## 5. Platform Implementation Details

### 5.1. Mobile (Android/iOS)
* **Battery-Aware Stress:** The engine automatically throttles activity if battery level is < 20% or if the CPU overheats, shifting to a passive "Stealth" mode.
* **JNI Efficiency:** On Android, the TUN file descriptor is passed directly to Rust via `std::os::unix::io::FromRawFd`, eliminating buffer copying between the JVM and native code.

### 5.2. Desktop (Windows/Linux/macOS)
* **Wintun Layer:** Utilizing the fastest available Windows driver for throughput up to 10 Gbps.
* **Visual Debugger:** A JavaFX dashboard displaying the "Battle Map": estimated DPI load, current evasion strategy, and route health.

---

## 6. Roadmap (Milestones)

* **Phase 1: Foundation (0-2 months):** Rust core with `uTLS` support and basic defragmentation. CLI version release.
* **Phase 2: Tactical (2-4 months):** Strategy Hunter implementation and TTL calibration. Android VpnService integration.
* **Phase 3: Resilience (4-6 months):** QUIC-Stealth and Connection Migration modules. iOS NetworkExtension support.
* **Phase 4: Critical Mass (6+ months):** Hydra-Net launch for anonymous config sharing. Goal: 100k nodes to test the "Fail-Open" hypothesis on national segments.

---

## 7. Risks & Mitigation
1.  **Risk: Whitelisting (Default Deny).** *Mitigation:* Support for **Domain Fronting** and mimicry of approved protocols (tunneling within TLS sessions to trusted CDNs).
2.  **Risk: PPS Limiting (Packet Rate Caps).** *Mitigation:* Hunter adapts noise intensity to stay below the ISP's filtering threshold ("Last Mile Protection").

---

**Structure of the "QUIC-Stealth" config**
```json
{
  "protocol": "QUIC/UDP",
  "evasion_level": "surgical",
  "padding": "dynamic_range_1200_1450",
  "spin_bit": "chaos_mode",
  "migration_trigger": "loss_15pct",
  "fake_initials": 50
}
```

---

## Philosophy and Legal Disclaimer

**Conclusion:** Hydra-Core is more than an unblocking tool; it is an instrument designed to make censorship **financially unsustainable**. We shift the cost of blocking from civil society onto the budgets of censorship agencies by forcing their hardware to its physical limits.

*Disclaimer: Project Hydra-Core is created as a research tool for studying network resilience. The developers are not responsible for usage that violates local regulations. We maintain that **freedom of information is a fundamental human right**, and technical countermeasures against State-level DPI are a necessary form of digital self-defense.*

**Freedom is not a permission; it is the technical impossibility of a ban.**

---

### Appendix: Global Connectivity Context & Threats

To understand the necessity of **Hydra-Core**, one must consider the escalating complexity of state-sponsored Internet filtering in specific regions:

#### 1. Russia (The "Sovereign Internet" & TSPU)
The Russian regulator (**Roskomnadzor**) has deployed a nationwide network of **TSPU** (Technical Measures to Counter Threats) hardware. These are specialized DPI (Deep Packet Inspection) boxes installed directly at the ISP level, controlled centrally via the cloud. 
* **The Problem:** Unlike simple IP-blocking, TSPU performs **protocol-level throttling** (e.g., slowing down YouTube or Twitter to 128kbps) and uses SNI-filtering to drop encrypted handshakes.
* **The Goal of Hydra-Core:** To force these centralized nodes into a "fail-open" state by saturating their computational buffers.

#### 2. China (The Great Firewall - GFW)
The GFW is the world's most advanced censorship system. It utilizes **Active Probing** and **Machine Learning** to identify non-standard traffic.
* **The Problem:** Even if traffic is encrypted, the GFW detects "entropy anomalies" (traffic that looks too random). 
* **The Goal of Hydra-Core:** Our **Virtual Router Model** and **Entropy Attack** modules aim to blend user data with common traffic patterns, making the "cost of detection" too high for the GFW's classification clusters.

#### 3. North Korea (Kwangmyong & Total Isolation)
While North Korea mostly relies on a physical "air-gap" from the global web, its external connections are monitored by a single state gateway.
* **The Problem:** Extreme whitelist-only filtering where any unknown protocol is dropped instantly.
* **The Goal of Hydra-Core:** To test the limits of **TLS Handshake Flooding** as a means of maintaining a stable "hole" in highly restrictive, low-bandwidth state gateways.

### Glossary of Terms for International Readers
* **State-level DPI:** Hardware clusters used by governments to inspect, modify, or drop packets based on content, not just destination.
* **Fail-Open:** A security design where a system stops filtering and allows all traffic to pass if its resources (CPU/RAM) are exhausted.
* **SNI Filtering:** A technique used by censors to see which website you are visiting even before the encrypted connection is fully established.

---
