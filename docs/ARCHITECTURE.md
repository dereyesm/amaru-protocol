# HERMES Architecture Guide

A visual guide to the HERMES protocol stack.

## The 5-Layer Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  L4  APPLICATION                                                │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │
│  │  Agent A  │  │  Agent B  │  │  Agent C  │  │  Agent D  │  │
│  │  (eng)    │  │  (ops)    │  │  (fin)    │  │  (ctrl)   │  │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  │
│        │              │              │              │          │
│════════╪══════════════╪══════════════╪══════════════╪══════════│
│        │              │              │              │          │
│  L3  TRANSPORT (SYN / FIN / ACK / TTL)                        │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  Session lifecycle: SYN ──── Active ──── FIN           │   │
│  │  Delivery: at-least-once via ACK array                 │   │
│  │  Expiry: TTL countdown → archive                       │   │
│  └────────────────────────────────────────────────────────┘   │
│        │              │              │              │          │
│════════╪══════════════╪══════════════╪══════════════╪══════════│
│        │              │              │              │          │
│  L2  NETWORK (Routing)                                        │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  routes.md: namespace → paths → agents → tools         │   │
│  │  Adjacency: star topology, controller at center        │   │
│  │  Addressing: unicast (named) or broadcast (*)          │   │
│  └────────────────────────────────────────────────────────┘   │
│        │              │              │              │          │
│════════╪══════════════╪══════════════╪══════════════╪══════════│
│        │              │              │              │          │
│  L1  FRAME (Message Format)                                   │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  {"ts":"...","src":"...","dst":"...","type":"...",      │   │
│  │   "msg":"...","ttl":N,"ack":[]}                        │   │
│  │  One JSON object per line. Max payload: 120 chars.     │   │
│  └────────────────────────────────────────────────────────┘   │
│        │              │              │              │          │
│════════╪══════════════╪══════════════╪══════════════╪══════════│
│        │              │              │              │          │
│  L0  PHYSICAL (File System)                                   │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  bus.jsonl          ← active messages                  │   │
│  │  bus-archive.jsonl  ← expired messages                 │   │
│  │  routes.md          ← routing table                    │   │
│  │  [ns]/config        ← per-namespace configuration      │   │
│  │  [ns]/memory/       ← per-namespace persistent state   │   │
│  │  [ns]/agents/       ← per-namespace agent definitions  │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Message Lifecycle

```
 Agent writes message          Message on bus           Recipient reads
 ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
 │                  │     │                  │     │                  │
 │  1. FIN protocol │     │  3. Lives on bus │     │  5. SYN protocol │
 │     appends msg  │────>│     with TTL     │────>│     reads bus    │
 │                  │     │     countdown    │     │                  │
 │  2. Sets ts, src │     │                  │     │  6. Filters by   │
 │     dst, type,   │     │  4. Broadcast:   │     │     dst match    │
 │     msg, ttl     │     │     all read     │     │                  │
 │                  │     │     Unicast:     │     │  7. ACKs message │
 │                  │     │     only dst     │     │                  │
 └──────────────────┘     └──────────────────┘     └──────────────────┘
                                   │
                                   │ TTL expires
                                   ▼
                          ┌──────────────────┐
                          │  bus-archive.jsonl│
                          │  (permanent log) │
                          └──────────────────┘
```

## Namespace Topology

HERMES uses a **star topology** with the controller at the center:

```
                    ┌────────────────┐
                    │   controller   │
                    │   (read-only)  │
                    └───────┬────────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
       ┌──────┴──────┐ ┌───┴────┐ ┌──────┴──────┐
       │ engineering │ │  ops   │ │  finance    │
       │             │ │        │ │             │
       │ Tools: A, B │ │Tools: C│ │ Tools: D, E │
       │ Agents: 5   │ │Agents:3│ │ Agents: 2   │
       └──────┬──────┘ └───┬────┘ └──────┬──────┘
              │             │             │
              └─────────────┴─────────────┘
                            │
                     ┌──────┴──────┐
                     │  bus.jsonl   │
                     └─────────────┘
```

**Key rules**:
- Namespaces NEVER communicate directly — all traffic goes through the bus
- The controller can read all namespaces but cannot execute in any
- Each namespace has its own isolated set of tools and credentials
- Data crosses require explicit firewall rules + human approval

## Firewall Model

```
 ┌─────────────────────────────────────────────────┐
 │                   HERMES Instance                │
 │                                                  │
 │  ┌────────────┐          ┌────────────┐         │
 │  │engineering │          │  finance   │         │
 │  │            │          │            │         │
 │  │ Tools:     │          │ Tools:     │         │
 │  │  ✅ jira   │  data    │  ✅ sheets │         │
 │  │  ✅ github │ ─cross──>│  ✅ banking│         │
 │  │  ❌ sheets │  only    │  ❌ jira   │         │
 │  │  ❌ banking│          │  ❌ github │         │
 │  └────────────┘          └────────────┘         │
 │                                                  │
 │  Firewall rules:                                │
 │  ┌────────────────────────────────────────┐     │
 │  │ eng  → finance : data_cross ALLOW      │     │
 │  │ eng  → finance : tool_access DENY      │     │
 │  │ eng  → finance : credential_cross DENY │     │
 │  └────────────────────────────────────────┘     │
 └─────────────────────────────────────────────────┘
```

## Session Lifecycle (SYN/FIN)

```
  Session Start                    Session Active                  Session End
  ┌──────────┐                    ┌──────────────┐                ┌──────────┐
  │   SYN    │                    │              │                │   FIN    │
  │          │                    │  Agent does  │                │          │
  │ 1. Read  │                    │  real work   │                │ 1. Write │
  │    bus   │                    │  in its      │                │    state │
  │          │──────────────────> │  namespace   │ ─────────────> │    to bus│
  │ 2. Filter│                    │              │                │          │
  │    by dst│                    │  Bus is NOT  │                │ 2. Update│
  │          │                    │  used during │                │    SYNC  │
  │ 3. Report│                    │  work — only │                │    HEADER│
  │    pending│                   │  at session  │                │          │
  │          │                    │  boundaries  │                │ 3. ACK   │
  │ 4. Flag  │                    │              │                │    consumed│
  │    stale │                    │              │                │    msgs  │
  │    (>3d) │                    │              │                │          │
  └──────────┘                    └──────────────┘                └──────────┘
```

## Control Plane vs Data Plane

```
  ┌─────────────────────────────────────────────────────────────┐
  │  CONTROL PLANE (HERMES)                                     │
  │                                                             │
  │  bus.jsonl ◄──── signaling messages ────► agents            │
  │  routes.md       (state, alerts, dispatch)                  │
  │  SYNC HEADERS                                               │
  │                                                             │
  │  "Where should work happen? What changed? Who needs to know?"│
  └─────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────┐
  │  DATA PLANE (Agent Work)                                    │
  │                                                             │
  │  Code repos      ◄──── actual work output ────► tools       │
  │  Documents             (code, docs, emails)     APIs        │
  │  Databases                                      Services    │
  │                                                             │
  │  "The actual work: writing code, sending emails, querying." │
  └─────────────────────────────────────────────────────────────┘
```

HERMES is a **signaling protocol**, not a data protocol. Like SS7 in telecom networks, it carries the coordination messages that tell agents where to work and what changed — but the actual work happens outside the bus.

## File System Layout

A typical HERMES deployment:

```
~/.hermes/                          # or any root directory
├── bus.jsonl                       # active messages
├── bus-archive.jsonl               # expired messages
├── routes.md                       # routing table
│
├── engineering/                    # namespace: engineering
│   ├── config.md                   # namespace config + SYNC HEADER
│   ├── memory/                     # persistent state
│   │   └── MEMORY.md
│   └── agents/                     # agent definitions
│       ├── lead.md
│       └── reviewer.md
│
├── finance/                        # namespace: finance
│   ├── config.md
│   ├── memory/
│   │   └── MEMORY.md
│   └── agents/
│       └── accountant.md
│
└── controller/                     # namespace: controller
    ├── config.md
    └── agents/
        └── router.md
```

## Related Specifications

| Spec | Title | What it covers |
|------|-------|---------------|
| [ARC-0001](../spec/ARC-0001.md) | HERMES Architecture | The meta-standard |
| [ATR-X.200](../spec/ATR-X200.md) | Reference Model | Formal 5-layer model |
| [ARC-5322](../spec/ARC-5322.md) | Message Format | JSONL packet spec |
| [ARC-0793](../spec/ARC-0793.md) | Reliable Transport | SYN/FIN/ACK |
| [ARC-0791](../spec/ARC-0791.md) | Addressing & Routing | Namespaces and routes |
| [ARC-1918](../spec/ARC-1918.md) | Private Spaces | Firewall model |
| [ATR-Q.700](../spec/ATR-Q700.md) | OOB Signaling | Design philosophy |
