---
Author:
    ContentProducer: Siyuan Deng
    ContentPropagator: Siyuan Deng
    University: Imperial College London
---

# Knowledge Hygiene

> A self-maintaining memory management skill for AI agents. Keeps your agent's long-term knowledge fresh, consistent, and free of stale echoes.

---

## What It Does

`knowledge-hygiene` runs automatically at the start of every session to scan a **Knowledge Registry** — a structured markdown table tracking every piece of persistent knowledge the agent holds about itself, its user, and its environment.

It computes a freshness status for each entry:
| Status | Meaning | Used by Agent |
|--------|---------|---------------|
| `ACTIVE` | Fresh, within expiry window | ✅ Yes |
| `STALE` | Past expiry, needs human review | ⚠️ Warning shown |
| `ARCHIVED` | No longer relevant | ❌ Skipped |

Stale entries are flagged so they don't silently mislead the agent — but they're never auto-deleted. A human decides what to keep.

---

## Why It Exists

AI agents accumulate knowledge over time: user preferences, system rules, skill notes, lessons learned. Without active management:

- **Stale information** silently degrades decision quality
- **Contradicting records** create internal confusion  
- **Unused knowledge** keeps loading into context, wasting tokens

Knowledge Hygiene is the answer — a lightweight, zero-dependency discipline that treats memory like a garden: tended regularly, not left to grow wild.

---

## Files

```
knowledge-hygiene/
├── SKILL.md          # Full skill specification
├── hygiene.py        # Core engine (pure stdlib)
└── tests/
    └── test_hygiene_core.py   # Unit tests
```

**Runtime dependency:** Python standard library only (no external packages required).

---

## Entry Points

| Function | Description |
|----------|-------------|
| `scan(quiet=False)` | Scan all registry entries, compute status, return diff |
| `report()` | Human-readable health report |
| `check_entry(entry)` | Compute status for a single entry |
| `parse_expiry(expiry_str)` | Parse "3 / 永久 / 6个月" style expiry |
| `parse_registry(content)` | Parse `KNOWLEDGE_REGISTRY.md` table |

---

## Registry Format

The agent maintains its registry as a markdown table at `memory/KNOWLEDGE_REGISTRY.md`:

```markdown
| ID | Category | Description | LastUpdated | Expiry | Status | Notes |
|----|----------|-------------|-------------|--------|--------|-------|
| KR-001 | 纪律 | D0/D1/D2 执行三定律 | 2026-04-09 | 永久 | ACTIVE | ... |
| KR-021 | 数据 | 今日A股实时数据 | 2026-04-13 | 1天 | ACTIVE | ... |
```

Categories (`Category`): 教训 / 技能 / 架构 / 纪律 / 数据 / 其他

Expiry format: `N` (days) / `N个月` (months) / `永久` (permanent)

---

## Status Lifecycle

```
ACTIVE ──(expiry exceeded)──▶ STALE ──(1+ months old)──▶ ARCHIVED
         ▲                                                         │
         └──────────── (human confirms still valid) ───────────────┘
```

---

## Usage Example

```python
from hygiene import scan, report

# Run at session start (automatic via HEARTBEAT.md or called explicitly)
result = scan(quiet=False)

# result["ok"]          → bool, scan succeeded
# result["stale_ids"]    → list of stale entry IDs
# result["changed"]     → bool, any state changed since last scan

if result["changed"]:
    print(report())
```

---

## Design Principles

1. **Human-in-the-loop** — Nothing is auto-deleted. Stale ≠ useless.
2. **Zero external deps** — Pure Python stdlib. Drop-in install.
3. **Plain text storage** — Markdown registry is human-readable and version-controllable.
4. **Quiet by default** — Silent in normal sessions; only speaks when something needs attention.

---

## Author

Built for [OpenClaw](https://github.com/openclaw/openclaw) / [MaxClaw](https://clawhub.com), used by the Dorian agent.

Originally installed at `workspace/skills/knowledge-hygiene/`.
