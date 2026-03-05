# Extended Detection and Response System with Centralized SIEM

A modular security platform combining lightweight XDR endpoint agents with a centralized SIEM server for real-time threat detection, analysis, and automated response.

Built as a senior capstone project at Qassim University (2024–2025) by a 5-person team.

---

## Architecture

```
┌──────────────────────┐         ┌──────────────┐         ┌──────────────────────────┐
│   Endpoint (Windows) │         │              │         │      SIEM Server         │
│                      │         │   Apache     │         │                          │
│  ┌────────────────┐  │  Kafka  │   Kafka      │  Kafka  │  ┌────────────────────┐  │
│  │   XDR Agent    │──┼────────►│   Broker     │────────►│  │  Kafka Consumer    │  │
│  │   (C)          │  │  (enc+  │              │         │  └────────┬───────────┘  │
│  └────────────────┘  │  comp)  └──────────────┘         │           │              │
│                      │                                  │  ┌────────▼───────────┐  │
│  Collects:           │         ┌──────────────┐         │  │  Decompress +      │  │
│  • Network packets   │  ZeroMQ │              │  ZeroMQ │  │  Decrypt           │  │
│  • Process events    │◄────────│  Response    │◄────────│  └────────┬───────────┘  │
│  • Windows event logs│         │  Actions     │         │           │              │
│  • File integrity    │         └──────────────┘         │  ┌────────▼───────────┐  │
│  • File scanning     │                                  │  │  Detection Engines │  │
│  • System health     │                                  │  │  • Rule Engine     │  │
│                      │                                  │  │  • Correlation     │  │
└──────────────────────┘                                  │  │  • YARA (94% det.) │  │
                                                          │  │  • ML Anomaly Det. │  │
                                                          │  └────────┬───────────┘  │
                                                          │           │              │
                                                          │  ┌────────▼───────────┐  │
                                                          │  │  Elasticsearch     │  │
                                                          │  │  + Django API      │  │
                                                          │  │  + Web Dashboard   │  │
                                                          │  └────────────────────┘  │
                                                          └──────────────────────────┘
```

## Repositories

| Repository | Description | Language |
|-----------|-------------|----------|
| **xdr-agent** | Modular endpoint agent — 10 independent modules for data collection, compression, encryption, and transmission | C |
| **siem-server** | Centralized backend — Kafka consumer, multi-engine detection (Rule, Correlation, YARA, ML), Elasticsearch indexing, Django API, web dashboard | Python |

## How It Works

**XDR Agent (C)** — Runs on Windows endpoints as a set of independent modules managed by a parent process. Each module handles one task:

| Module | What It Does |
|--------|-------------|
| Network Data Collector | Captures raw packets via Npcap |
| Process Monitor | Tracks process creation via WMI |
| Events Data Collector | Reads Windows Security/Application/System event logs |
| File Integrity Checker | SHA-256 hashing of sensitive files against baseline |
| File Scanner | Scans new/modified files for malicious patterns |
| System Health Monitor | CPU, RAM, disk, uptime metrics |
| Compressor | ZSTD compression (~60% size reduction) |
| Encryptor | AES encryption via OpenSSL |
| Kafka Communicator | Forwards encrypted+compressed data to SIEM |
| Action Executor | Receives and executes response commands from SIEM via ZeroMQ |

The parent process manager launches, monitors, and auto-restarts any module that crashes.

**SIEM Server (Python)** — Consumes Kafka messages, decompresses and decrypts them, normalizes the data, and routes it through four detection engines:

- **Rule Engine** — Static rules for known threat patterns
- **Correlation Engine** — Links related events across data categories to detect multi-stage attacks
- **YARA Engine** — Signature-based file scanning (94% detection rate in testing)
- **ML Engine** — Anomaly detection for network traffic patterns

Alerts are indexed into Elasticsearch and displayed on a web dashboard built with Django and Bootstrap.

## Performance (Tested)

| Metric | Result |
|--------|--------|
| YARA detection rate | 94% on known rule-matched threats |
| ZSTD compression | ~60% average message size reduction |
| Kafka throughput | 200+ messages/second sustained |
| False positive rate | Low across ML, Rule, and Correlation engines |
| Agent stability | No crashes or memory leaks under stress testing |

## Tech Stack

**XDR Agent:** C, ZeroMQ, OpenSSL (AES), ZSTD, cJSON, libxml2, librdkafka, Npcap

**SIEM Server:** Python, Django, FastAPI, Elasticsearch, kafka-python, yara-python, scikit-learn, Bootstrap, WebSockets

**Infrastructure:** Apache Kafka, Elasticsearch, SQLite (config), ZeroMQ (bidirectional agent ↔ server)

## Team

| Member | Primary Role |
|--------|-------------|
| Mohammed Al-Dughaim | Project lead, XDR agent development (C) |
| Abdulmajeed Al-Rasheed | SIEM server backend |
| Ibrahim Al-Bulayhi | Frontend dashboard |
| Fahad Al-Sunedi | Detection engines |
| Amro Al-Tawashi | Testing and integration |

Supervised by **Dr. Abdullah Al-Abdulatif**, Qassim University, Department of Computer Science.

## License

MIT License (original code). Third-party libraries retain their respective licenses — see individual repository READMEs for details.
