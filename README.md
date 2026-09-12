# Portage AI

An agentic operations layer for freight & logistics teams — my submission for the Teoh Capital /
SIMBA case study.

Dijae · dijaedadula@gmail.com

**👉 [Case Study Hub](https://hub-kappa-two.vercel.app)** — one dashboard with everything below:
the interactive prototype, the SIMBA write-up, and a presentable version of the deck, all in one
place. Access-code protected; if you're the hiring team, use the code I sent by email.

- 📊 **Scoping pack (PPT):** `scoping-pack/Portage-AI-Scoping-Pack.pptx`
- 🖥️ **Interactive prototype:** `prototype/index.html` (open directly in a browser — no server or
  install needed, see [Running the prototype](#running-the-prototype))
- 🗺️ **Product roadmap:** [`docs/ROADMAP.md`](docs/ROADMAP.md)
- 🤖 **Agent architecture & the top 3 agentic functions:** [`docs/AGENTS.md`](docs/AGENTS.md)
- ✅ **Evaluation & guardrail plan:** [`docs/EVALS.md`](docs/EVALS.md)

**Part 2** — the SIMBA mobile app write-up — is a shorter, separate piece:
[`part-2-simba-mobile-app/README.md`](part-2-simba-mobile-app/README.md).

---

## Why I scoped it this way

The brief hands you a domain — an AI-first agentic logistics app — but the real test is open-ended:
can you take something that broad and narrow it to a system a business would actually trust running
on its own? The JD is specific about what "trust" means to this team: tool use, planning, retrieval,
guardrails, human-in-the-loop checkpoints, evals, and integration with real systems.

So rather than build a generic shipment tracker with a chatbot layered on top, I scoped this down to
one sharp slice — a **freight/parcel operations copilot** for a small logistics team (a 3PL, a
freight forwarder, or an internal logistics desk). It's the fastest way to demonstrate everything the
JD is actually testing for in one coherent system, instead of a handful of shallow features spread
across "logistics" in general.

## What it is

Portage AI sits on top of a logistics team's existing stack — TMS, carrier APIs, customs and policy
documentation, inbox — and runs three agents across a shipment's lifecycle: booking it, watching it
in transit, and closing it out. A human stays the approver for anything costly or risky; the agents
own everything routine. The full breakdown of each agent — its tools, memory, planning loop, and
exactly when it hands off to a person — is in [`docs/AGENTS.md`](docs/AGENTS.md).

## Running the prototype

No build step, no dependencies, no API keys. It's a single static HTML file with mock data and a
simulated agent trace (see `docs/AGENTS.md` for what each simulated tool call maps to in a real
integration).

```bash
# from the repo root
open prototype/index.html      # macOS
start prototype/index.html     # Windows
```

Or just double-click `prototype/index.html`.

A hosted version — a shareable link, nothing to download — is in the scoping pack and in the
submission email.

## Repo structure

```
docs/
  ROADMAP.md         product roadmap (Now / Next / Later) with the exit criteria for each phase
  AGENTS.md           the 3 agentic functions: what each does, tools/memory/planning/guardrails
  EVALS.md            how I'd measure agent quality, reliability, and cost before and after launch
  ARCHITECTURE.md     system diagram - orchestration, tool layer, memory, HITL, observability
prototype/
  index.html          interactive prototype (single file, mock data, no backend)
scoping-pack/
  Portage-AI-Scoping-Pack.pptx
```

## Prototype vs. production

Every "tool call" in the prototype — rate lookups, carrier booking, tracking webhooks, document
extraction — is mocked with realistic fixture data, so the agent reasoning and the human-in-the-loop
flow can be judged without any real integrations or API keys sitting behind them. `docs/AGENTS.md`
and `docs/ARCHITECTURE.md` spell out exactly which real system each mock stands in for — the "rate
lookup" tool is a TMS/carrier-rate API call, "extract document" is a structured-output call against
an uploaded BOL or POD — and what would actually change to take this from a POC to a pilot.
