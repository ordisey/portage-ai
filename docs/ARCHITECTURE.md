# System architecture

```mermaid
flowchart TB
    subgraph Triggers
        A1[New shipment request]
        A2[Tracking / EDI webhook event]
        A3[Document received: BOL / POD / invoice]
    end

    subgraph Orchestrator["Orchestrator — plan / act / observe / escalate-or-continue"]
        LA[Lane Agent]
        WA[Watch Agent]
        DA[Doc & Comms Agent]
    end

    subgraph Tools["Tool layer (mocked in this POC, real APIs in the pilot)"]
        T1[Carrier rate & booking API]
        T2[Tracking / EDI feed]
        T3[Document extraction]
        T4[Customs & policy RAG store]
        T5[Customer notification]
    end

    subgraph Shared["Shared state"]
        MEM[(Shipment memory store)]
        LOG[(Audit log)]
        Q[Human approval queue]
    end

    A1 --> LA
    A2 --> WA
    A3 --> DA

    LA <--> T1
    LA <--> T4
    WA <--> T2
    WA <--> T4
    DA <--> T3
    DA <--> T4
    LA & WA & DA <--> T5

    LA <--> MEM
    WA <--> MEM
    DA <--> MEM

    LA -. escalate .-> Q
    WA -. escalate .-> Q
    DA -. escalate .-> Q
    Q -- human decision --> LA & WA & DA

    LA --> LOG
    WA --> LOG
    DA --> LOG
    Q --> LOG
```

## How I thought about it

1. **One orchestration pattern, three agents.** Each one runs the same
   plan → act → observe → escalate-or-continue loop, with its own tools and guardrail thresholds.
   I wanted a reviewer — or whoever inherits this after me — to only have to understand the pattern
   once, not three times.
2. **Shared memory, not siloed agents.** All three read and write the same per-shipment record. A
   discrepancy the Doc Agent finds is already context for the Watch Agent's next diagnosis. They're
   one system watching one shipment, not three bots that happen to share a database.
3. **One approval queue.** Every escalation from every agent lands in the same human queue, in the
   same shape — option(s), rationale, cost/risk delta — so an ops person isn't switching between
   three different tools to do their job.
4. **Everything is logged before it's autonomous.** The audit log isn't an afterthought bolted on
   before launch. It's how thresholds get tuned in the Next phase, and how a non-engineer can verify
   the system is doing what it's supposed to (see `docs/EVALS.md`).
5. **Mocked tools have a named real-world counterpart.** Nothing in the prototype is a capability I
   invented for the demo. Every mocked tool call in `prototype/index.html` maps to a specific real
   integration point listed in `docs/AGENTS.md`, so getting from this POC to a pilot is a list of
   integrations, not a redesign.

## What changes between the POC and a pilot

| Layer | POC (this repo) | Pilot (Next phase) |
|---|---|---|
| Tools | Fixture JSON, deterministic | Real TMS/carrier/EDI APIs |
| Memory | In-browser mock state | Persistent shipment DB |
| RAG | Static excerpts | Real vector store over the account's actual policy/customs docs |
| Guardrail thresholds | Illustrative defaults | Tuned against real human approve/reject decisions |
| Auto-execution | None — every action is shown, nothing runs silently | Only guardrail-cleared paths, per `docs/ROADMAP.md` |
