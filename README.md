# ⚡ CollabEditor: Real-Time CRDT Collaborative Engine

[![C++20](https://img.shields.io/badge/C%2B%2B-20-blue.svg?style=flat&logo=c%2B%2B)](https://en.cppreference.com/w/cpp/20)
[![WebSockets](https://img.shields.io/badge/Networking-uWebSockets-orange.svg)](https://github.com/uNetworking/uWebSockets)
[![Architecture](https://img.shields.io/badge/Sync_Engine-RGA_CRDT-brightgreen.svg)](#architecture)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

CollabEditor is a real-time collaborative text editor engine designed around a Replicable Growable Array (RGA) Conflict-Free Replicated Data Type (CRDT). The planned system uses modern C++20 and WebSockets to provide strong eventual consistency across distributed clients without operational transformation or lock contention.

> **Project status:** The repository currently contains the project structure and documentation scaffold. The engine, client, tests, and deployment files are being built incrementally.

## Features

- **Conflict-free synchronization:** RGA-based state convergence for edits received out of order.
- **Non-blocking networking:** Bidirectional event streaming through asynchronous WebSocket I/O.
- **Deterministic ordering:** Node identifiers use `<LamportTimestamp, ClientID, LocalSequence>` keys.
- **Persistent recovery:** An append-only write-ahead log (WAL) is planned for crash recovery.
- **Tombstone lifecycle:** Logical deletion preserves causal parent references.

## Technology Stack

| Layer | Technology | Purpose |
| --- | --- | --- |
| Core engine | C++20 | RGA management and application logic |
| Networking | uWebSockets / libuv | Asynchronous WebSocket communication |
| Serialization | Protocol Buffers / FlatBuffers | Compact network payloads |
| Build system | CMake 3.20+ | Cross-platform builds |
| Containerization | Docker | Reproducible deployment |
| Frontend | React + Monaco Editor | Browser-based editing interface |

## Architecture

```text
Client 1 (Monaco UI) ──┐
                       ├── WebSockets ──> C++ WebSocket Event Engine
Client 2 (Monaco UI) ──┘                         │
                                                 ▼
                                      In-memory RGA Sequence CRDT
                                                 │
                                                 ▼
                                      Append-only WAL Persistence
```

## Repository Structure

```text
collab-editor/
├── client/       # React and Monaco Editor client
├── server/       # Collaborative editing server
├── deploy/       # Deployment configuration
├── nginx/        # Reverse-proxy configuration
├── LICENSE
└── README.md
```

## Getting Started

### Prerequisites

- GCC 11+ or Clang 13+ with C++20 support
- CMake 3.20+
- libuv and OpenSSL
- Docker (optional)

### Clone the repository

```bash
git clone https://github.com/Ashiii27/collab-editor.git
cd collab-editor
```

### Build from source

```bash
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --parallel
```

### Run with Docker

```bash
docker build -t collab-editor:latest .
docker run -d -p 9001:9001 --name collab_service collab-editor:latest
```

## Wire Protocol

WebSocket events are planned to use a binary or packed JSON frame with the following fields:

| Field | Type | Description |
| --- | --- | --- |
| `type` | `uint8_t` | `0x01` Insert, `0x02` Delete, or `0x03` Sync Document |
| `lamport` | `uint64_t` | Sender's current Lamport timestamp |
| `client_id` | `uint32_t` | Unique client connection identifier |
| `seq` | `uint32_t` | Monotonic local sequence number |
| `parent_id` | `NodeID` | Target parent node key: `<Lamport, ClientID, Seq>` |
| `value` | `char` | Inserted character payload |

## Performance Targets

- Local operation latency: $\mathcal{O}(1)$ with an active editor cursor reference.
- Remote integration complexity: $\mathcal{O}(k)$, where $k$ is the number of concurrent edits at the target parent node.
- Integration throughput: sub-millisecond operation processing across 1,000+ concurrent sessions.

## Contributing

Contributions, issues, and design discussions are welcome. Please open an issue before submitting a large architectural change.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.