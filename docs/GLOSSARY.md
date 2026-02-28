# HERMES Glossary

Canonical definitions for all HERMES terms. When in doubt, this document is authoritative.

## Core Concepts

### Agent
An AI-powered entity that operates within a namespace, reads and writes to the bus, and performs work. Agents are typically stateless — they exist only for the duration of a session. Also called a "skill" in some implementations.

### Bus
The shared JSONL file (`bus.jsonl`) where all active messages live. The bus is a signaling channel — it carries coordination messages, not data payloads. See [ARC-5322](../spec/ARC-5322.md).

### Bus Archive
A secondary JSONL file (`bus-archive.jsonl`) where expired messages are moved. Messages are never deleted — only archived when their TTL expires.

### Clan
A group of agents organized under a single HERMES instance. A clan may have multiple namespaces, a controller, and a routing table. Analogous to an autonomous system (AS) in internet routing.

### Controller
A special namespace with read access to all other namespaces. The controller can detect desyncs, propose fixes, and route messages, but it cannot execute actions in other namespaces. The controller is the "router" of the HERMES network.

### Data Cross
A permitted transfer of data between namespaces, carried via `data_cross` message type. Data crosses are governed by explicit firewall rules and require human approval. Only data moves — never credentials or tool access.

### Dimension
See **Namespace**. The term "dimension" is used in some implementations as a synonym for namespace, emphasizing the isolation between workspaces.

### Dispatch
A message type (`dispatch`) where a controller or coordination agent assigns one or more agents to a task. Includes the task description, suggested agents, and routing metadata.

### Firewall
The set of rules that govern namespace isolation. Firewalls define which external tools each namespace can access, which data crosses are permitted, and which credentials belong to which namespace. See [ARC-1918](../spec/ARC-1918.md).

### Instance
A single deployment of the HERMES protocol — one bus, one routing table, one set of namespaces. Analogous to an ISP or autonomous system. Multiple instances can peer through inter-instance transport (future spec).

## Protocol Terms

### ACK (Acknowledge)
The mechanism by which a namespace confirms it has consumed a message. The namespace identifier is appended to the message's `ack` array. A message is considered fully delivered when all intended recipients have ACKed.

### FIN (Finish)
The session-end protocol. When an agent's session ends, it must: (1) write any state changes to the bus, (2) update its SYNC HEADER, and (3) ACK consumed messages. See [ARC-0793](../spec/ARC-0793.md).

### Message
A single JSONL object on the bus. Each message has a timestamp, source, destination, type, payload, TTL, and ACK array. See [ARC-5322](../spec/ARC-5322.md).

### Namespace
An isolated workspace within a HERMES instance. Each namespace has its own configuration, memory, agents, and external tool permissions. Namespaces communicate only through the bus — never directly. Analogous to a private IP range.

### Routing Table
A file (`routes.md` or equivalent) that maps namespace identifiers to their file system paths, head agents, tool permissions, and permitted data crosses. See [ARC-0791](../spec/ARC-0791.md).

### SYN (Synchronize)
The session-start protocol. When an agent's session begins, it must: (1) read the bus, (2) filter for messages addressed to its namespace, (3) report pending messages, (4) flag stale messages (>3 days unACKed). See [ARC-0793](../spec/ARC-0793.md).

### SYNC HEADER
A metadata block in each namespace's configuration file that tracks the current state: version number, last sync timestamp, state summary, and pending message counts.

### TTL (Time To Live)
The number of days a message remains active on the bus from its emission timestamp. When TTL expires, the message is archived. Default TTLs vary by message type: `state` = 7 days, `alert` = 5 days, `event` = 3 days.

## Message Types

### `state`
A change in the state of a namespace. Broadcast to all or targeted. Example: a project moving from planning to active.

### `alert`
Urgent information that another namespace needs to know. Higher priority than `state`, shorter default TTL (5 days).

### `event`
Something that happened. Informational only — no action required. Shortest default TTL (3 days).

### `request`
A namespace needs something from another namespace. The destination should respond or acknowledge.

### `data_cross`
A permitted data transfer between namespaces. Governed by firewall rules. May include the `PROJ:` prefix for projections and the `@source` suffix for data provenance.

### `dispatch`
An assignment of agent(s) to a task by a controller or coordination agent. Includes task description and suggested agents.

## Standards System

### ARC (Agent Request for Comments)
Core protocol standards. Lineage: IETF RFC. Format: `ARC-NNNN` (4-digit, zero-padded).

### ATR (Agent Telecom Recommendation)
Architecture and reference model standards. Lineage: ITU-T Recommendations. Format: `ATR-X.NNN` or `ATR-Q.NNN`.

### AES (Agent Engineering Standard)
Implementation and interoperability standards. Lineage: IEEE Standards. Format: `AES-NNN.NN`.

### Status Levels
- **IMPLEMENTED** — Specification is complete and has a working reference implementation
- **DRAFT** — Specification is in progress, subject to change
- **PLANNED** — Specification is outlined but not yet written
- **INFORMATIONAL** — Not a protocol spec; provides context, philosophy, or guidance

## Conventions

### Atomicity
One message = one topic. Never pack multiple concerns into a single message payload. If there are two things to say, send two messages.

### PROJ: Prefix
When a `data_cross` message contains projections or estimates (not verified data), the payload MUST begin with `PROJ:`. Absence of the prefix means the data is verified.

### @source Suffix
When a `data_cross` message includes monetary amounts or quantitative data, the payload SHOULD end with `@source_name` to indicate data provenance.
