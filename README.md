# bobsql

BobSQL is not a SQL server, it's just another Bob.
```
┌─────────────────────────────────────────────────────────────────┐
│                              BobSQL                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐    ┌──────────────┐    ┌─────────────────┐    │
│   │ TDS Server  │────│  tsqlparser  │────│  Execution      │    │
│   │ (Protocol)  │    │  (Parse SQL) │    │  Engine         │    │
│   └─────────────┘    └──────────────┘    └─────────┬───────┘    │
│                                                    │            │
│                              ┌─────────────────────┴──────────┐ │
│                              │         HOT PATH?              │ │
│                              └─────────────────────┬──────────┘ │
│                                                    │            │
│                    ┌───────────────┬───────────────┤            │
│                    ▼               ▼               ▼            │
│             ┌────────────┐  ┌────────────┐  ┌────────────┐      │
│             │ Interpreted│  │  tgpiler   │  │  Compiled  │      │
│             │   (cold)   │  │ (compile)  │──│   (hot)    │      │
│             └─────┬──────┘  └────────────┘  └─────┬──────┘      │
│                   │                               │             │
│                   └───────────────┬───────────────┘             │
│                                   ▼                             │
│                         ┌─────────────────┐                     │
│                         │ tgpiler Adapters│                     │
│                         └────────┬────────┘                     │
│                                  │                              │
│                   ┌──────────────┼──────────────┐               │
│                   ▼              ▼              ▼               │
│              ┌────────┐    ┌────────┐     ┌─────┐               │
│              │ SQLite │    │Postgres│     │MySQL│               │
│              └────────┘    └────────┘     └─────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        BobSQL Cluster                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ┌──────────────────┐                         │
│                    │   Raft (light)   │                         │
│                    ├──────────────────┤                         │
│                    │ • Leader election│                         │
│                    │ • Membership     │                         │
│                    │ • Discovery      │                         │
│                    │ • Schema DDL     │                         │
│                    │ • Epoch/term     │                         │
│                    └────────┬─────────┘                         │
│                             │                                   │
│              "Who's leader? Who's alive?"                       │
│                             │                                   │
│         ┌───────────────────┼───────────────────┐               │
│         ▼                   ▼                   ▼               │
│   ┌──────────┐        ┌──────────┐        ┌──────────┐          │
│   │  Node 1  │        │  Node 2  │        │  Node 3  │          │
│   │ Postgres │        │  MySQL   │        │  SQLite  │          │
│   └──────────┘        └──────────┘        └──────────┘          │
│         │                   │                   │               │
│         └───────────────────┼───────────────────┘               │
│                             │                                   │
│                     DXP for data ops                            │
│                             │                                   │
│              ┌──────────────┴──────────────┐                    │
│              │  0.5PS  1PS  2PS  3PS       │                    │
│              │  (per-operation selection)  │                    │
│              └─────────────────────────────┘                    │
│               https://github.com/ha1tch/dxp                     │
└─────────────────────────────────────────────────────────────────┘

```
Raft carries kilobytes (membership, leader, schema). DXP carries gigabytes (data operations).

**Separation of concerns:**

| Layer | Protocol | Traffic | Frequency |
|-------|----------|---------|-----------|
| Control plane | Raft | Tiny | Rare |
| Data plane | DXP patterns | Massive | Constant |

hashicorp/raft is the right choice for the control plane. It's proven, it's Go, it handles the boring stuff. You just don't route data through it.
