<div align="center">

<br/>

```
███████╗ ██████╗ ██╗      █████╗ ██████╗     ███████╗██╗  ██╗██╗███████╗██╗     ██████╗
██╔════╝██╔═══██╗██║     ██╔══██╗██╔══██╗    ██╔════╝██║  ██║██║██╔════╝██║     ██╔══██╗
███████╗██║   ██║██║     ███████║██████╔╝    ███████╗███████║██║█████╗  ██║     ██║  ██║
╚════██║██║   ██║██║     ██╔══██║██╔══██╗    ╚════██║██╔══██║██║██╔══╝  ██║     ██║  ██║
███████║╚██████╔╝███████╗██║  ██║██║  ██║    ███████║██║  ██║██║███████╗███████╗██████╔╝
╚══════╝ ╚═════╝ ╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝    ╚══════╝╚═╝  ╚═╝╚═╝╚══════╝╚══════╝╚═════╝
```

**Secure IoT Gateway for Smart Solar Inverters**

*Hardware root of trust · Mutual TLS 1.3 · Edge AI anomaly detection · Grafana telemetry*

<br/>

[![ESP-IDF](https://img.shields.io/badge/ESP--IDF-v5.1-2196F3?style=for-the-badge&logo=espressif&logoColor=white)](https://docs.espressif.com/projects/esp-idf/)
[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![TensorFlow Lite](https://img.shields.io/badge/TFLite-2.13-FF6F00?style=for-the-badge&logo=tensorflow&logoColor=white)](https://www.tensorflow.org/lite)
[![MQTT](https://img.shields.io/badge/MQTT-5.0-660066?style=for-the-badge)](https://mqtt.org/)
[![Grafana](https://img.shields.io/badge/Grafana-Cloud-F46800?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=for-the-badge)](https://opensource.org/licenses/MIT)

<br/>

> A **three-tier defense-in-depth cybersecurity platform** for Distributed Energy Resources — retrofitting legacy solar inverters with hardware-rooted trust, encrypted MQTT communications, and an edge-deployed LSTM Autoencoder that detects False Data Injection attacks at **96.0% F1-score** with **<130 ms** end-to-end latency.

<br/>

---

</div>

<br/>

## ◈ Table of Contents

- [Overview](#-overview)
- [Validated Results](#-validated-results)
- [Security Architecture](#-security-architecture)
- [Key Capabilities](#-key-capabilities)
- [Hardware Requirements](#-hardware-requirements)
- [Software Stack](#-software-stack)
- [Data Flow](#-data-flow)
- [Installation & Setup](#-installation--setup)
- [Testing & Results](#-testing--results)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [Citation](#-citation)
- [License](#-license)

<br/>

---

## ◈ Overview

As the power grid shifts from centralized plants to millions of Distributed Energy Resources, legacy protocols like **Modbus RTU/TCP** — which transmit data in cleartext with zero authentication — become an increasingly dangerous attack surface. A compromised inverter fleet can destabilize grid frequency, cause equipment damage, and enable large-scale energy fraud.

Solar Shield addresses this gap with a **three-tier hardware/software architecture** that retrofits existing solar inverters without modifying inverter firmware:

```
THREAT SURFACE                    SOLAR SHIELD DEFENSE
──────────────────────────────────────────────────────
Malicious firmware upload    →    RSA-3072 Secure Boot V2 (eFuse-anchored)
Keys/certs at rest           →    AES-256-XTS Flash Encryption
Cleartext Modbus traffic     →    MQTT over TLS 1.3 (ChaCha20-Poly1305)
Unauthenticated clients      →    Mutual TLS with X.509 per-device certificates
Semantic data injection      →    LSTM Autoencoder (10-second sliding window)
Network DoS                  →    Exponential backoff + Hardware Watchdog Timer
Electrical transients        →    TVS diodes + PC817 optocoupler isolation
```

Validated on **physical solar inverter hardware**. Results exceed all non-functional requirements.

<br/>

---

## ◈ Validated Results

| Metric | Measured | Requirement | Status |
|--------|:--------:|:-----------:|:------:|
| F1-Score (LSTM Autoencoder) | **96.0%** | > 95% | ✓ |
| End-to-end anomaly latency | **129 ms** | < 500 ms | ✓ |
| False Positive Rate | **1.2%** | — | ✓ |
| R-TADS CPU usage (Pi 4) | **38%** | < 70% | ✓ |
| TLS handshake (first connect) | **120 ms** | — | ✓ |
| 0-RTT session resumption | **~14 ms** | — | ✓ |
| TFLite inference per window | **54 ms** | — | ✓ |
| Memory footprint (TFLite) | **195 MB** | — | ✓ |

> **Benchmark comparison:** Snort (legacy signature IDS) failed entirely on the wear-out attack. Random Forest achieved 90.4% F1. The LSTM Autoencoder captures temporal patterns that stateless classifiers cannot.

<br/>

---

## ◈ Security Architecture

```
┌──────────────────┐   RS485 / Modbus RTU     ┌───────────────────────────────┐
│  Solar Inverter  │◄────────────────────────►│   Solar Shield Edge           │
│                  │                          │   Controller  (ESP32-S3)      │
│  Growatt / SMA   │                          │                               │
│  Fronius / etc.  │                          │   ┌───────────────────────┐   │
└──────────────────┘                          │   │  Secure Boot V2       │   │
                                              │   │  RSA-3072  ·  eFuse   │   │
                                              │   └───────────────────────┘   │
                                              │   ┌───────────────────────┐   │
                                              │   │  Flash Encryption     │   │
                                              │   │  AES-256-XTS          │   │
                                              │   └───────────────────────┘   │
                                              │   ┌───────────────────────┐   │
                                              │   │  mbedTLS              │   │
                                              │   │  TLS 1.3 · ChaCha20   │   │
                                              │   └───────────────────────┘   │
                                              └──────────────┬────────────────┘
                                                             │
                                                  MQTT over TLS 1.3
                                                  mTLS · port 8883
                                                             │
                                                             ▼
                                              ┌──────────────────────────────┐
                                              │   Raspberry Pi 4 Gateway     │
                                              │                              │
                                              │   Mosquitto 2.0 (mTLS)       │
                                              │   R-TADS  ·  LSTM AE         │
                                              │   TensorFlow Lite runtime    │
                                              └──────────────┬───────────────┘
                                                             │
                                                    Secure outbound bridge
                                                             │
                                                             ▼
                                              ┌──────────────────────────────┐
                                              │   Cloud Platform             │
                                              │                              │
                                              │   Prometheus  (TSDB)         │
                                              │   Grafana     (dashboard)    │
                                              │   Webhooks    (alerts)       │
                                              └──────────────────────────────┘
```

<br/>

---

## ◈ Key Capabilities

### Device Layer — ESP32-S3

| Capability | Implementation | Security Benefit |
|-----------|---------------|-----------------|
| **Secure Boot V2** | RSA-3072, public key hash burned to eFuse | Unsigned firmware rejected at boot; device bricks if eFuse tampered |
| **Flash Encryption** | AES-256-XTS on all NVS partitions | Certificates and private keys protected at rest |
| **Hardware Isolation** | SM712 TVS diodes + PC817 optocouplers | Survives electrical transients; galvanic isolation from inverter bus |
| **TLS 1.3 only** | mbedTLS, `TLS_CHACHA20_POLY1305_SHA256` | No cleartext; no downgrade to TLS 1.2 or below |
| **mTLS client auth** | Per-device X.509 certificate (from local CA) | Server rejects any client without a valid signed certificate |

### Edge Layer — Raspberry Pi 4

| Capability | Implementation | Security Benefit |
|-----------|---------------|-----------------|
| **LSTM Autoencoder** | TFLite, 10-second sliding window | Detects semantic attacks invisible to signature-based IDS |
| **Dynamic thresholding** | μ + 1.5σ on reconstruction error | Minimizes false positives (1.2% FPR) without manual tuning |
| **Anomaly classification** | MSE spike pattern matching | Distinguishes oscillating-power wear-out from sudden voltage drops |
| **mTLS broker** | Mosquitto 2.0, client cert required | All ESP32 clients authenticated before any message accepted |

### Resilience

| Capability | Implementation |
|-----------|---------------|
| **Network DoS recovery** | Exponential backoff on MQTT reconnect |
| **Process crash recovery** | Hardware Watchdog Timer on ESP32 |
| **0-RTT resumption** | TLS session ticket cache (~14 ms reconnect) |

<br/>

---

## ◈ Hardware Requirements

| Component | Model / Spec | Role |
|-----------|-------------|------|
| **Edge Controller** | ESP32-S3 dev board with RS485 header | Secure Modbus bridge + crypto engine |
| **RS485 Transceiver** | MAX485 or SN65HVD72 | Level shifting for Modbus RTU |
| **Surge Protection** | SM712 TVS diodes | Overvoltage and ESD protection |
| **Isolation** | PC817 optocoupler | Galvanic isolation from inverter bus |
| **Gateway** | Raspberry Pi 4 (4 GB RAM) | Mosquitto broker + TFLite inference |
| **Inverter** | Any SunSpec-compliant inverter with RS485 port | Physical DER under test |
| **Network** | WPA3-secured Wi-Fi router | Isolated local LAN segment |

> **Optional:** ATECC608A external TPM for enterprise-grade PKI key storage.

<br/>

---

## ◈ Software Stack

### ESP32-S3 Firmware (C++ / FreeRTOS)

| Component | Version | Purpose |
|-----------|:-------:|---------|
| ESP-IDF | v5.1 | Base RTOS + Secure Boot V2 framework |
| mbedTLS | bundled | TLS 1.3 with `CHACHA20_POLY1305_SHA256` only |
| esp_mqtt | bundled | MQTT 5.0 client over mTLS |
| ModbusMaster | latest | RS485 polling, SunSpec Model 103 scaling |
| NVS (encrypted) | bundled | Certificate and key storage at rest |

### Raspberry Pi Gateway (Python 3.10)

| Component | Version | Purpose |
|-----------|:-------:|---------|
| Eclipse Mosquitto | 2.0 | MQTT broker, mTLS on port 8883 |
| TensorFlow Lite | 2.13 | Edge inference runtime (R-TADS) |
| Paho-MQTT | latest | Internal broker bridging |
| NumPy / Pandas | latest | Sliding window preprocessing |
| Grafana Agent | latest | Prometheus metrics exporter |

### Cloud

| Component | Purpose |
|-----------|---------|
| Prometheus | Time-series storage for telemetry |
| Grafana Cloud | Live dashboard + email/SMS alert webhooks |

<br/>

---

## ◈ Data Flow

```
1.  ESP32 polls inverter at 1 Hz
    (SunSpec Model 103 — raw integer registers)
         │
         ▼
2.  Scale registers → floating-point engineering units
    (voltage V, current A, power W, frequency Hz)
         │
         ▼
3.  Serialize to JSON payload
         │
         ▼
4.  Encrypt with TLS 1.3 (ChaCha20-Poly1305)
    Publish to Mosquitto broker (mTLS, port 8883)
         │
         ▼
5.  R-TADS ingests 10-second sliding window
    LSTM Autoencoder reconstructs expected signal
    Computes MSE reconstruction error
         │
       ┌─┴────────────────────┐
       │                      │
  MSE < μ + 1.5σ         MSE ≥ μ + 1.5σ
       │                      │
       ▼                      ▼
  Normal data            ANOMALY ALERT
  → Prometheus           → Grafana webhook
  → Grafana dashboard    → Email / SMS
```

<br/>

---

## ◈ Installation & Setup

### Prerequisites

```
ESP-IDF   v5.1+
Python    3.10+
OpenSSL   3.x
Mosquitto 2.0+
```

### Step 1 — Clone

```bash
git clone https://github.com/RootSecX/SolarShield.git
cd SolarShield
```

### Step 2 — ESP32-S3 Secure Boot *(one-time, irreversible)*

> ⚠️ Perform on an air-gapped machine. Once eFuses are burned, this cannot be undone.

```bash
# Generate RSA-3072 signing key
espsecure.py generate_signing_key --version 2 secure_boot_key.pem

# Extract public key hash
espsecure.py extract_public_key --keyfile secure_boot_key.pem public_key.bin

# Burn public key hash to eFuse
espefuse.py --port /dev/ttyUSB0 burn_key BLOCK_KEY0 public_key.bin

# Write-protect eFuse block
espefuse.py --port /dev/ttyUSB0 burn_efuse WR_DIS_BLK0
```

### Step 3 — Generate mTLS X.509 certificates

```bash
# Local Certificate Authority
openssl req -new -x509 -days 3650 -keyout ca.key -out ca.crt

# Raspberry Pi server certificate
openssl req -new -keyout server.key -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out server.crt

# ESP32 client certificate (repeat per device)
openssl req -new -keyout client1.key -out client1.csr
openssl x509 -req -in client1.csr -CA ca.crt -CAkey ca.key \
  -out client1.crt
```

### Step 4 — Flash ESP32 firmware

```bash
idf.py set-target esp32s3

# Set Wi-Fi SSID/password, MQTT broker IP, certificate paths
idf.py menuconfig

idf.py build
idf.py -p /dev/ttyUSB0 flash monitor
```

### Step 5 — Configure Raspberry Pi gateway

```bash
# Mosquitto with mTLS
sudo apt install mosquitto mosquitto-clients
sudo cp mosquitto.conf /etc/mosquitto/conf.d/solar-shield.conf
sudo systemctl restart mosquitto

# Python environment
python3 -m venv venv
source venv/bin/activate
pip install tflite-runtime paho-mqtt numpy pandas

# Start R-TADS inference engine
python3 rtads/rtads_service.py
```

### Step 6 — Grafana Cloud

```
1. Import /grafana/dashboard.json into your Grafana instance
2. Configure a Prometheus data source → gateway exporter URL
3. Set alert rule: anomaly_score > threshold for > 5 seconds
4. Add webhook notification channel (email / SMS / PagerDuty)
```

<br/>

---

## ◈ Testing & Results

### Security test matrix

| Test | Attack Simulated | Outcome |
|------|-----------------|---------|
| **TC-02** | Flashing unsigned firmware over USB | Secure Boot halted at eFuse mismatch — device did not boot |
| **TC-03** | Rogue MQTT client without valid certificate | Mosquitto rejected mTLS handshake — connection refused |
| **TC-04** | ARP spoofing + Wireshark packet capture | Payload fully encrypted — ChaCha20-Poly1305 ciphertext only |
| **TC-05** | False Data Injection — voltage forced to 0V | Anomaly detected in **185 ms** |
| **TC-06** | Wear-out attack — oscillating power 0% ↔ 100% | LSTM captured temporal pattern — MSE spike triggered alert |
| **TC-08** | TCP SYN flood via hping3 | 4.2 sec delay — no crash; Hardware Watchdog recovered service |

### Performance benchmarks

| Parameter | Measured | NFR Target | Pass |
|-----------|:--------:|:----------:|:----:|
| End-to-end anomaly latency | 129 ms | < 500 ms | ✓ |
| TLS handshake (cold start) | 120 ms | — | — |
| 0-RTT session resumption | ~14 ms | — | — |
| R-TADS CPU usage (Pi 4) | 38% | < 70% | ✓ |
| TFLite inference per window | 54 ms | — | — |
| Memory footprint (TFLite) | 195 MB | — | — |
| F1-Score (LSTM Autoencoder) | **96.0%** | > 95% | ✓ |
| False Positive Rate | **1.2%** | — | — |

### Classifier comparison

| Model | F1-Score | Wear-out Attack |
|-------|:--------:|:--------------:|
| **LSTM Autoencoder (Solar Shield)** | **96.0%** | Detected |
| Random Forest | 90.4% | Detected |
| Snort (signature IDS) | — | Failed entirely |

<br/>

---

## ◈ Roadmap

Contributions welcome — open an issue before starting a large feature:

- [ ] **Federated Learning** — adapt R-TADS to concept drift from inverter aging without full retraining
- [ ] **Hardware TPM** — ATECC608A integration for enterprise PKI and key attestation
- [ ] **IEC 61850** — expand beyond Modbus to substation automation (GOOSE / MMS)
- [ ] **DNP3 / IEC 104** — additional protocol parsers for grid SCADA environments
- [ ] **Blockchain audit trail** — hash anomaly logs into a permissioned ledger for immutable forensics
- [ ] **Multi-gateway orchestration** — Kubernetes edge cluster for > 100 inverters per site
- [ ] **HIL test automation** — Hardware-in-the-loop CI pipeline for firmware validation
- [ ] **Jetson / Rockchip port** — R-TADS inference on higher-throughput edge hardware

<br/>

---

## ◈ Contributing

This project follows **IEC 62443-4-1** secure development lifecycle principles.

```bash
# 1. Fork → clone → branch
git checkout -b feature/dnp3-parser

# 2. Make changes; run unit tests
pytest tests/

# 3. Commit with a descriptive message
git commit -m "feat: add DNP3 protocol parser with SunSpec mapping"

# 4. Push and open a Pull Request
git push origin feature/dnp3-parser
```

Priority contribution areas: protocol parsers · edge device ports · HIL test automation.

<br/>

---

## ◈ Citation

If you use Solar Shield in published research, please cite:

```bibtex
@misc{solarshield2026,
  author    = {Solar Shield Team},
  title     = {Solar Shield: Secure IoT Gateway for Smart Solar Inverters},
  year      = {2026},
  publisher = {GitHub},
  url       = {https://github.com/RootSecX/solar-shield}
}
```

<br/>

---

## ◈ License

Distributed under the **MIT License** — see [`LICENSE`](LICENSE) for full terms.

<br/>

---

<div align="center">

Built with `ESP-IDF` · `mbedTLS` · `TensorFlow Lite` · `Mosquitto` · `Grafana` · `Prometheus`

<br/>

*Green energy should not become a gateway for cyberattacks.*

<br/>

*If Solar Shield helped secure your DER infrastructure, consider leaving a ⭐*

</div>
