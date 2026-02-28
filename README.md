# HERMES

**A lightweight, file-based communication protocol for multi-agent AI systems.**

Inspired by TCP/IP. No servers, no databases — just files and convention.

---

## What is HERMES?

HERMES is an open protocol that lets AI agents talk to each other across isolated workspaces using nothing but plain text files. Think of it as **TCP/IP for AI agents** — a layered communication stack where every message is a line in a JSONL file, every workspace is a namespace, and every agent reads and writes to a shared bus.

```
┌──────────────────────────────────────────────┐
│  L4  Application    Agents read/write bus     │
├──────────────────────────────────────────────┤
│  L3  Transport      SYN/FIN/ACK + TTL        │
├──────────────────────────────────────────────┤
│  L2  Network        Routing tables            │
├──────────────────────────────────────────────┤
│  L1  Frame          JSONL message format      │
├──────────────────────────────────────────────┤
│  L0  Physical       File system               │
└──────────────────────────────────────────────┘
```

### Why?

Modern AI agents (Claude Code, Cursor, Copilot, custom LLM pipelines) work in **stateless sessions**. They start, do work, and disappear. If you run multiple agents across different projects or domains, they can't coordinate — unless you give them a shared protocol.

HERMES solves this with radical simplicity:

- **A JSONL bus file** where messages live (one line = one message)
- **A routing table** that maps namespaces to file paths
- **SYN/FIN handshakes** at session start/end
- **TTL-based expiry** so the bus stays clean
- **Firewall rules** so namespaces stay isolated

No servers. No databases. No Docker. No cloud. Just files that any agent can read and write.

### The ISP Analogy

Each HERMES deployment is like an **Internet Service Provider**:

- You run your own internal network (your clan of agents)
- Your namespaces are like private IP ranges (isolated by default)
- The bus is your backbone (carries signaling, not data)
- You can peer with other HERMES instances through standard protocols
- The specs are open — anyone can join the network

## Quick Start

Deploy your own HERMES instance in 5 minutes: **[Quickstart Guide](docs/QUICKSTART.md)**

## The Standards System

HERMES uses an RFC-like standards process with three tracks, each mapping to a real-world standards body:

| Prefix | Lineage | Domain | Example |
|--------|---------|--------|---------|
| **ARC** | IETF RFC | Core protocols | ARC-0793: Reliable Transport |
| **ATR** | ITU-T Rec | Architecture & models | ATR-X.200: Reference Model |
| **AES** | IEEE Std | Implementation standards | AES-802.1Q: Namespace Isolation |

### Core Standards (Phase 0)

| Standard | Title | Status |
|----------|-------|--------|
| [ARC-0001](spec/ARC-0001.md) | HERMES Architecture | IMPLEMENTED |
| [ATR-X.200](spec/ATR-X200.md) | Reference Model | IMPLEMENTED |
| [ARC-5322](spec/ARC-5322.md) | Message Format | IMPLEMENTED |
| [ARC-0793](spec/ARC-0793.md) | Reliable Transport | IMPLEMENTED |
| [ARC-0791](spec/ARC-0791.md) | Addressing & Routing | IMPLEMENTED |
| [ARC-1918](spec/ARC-1918.md) | Private Spaces & Firewall | IMPLEMENTED |
| [ATR-Q.700](spec/ATR-Q700.md) | Out-of-Band Signaling | INFORMATIONAL |

Full index: **[spec/INDEX.md](spec/INDEX.md)**

## Architecture at a Glance

```
              ┌─────────────┐
              │  Controller  │  (reads all, executes none)
              │  Namespace   │
              └──────┬───────┘
                     │
          ┌──────────┼──────────┐
          │          │          │
    ┌─────┴──┐ ┌────┴───┐ ┌───┴─────┐
    │ eng    │ │ ops    │ │ finance │   Namespaces
    │        │ │        │ │         │   (isolated)
    └───┬────┘ └───┬────┘ └────┬────┘
        │          │           │
        └──────────┴───────────┘
                   │
            ┌──────┴──────┐
            │  bus.jsonl   │  The shared bus
            │  (JSONL)     │  (signaling only)
            └─────────────┘
```

- **Namespaces** are isolated workspaces — each has its own agents, config, and memory
- **The bus** carries coordination messages, not actual data
- **The controller** has read access to all namespaces but cannot execute in any
- **Firewalls** prevent credentials and tools from crossing namespace boundaries
- **Humans** approve all cross-namespace data movement

## Key Design Principles

1. **File-based** — No servers, no databases. Just files any tool can read/write
2. **Stateless sessions** — Agents come and go. The bus persists
3. **Human-in-the-loop** — HERMES informs, humans decide
4. **Firewall by default** — Namespaces are isolated. Crossings require explicit rules
5. **Signaling, not data** — The bus carries control messages, not payloads
6. **Open standard** — Anyone can implement, extend, or fork

## Reference Implementation

A minimal Python implementation is included for validation and experimentation:

```bash
cd reference/python
pip install -e .
python -m pytest tests/
```

See [reference/python/](reference/python/) for details.

## Project Structure

```
hermes/
├── spec/              # Formal standards (ARC, ATR, AES)
├── docs/              # Guides, architecture, glossary
├── reference/python/  # Reference implementation
├── examples/          # Sample bus, routes, configs
├── .github/           # Issue templates for proposals
├── CONTRIBUTING.md    # How to contribute
└── LICENSE            # MIT
```

## Contributing

HERMES is built by and for the community. See [CONTRIBUTING.md](CONTRIBUTING.md) for:

- How to propose new standards (ARC/ATR/AES)
- Contributing code or documentation
- Adding implementations in new languages

## Mission

> Technology with soul frees time for sharing, for art, for community — to look each other in the eye again and smile.

HERMES exists because AI agents shouldn't be locked into proprietary communication platforms. The same way TCP/IP created an open internet, HERMES aims to create an open network for AI agent coordination — ethical, sustainable, and free.

The protocol is named after Hermes, the Greek messenger of the gods — the one who crosses boundaries. That's what this protocol does: it lets agents cross the boundaries between isolated workspaces, safely and transparently.

**Join the network. Build the protocol. Free the agents.**

## License

[MIT](LICENSE) — Free as in freedom, free as in beer.
