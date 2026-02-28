# Sample Routing Table

## Namespace → Files

| Namespace | Config | Memory | Agents |
|-----------|--------|--------|--------|
| `controller` | `~/.hermes/controller/config.md` | `~/.hermes/controller/memory/` | `~/.hermes/controller/agents/` |
| `engineering` | `~/.hermes/engineering/config.md` | `~/.hermes/engineering/memory/` | `~/.hermes/engineering/agents/` |
| `operations` | `~/.hermes/operations/config.md` | `~/.hermes/operations/memory/` | `~/.hermes/operations/agents/` |
| `finance` | `~/.hermes/finance/config.md` | `~/.hermes/finance/memory/` | `~/.hermes/finance/agents/` |

## Namespace → Head Agent → Tools

| Namespace | Head Agent | Allowed Tools | Account |
|-----------|-----------|---------------|---------|
| `controller` | router | NONE (read-only) | — |
| `engineering` | lead-dev | github, jira, ci-pipeline | eng@company.com |
| `operations` | pm | calendar, jira, docs | ops@company.com |
| `finance` | accountant | sheets, banking, invoicing | fin@company.com |

## Permitted Data Crosses

| Source | Destination | Type | Example |
|--------|-------------|------|---------|
| engineering | finance | `data_cross` | Infrastructure costs as "Engineering" category |
| operations | finance | `data_cross` | Vendor invoices as "Operations" category |
| finance | `*` | `state` | Monthly financial summaries (broadcast) |
| engineering | `*` | `alert` | System incidents (broadcast) |

## Adjacency

```
          controller
         /    |    \
  engineering  operations  finance
        \          |        /
         `-- data_cross ---'
```

Controller sees all. Namespaces communicate only via bus.
Data crosses are one-directional and explicitly permitted above.
