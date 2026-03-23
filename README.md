<h3>
  <a href="README.md">ENG</a> | 
  <a href="README-RU.md">RUS</a>
</h3>

# Technical Specifications: Project "Hydra-Core"
**Version:** 1.0 (Draft)  
**Status:** Conceptual Design  
**Stack:** Rust (Core), Kotlin/Java (Android/Desktop), Swift (iOS/macOS), JSON (Configs)  
**License:** MIT

---

## 1. General Concept Overview
**Hydra-Core** is a cross-platform library and suite of applications designed to bypass **state-level DPI (Deep Packet Inspection)** and **ISP-level censorship systems**.

**Key Innovation:** Unlike passive bypass tools, Hydra-Core utilizes an **Active Resource Exhaustion** strategy. The application simulates the behavior of a high-load home router, generating a mix of legitimate user traffic and "garbage" sessions. These sessions force **government censorship nodes** to perform expensive computational operations, leading to their throttling or complete failure (fail-open).

---

## 2. System Architecture

### 2.1. Abstraction Layers
The project is divided into three layers:
1.  **L3/L4 Core (Rust):** Low-level packet processing, defragmentation logic, and noise generator.
2.  **Orchestrator (Rust + FFI):** Logic for managing JSON configs, link quality monitoring (Health-checks), and strategy selection.
3.  **Platform Wrapper (Java/Kotlin/Swift):** OS-level VPN interface, UI, and application lifecycle.

### 2.2. Network Interaction
The application creates a virtual network interface (**TUN/TAP**). All device traffic is routed into the Rust core.
* **Inbound:** Raw packets captured from OS applications.
* **Processing:** The core analyzes the packet (L3/L4), applies evasion modifications (Split, Desync, Fake DNS), and injects "parasitic" traffic.
* **Outbound:** The modified stream is transmitted via standard OS sockets to bypass **ISP filtering boxes**.

---

## 3. Core Technical Specification (Rust)

### 3.1. Packet Manipulation Module (DPI-Evasion)
Implementation of advanced evasion logic in memory-safe Rust:
* **TCP Segment Splitting:** Fragmenting the SNI request into parts (e.g., 1 byte + remainder) to prevent the **State DPI** from reassembling the header in time.
* **Window Size Manipulation:** Setting an extremely small window size to force the censor's hardware to work harder to parse the traffic.
* **HTTP/TLS Desync:** Sending fake packets with incorrect checksums or expired TTLs. These are ignored by the destination server but are processed by **censorship boxes**, desynchronizing their state machine.

### 3.2. Stress Engine (Load Generator)
The core component. To minimize the impact on user $bandwidth$, we utilize **CPU-intensive** attacks against the **National Firewall infrastructure**:
* **TLS Handshake Flood:** Generating thousands of short sessions with valid TLS fingerprints (Firefox/Chrome) that terminate immediately after the *Client Hello*. The censorship node is forced to store the state of each session in its memory.
* **QUIC Churn:** Generating UDP packets simulating the QUIC protocol. Encrypted QUIC is computationally difficult for **State-level DPI** to analyze; a mass influx of these sessions causes classification algorithm overloads.
* **Entropy Attack:** Sending packets with high entropy (appearing as encrypted traffic) to force the censor to apply heavy heuristics for protocol identification.

### 3.3. "Virtual Router" Model
To mask its signature against behavioral analysis, the Rust core:
* Randomizes the **TTL** (Time To Live) field for different streams (simulating multiple devices behind a NAT).
* Uses various **TCP Fingerprints** (different header sizes, SACK options, Window Scale) for parallel connections.
* Mixes traffic: 20% real user data, 80% "smart noise" designed to stress the **ISP-level filter**.

---

## 4. Automation System and JSON Configs

### 4.1. Strategy Structure
The configuration allows logic updates without rebuilding the application, enabling rapid response to new **censorship signatures**.
```json
{
  "provider_id": "ISP_GLOBAL_ID",
  "strategies": [
    {
      "name": "aggressive_streaming_fix",
      "target_hosts": ["video-platform.com", "service.com"],
      "evasion": {
        "split_pos": 2,
        "method": "fake_packet",
        "payload_type": "tls_v1.3"
      },
      "stress": {
        "enabled": true,
        "threads": 4,
        "mode": "handshake_flood"
      }
    }
  ]
}
```

### 4.2. Strategy Hunter
An automated search algorithm that:
1.  Fetches base configs from a remote repository.
2.  Pings "beacons" (Telegram, Google, Cloudflare).
3.  If latency exceeds 500ms or packet loss is detected, it begins brute-forcing evasion parameters.
4.  Upon finding a working combination, it saves it locally and shares it anonymously with the network.

---

## 5. Platform Implementation

### 5.1. Android (Java/Kotlin)
* Utilizes `VpnService` API to route global traffic.
* Passes the TUN file descriptor (FD) to the Rust core via JNI for high-performance processing.

### 5.2. iOS (Swift + Rust)
* Utilizes `NetworkExtension` (Packet Tunnel Provider).
* **Power Management:** The "Stress Engine" is throttled based on battery state to prevent the OS from killing the extension for high energy usage.

### 5.3. Desktop (Windows/Linux)
* **Windows:** Uses the `Wintun` driver for low-latency interface creation.
* **UI:** A JavaFX-based dashboard to visualize real-time bypass efficiency and **DPI load levels**.

---

## 6. Roadmap (Milestones)

* **Phase 1: Ignition (0-2 months):** Core Rust development, TUN implementation, and basic CLI for Linux.
* **Phase 2: Infection (2-4 months):** **Stress Engine** deployment (TLS flood) and Android JNI bridge.
* **Phase 3: Hydra (4-6 months):** Strategy Hunter automation and iOS NetworkExtension support.
* **Phase 4: Global Impact (6+ months):** Crate publication and scaling the network to achieve critical mass (100k+ nodes).

---

## 7. Philosophy and Legal Disclaimer
**Hydra-Core** is a research tool for studying network resilience against aggressive censorship. The developers are not responsible for usage that violates local regulations. We maintain that **freedom of information is a fundamental human right**, and technical countermeasures against **State-level DPI** are a necessary form of digital self-defense.

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
