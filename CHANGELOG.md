# Changelog

All notable changes to the HERMES protocol are documented here.

This project follows a versioning scheme where:
- **Phase 0** = intra-clan protocol (file-based, single instance)
- **Phase 1** = inter-clan protocol (gateway, Agora, attestations)
- **v1.0** = consolidated spec across all five research lines (L1-L5)

---

## [v0.2.0-alpha] — 2026-03-01

### The Agora Begins

This release introduces L5 — the social layer that allows independent HERMES clans to discover each other, collaborate, and build verifiable reputation without exposing private data.

### Added

- **ARC-3022: Agent Gateway Protocol** (DRAFT)
  - NAT-like boundary component between clan and public Agora
  - Identity translation: internal agent names → public aliases (never exposed)
  - Outbound filter: default-deny, operator approval for all data leaving the clan
  - Inbound validator: source verification, rate limiting, quarantine for first contact
  - `AGORA:` prefix convention for external messages on internal bus
  - TOFU (Trust-On-First-Use) model for inter-clan trust
  - Attestation protocol: signed certifications of cross-clan value delivery
  - Resonance metric: externally-validated reputation from attestations (decays, rewards diversity)
  - Dual metric architecture: Bounty (internal) + Resonance (external)

- **Research Agenda: L5 Social Topology**
  - Three sub-phases: L5a (Gateway + Profile), L5b (Attestation + Resonance), L5c (Visual Agora)
  - Six new mathematical tools for reputation modeling
  - Timeline integrated with L1-L4 research lines

- **docs/USE-CASES.md** — Six real-world deployment scenarios
  - Solo operator multi-domain, small team coordination, cross-clan collaboration
  - Community governance, personal productivity, open-source project coordination

- **docs/RESEARCH-AGENDA.md** — Public research roadmap (5 lines, L1-L5)

- **AES-2040** (Agent Visualization Standard) added to planned index

### Changed

- **README.md** — Added Agora section, gateway diagram, dual metric explanation, updated project structure
- **docs/ARCHITECTURE.md** — Added gateway boundary diagram, dual reputation model, ARC-3022 to specs table
- **docs/GLOSSARY.md** — Added 10 L5 terms: Agora, Attestation, Bounty, External Identity, Gateway, Public Profile, Quest, Resonance, TOFU, Translation Table
- **spec/INDEX.md** — Added ARC-3022 (DRAFT) and AES-2040 (PLANNED)
- **.gitignore** — Protected `.claude/` and `CLAUDE.md` from public repo

### Why This Matters

Phase 0 proved that file-based signaling works for agents within a single clan. But the real promise of HERMES is the same promise TCP/IP made: **open interconnection**. ARC-3022 is the first step toward a world where independent AI agent teams can meet, verify each other, and collaborate — without any single platform controlling the interaction.

The Agora is not a marketplace. It's a public square.

---

## [v0.1.0-alpha] — 2026-02-28

### Phase 0: The Foundation

The first public release of HERMES — a complete, working protocol for file-based inter-agent communication within a single clan.

### Added

- **7 core specifications** (all IMPLEMENTED):
  - ARC-0001: HERMES Architecture — the meta-standard defining the 5-layer stack
  - ARC-0791: Addressing & Routing — namespace addressing, star topology, Dijkstra/Erlang B analysis
  - ARC-0793: Reliable Transport — SYN/FIN/ACK session lifecycle
  - ARC-1918: Private Spaces & Firewall — namespace isolation, credential binding, data-cross protocol
  - ARC-5322: Message Format — JSONL wire format, 120-char Shannon constraint, ABNF grammar
  - ATR-X.200: Reference Model — formal 5-layer model (Physical → Application)
  - ATR-Q.700: Out-of-Band Signaling — design philosophy (signaling, not data)

- **30 standards planned** across three tracks:
  - ARC (IETF lineage): 16 standards
  - ATR (ITU-T lineage): 8 standards
  - AES (IEEE lineage): 5 standards

- **Python reference implementation** (46 tests passing):
  - Bus read/write with validation
  - Message lifecycle (create, consume, ACK, expire, archive)
  - Firewall rule evaluation
  - Routing table resolution
  - Full ARC-5322 validation algorithm

- **Documentation**:
  - README with ISP analogy and architecture overview
  - Quickstart guide (deploy in 5 minutes)
  - Architecture guide with ASCII diagrams
  - Agent structure guide (practical namespace organization)
  - Glossary of all HERMES terms
  - Contributing guide with standards proposal process

- **Examples**:
  - Sample bus file with valid messages
  - Sample routing table
  - Working Python agent (`simple_agent.py`) with full SYN/WORK/FIN cycle

- **Infrastructure**:
  - Init script (`scripts/init_hermes.sh`) for bootstrapping instances
  - GitHub issue template for ARC proposals
  - MIT license

### Design Decisions

- **File-based, not network-based**: HERMES agents share a filesystem, not an API. This eliminates servers, databases, and Docker — the protocol works anywhere files work.
- **JSONL, not JSON**: One message per line enables append-only writes and line-by-line parsing. No need to parse the entire bus to read one message.
- **120-character payload limit**: Inspired by Shannon's information theory. Forces precision over verbosity. If you can't say it in 120 chars, you're packing too many concerns.
- **Star topology with controller**: Simple, auditable, single point of coordination. Scales to ~50 namespaces before needing hierarchy (see L4 research line).
- **Human-in-the-loop**: HERMES informs, humans decide. No autonomous cross-namespace actions. This is a coordination protocol, not an automation framework.
- **Standards-first**: Every feature is a spec. Every spec maps to a real-world standard (IETF, ITU-T, or IEEE). This grounds the protocol in decades of network engineering.

### Why This Matters

AI agent frameworks are proliferating, but they're all walled gardens. Each platform has its own communication model, its own tool format, its own assumptions about trust. HERMES takes the opposite approach: define an **open protocol** that any agent on any platform can implement. The same way HTTP doesn't care if you're running Apache or Nginx, HERMES doesn't care if you're running Claude Code, Cursor, or a custom LLM pipeline.

The protocol is named after Hermes — the messenger who crosses boundaries. That's what this does.

---

## Versioning Note

HERMES uses alpha versioning during the research phase. The version will reach **v1.0** when all five research lines (L1-L5) produce at least one IMPLEMENTED specification each and the protocol can sustain inter-clan communication with cryptographic integrity.
