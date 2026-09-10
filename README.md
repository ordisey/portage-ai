# Portage AI — an agentic operations layer for freight & logistics teams

**Case study submission — AI Agent Engineer, Teoh Capital / SIMBA**
Author: Dijae · dijae0@gmail.com

Part 1 of the case study asked for a scoped, agentic logistics POC: a product roadmap, the top 3
agentic functions, a scoping pack, and an interactive prototype. This repo is that submission.

- 📊 **Scoping pack (PPT):** `scoping-pack/Portage-AI-Scoping-Pack.pptx`
- 🖥️ **Interactive prototype:** `prototype/index.html` (open directly in a browser, no server or
  install required — see [Running the prototype](#running-the-prototype))
- 🗺️ **Product roadmap:** [`docs/ROADMAP.md`](docs/ROADMAP.md)
- 🤖 **Agent architecture & the top 3 agentic functions:** [`docs/AGENTS.md`](docs/AGENTS.md)
- ✅ **Evaluation & guardrail plan:** [`docs/EVALS.md`](docs/EVALS.md)

**Part 2** (Singapore telco mobile app for SIMBA — top 3 features, rationale, and prioritization)
is a short, separate write-up: [`part-2-simba-mobile-app/README.md`](part-2-simba-mobile-app/README.md).

---

## Why this scope

The case study fixes the domain (an "AI-first agentic logistics app"), but "logistics" is broad
enough to hide the real ask, which is really: *can this person scope an ambiguous problem into a
small set of agents that a business would actually trust in production?* The JD is explicit about
what "trust" means here — tool use, planning, RAG, guardrails, human-in-the-loop checkpoints,
evals, and integration with real systems.

So instead of a generic "track my package" app, this POC narrows to a **freight/parcel operations
copilot** for a small logistics team (3PL, freight forwarder, or an internal logistics desk) —
because that's the sharpest vehicle to demonstrate each of those things in one small system,
rather than a shipment-tracker with a chatbot bolted on.

## The product: Portage AI

Portage AI sits on top of a logistics team's existing systems (TMS, carrier APIs, customs/policy
docs, inbox) and runs three agents across the life of a shipment: booking it, watching it in
transit, and closing it out. A human stays the approver for anything costly or risky; the agents
own everything routine. See [`docs/AGENTS.md`](docs/AGENTS.md) for the full breakdown of each
agent's tools, memory, planning loop, and guardrails.

## Running the prototype

No build step, no dependencies, no API keys. It's a single static HTML file with mock data and a
simulated agent trace (see `docs/AGENTS.md` for how the simulated calls map to what would be real
tool calls in production).

```bash
# from the repo root
open prototype/index.html      # macOS
start prototype/index.html     # Windows
```

Or just double-click `prototype/index.html` in Explorer/Finder.

A hosted version (shareable link, no download needed) is linked in the scoping pack and in the
submission email.

## Repo structure

```
docs/
  ROADMAP.md         product roadmap (Now / Next / Later) with success metrics per phase
  AGENTS.md           the 3 agentic functions: what each does, tools/memory/planning/guardrails
  EVALS.md            how agent quality, reliability, and cost would be measured pre- and post-launch
  ARCHITECTURE.md     system diagram — orchestration, tool layer, memory, HITL, observability
prototype/
  index.html          interactive prototype (single file, mock data, no backend)
scoping-pack/
  Portage-AI-Scoping-Pack.pptx
```

## Notes on the prototype vs. production

Every "tool call" in the prototype (rate lookups, carrier booking, tracking webhooks, document
extraction) is mocked with realistic fixture data so the agent reasoning and HITL flow can be
evaluated without any real integrations or API keys. `docs/AGENTS.md` and `docs/ARCHITECTURE.md`
spell out exactly which real system each mock stands in for (e.g. the "rate lookup" tool would be
a TMS/carrier-rate API call; the "extract document" tool would be a structured-output call against
an uploaded BOL/POD) and what would change to take this from POC to pilot.
