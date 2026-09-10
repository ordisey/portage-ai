# Product roadmap — Portage AI

Three phases, each gated by evidence from the one before it rather than a fixed date on a calendar.
That's deliberate — a working prototype this week beats a perfect design next month, and I'd rather
earn the right to automate more than promise it upfront.

## Now — prove the agents actually work (this POC)

The goal here is narrow: show that the three agents — Lane, Watch, Doc & Comms — make correct,
explainable decisions on realistic shipment scenarios, with a human in the loop, before any of them
touch a live system.

- Ship the interactive prototype — mocked tools, fixture data, full human-approval flow
- Define the tool contracts for all three agents (`docs/AGENTS.md`), so real integrations can be
  swapped in later without touching the agent logic itself
- Build a first eval set — roughly 20-30 golden shipment scenarios per agent, covering the happy
  path, the escalation path, and the edge cases I already know about
- Put the approval-queue UX in front of two or three real ops people, even against the mock, to
  validate the interaction before wiring real data behind it

**Exit criteria:** the eval results and the reviewer feedback are strong enough to justify a real
pilot, and the approval queue is something an ops person would actually trust to act on quickly.

## Next — pilot on one real lane with one design partner

Here the goal shifts to replacing mocked tools with real integrations, on a narrow, contained slice
of real traffic, with a human still approving every agent action.

- Wire `lookup_carrier_rates` / `book_shipment` to a real TMS or carrier API, one lane at a time
- Wire `get_tracking_events` to a real carrier or EDI webhook feed
- Wire `extract_document_fields` to real BOL/POD uploads — structured extraction, not OCR built from
  scratch
- Every agent action still needs human sign-off, regardless of the guardrail thresholds in
  `docs/AGENTS.md`. Those thresholds get tuned against real approve/reject decisions in this phase;
  I'm not trusting them yet
- Expand the eval set with real, anonymized production scenarios, and start tracking cost and
  latency alongside accuracy
- Turn the audit log into something a non-engineer ops lead can actually read and act on, not just a
  debugging tool for me

**Exit criteria:** the agents' recommendations match what a human ops lead would have chosen on over
90% of real cases in the pilot lane, cost and latency stay within budget, and the design partner
wants to keep using it.

## Later — expand autonomy and surface area

Once Next has earned some trust, this phase lets the system act on its own for the low-risk
decisions that phase proved out, and widens from one lane to the full book of business.

- Turn on auto-execution for whichever guardrail-cleared paths the pilot actually validated —
  routine on-schedule updates, low-cost reroutes. Everything else stays human-approved
- Add a fourth surface only if the first three earn it — a negotiation/rate-shopping agent that
  proactively re-shops underperforming lanes, instead of only reacting shipment by shipment
- Expand from one design-partner lane to the full network, with per-account guardrail tuning instead
  of a single global threshold
- Formalize the eval pipeline into a standing regression suite that runs on every prompt or model
  change, so autonomy doesn't quietly regress as the system evolves

**Exit criteria:** defined per surface as it's turned on. This phase is paced by trust earned in
Next, not by a date on a roadmap slide.

## What I'm deliberately leaving out

- A full ERP/WMS replacement — Portage AI sits alongside a team's existing systems, it doesn't
  replace them
- Any autonomous financial action — payments, claims, contract commitments — stays human-approved
  indefinitely, regardless of phase
- A generic multi-vertical agent platform — this roadmap stays logistics-specific. A horizontal
  platform bet is a distraction until one vertical is actually proven
