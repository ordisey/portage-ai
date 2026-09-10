# Product roadmap — Portage AI

Three phases, each gated by evidence from the phase before it rather than a fixed calendar date —
consistent with "a working prototype this week beats a perfect design next month."

## Now — Prove the agents work on real logistics logic (this POC)

**Goal:** demonstrate that the three agents (Lane, Watch, Doc & Comms) make correct, explainable
decisions on realistic shipment scenarios, with human-in-the-loop guardrails, before touching any
live system.

- Ship the interactive prototype (mock tools, fixture data, full HITL approval flow)
- Define the tool contracts for all three agents (`docs/AGENTS.md`) so real integrations can be
  swapped in without changing agent logic
- Stand up a first eval set: ~20–30 golden shipment scenarios per agent covering the happy path,
  the escalation path, and known edge cases (`docs/EVALS.md`)
- Get the approval-queue UX in front of 2–3 real ops users (even against the mock) to validate the
  interaction pattern before wiring real data behind it

**Exit criteria:** eval pass rate and reviewer feedback are good enough to justify a real pilot,
and the approval-queue UX is one ops people actually trust to act quickly on.

## Next — Pilot on one real lane with one design partner

**Goal:** replace mocked tools with real integrations for a narrow, contained slice of real
traffic, with a human approving every agent action.

- Wire `lookup_carrier_rates` / `book_shipment` to a real TMS or carrier API for one lane
- Wire `get_tracking_events` to a real carrier/EDI webhook feed
- Wire `extract_document_fields` to real BOL/POD uploads (structured-output extraction, not OCR
  from scratch)
- Every agent action still requires human approval, regardless of the guardrail thresholds in
  `docs/AGENTS.md` — the thresholds get *tuned* against real approve/reject decisions in this phase,
  not trusted yet
- Expand the eval set with real (anonymized) production scenarios and start tracking cost and
  latency per agent, not just accuracy
- Build the audit log into something a non-engineer ops lead can actually read and act on

**Exit criteria:** the agents' recommendations match what a human ops lead would have chosen on
>90% of real cases in the pilot lane, cost/latency are within budget, and the design partner wants
to keep using it.

## Later — Expand autonomy and surface area

**Goal:** let the system act autonomously on low-risk decisions (per the tuned guardrails from
Next), and widen coverage from one lane to the full book of business.

- Turn on auto-execution for the guardrail-cleared paths identified in the pilot (e.g. routine
  on-schedule updates, low-cost reroutes) — everything else stays human-approved
- Add a 4th surface only if the first three justify it: a **negotiation/rate-shopping agent** that
  proactively re-shops underperforming lanes, rather than only reacting to individual shipments
- Expand from one design-partner lane to the full network; add per-account guardrail tuning instead
  of one global threshold
- Formalize the eval pipeline into a standing regression suite that runs on every prompt/model
  change (see `docs/EVALS.md`) so autonomy doesn't quietly regress as the system evolves

**Exit criteria:** defined per surface as it's turned on — this phase is explicitly paced by trust
earned in Next, not by a target date.

## What's deliberately out of scope

- Full ERP/WMS replacement — Portage AI sits alongside existing systems, it doesn't replace them
- Autonomous financial actions (payments, claims, contract commitments) — always human-approved,
  indefinitely, regardless of phase
- A generic multi-vertical agent platform — this roadmap stays logistics-specific; horizontal
  platform bets are a distraction until one vertical is proven
