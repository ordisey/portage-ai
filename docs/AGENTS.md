# The top 3 agentic functions

Portage AI runs three agents across a shipment's lifecycle. They share one orchestration layer,
one shipment memory store, and one human-approval queue — they are not three separate chatbots.

Each section below follows the same structure the JD asks for explicitly: **tool use, memory,
planning, retrieval, guardrails, and human-in-the-loop checkpoints.**

---

## 1. Lane Agent — booking & carrier selection

**Job:** turn a shipment request (structured form, email, or forwarded PO) into a booked,
carrier-confirmed shipment, at the best available rate that satisfies the account's routing rules.

| Dimension | Design |
|---|---|
| **Trigger** | New shipment request lands (form submission, inbound email parsed to structured intent, or API call from an upstream ERP) |
| **Planning** | Multi-step plan: parse request → identify candidate lanes/carriers → pull live rates → apply routing rules (preferred carrier, insurance minimums, transit-time SLA) → rank options → select or escalate |
| **Tools** | `get_shipment_details`, `lookup_carrier_rates` (TMS/carrier-rate API), `check_carrier_compliance` (insurance/certifications), `book_shipment` (TMS write), `notify_customer` |
| **Memory** | Short-term: the shipment record being built. Long-term: customer/lane preferences learned over time (e.g. "this account always excludes Carrier X after a claim") |
| **Retrieval** | RAG over the account's routing-rules doc and carrier contracts to resolve rules not captured as structured data (e.g. "no owner-operators for reefer freight over $50k") |
| **Guardrails** | Hard stop if no carrier meets compliance minimums. Rate-variance threshold — auto-book within policy; escalate if the cheapest compliant option is >15% above the lane's trailing average |
| **Human-in-the-loop** | Auto-books routine shipments; routes to an ops approval queue (one-click approve/counter/reject) when the threshold above is crossed, or the customer account is flagged high-touch |

**Why it's agentic and not a form:** the "best" carrier isn't a lookup, it's a constrained
optimization over live rates, compliance, and account-specific rules that change per shipment —
and the agent has to know when its own confidence is too low to act alone.

---

## 2. Watch Agent — exception & delay resolution

**Job:** continuously monitor in-transit shipments, catch problems before the customer does, and
resolve or escalate them.

| Dimension | Design |
|---|---|
| **Trigger** | Tracking/EDI webhook event (delay, customs hold, temperature excursion, failed delivery attempt), or a missed-milestone check (no scan in expected window) |
| **Planning** | Detect anomaly → pull shipment + event context → diagnose likely cause → generate 1–2 remediation options with cost/time tradeoffs → act or escalate |
| **Tools** | `get_tracking_events`, `get_shipment_details`, `lookup_alt_routing` (rebook/reroute options), `estimate_cost_impact`, `execute_remediation` (rebook/expedite), `create_ops_ticket`, `notify_customer` |
| **Memory** | Shipment history (has this lane been delay-prone before?), and running memory of the specific exception across multiple events (e.g. a customs hold that updates twice before resolving) |
| **Retrieval** | RAG over customs/regulatory playbooks and carrier SLA docs to interpret exception codes and know what recourse actually exists for a given carrier/lane |
| **Guardrails** | Cost-impact threshold for auto-remediation (e.g. auto-approve a reroute under $250 extra cost and <1 day added transit); anything above requires human approval. Never auto-cancels or auto-claims without a human |
| **Human-in-the-loop** | Low-risk fixes execute automatically with a logged rationale; high-cost/high-risk exceptions surface in the approval queue with the two ranked options, cost/time deltas, and one-click approve/deny |

**Why it's agentic and not an alert:** the interesting work isn't detecting the delay (that's a
webhook), it's diagnosing *why*, ranking real remediation options against cost/time/customer
impact, and knowing which of those decisions it's allowed to make on its own.

---

## 3. Doc & Comms Agent — document intelligence + customer communication

**Job:** validate shipment paperwork against the shipment record, catch discrepancies before
they become invoicing/customs problems, and keep the customer proactively informed in plain
language.

| Dimension | Design |
|---|---|
| **Trigger** | A document arrives (BOL, POD, commercial invoice, customs declaration) — via upload, inbound email attachment, or carrier API |
| **Planning** | Extract structured fields from the document → reconcile against the shipment record → flag/resolve discrepancies → decide what, if anything, needs to be communicated to the customer → draft (and, once trusted, send) the update |
| **Tools** | `extract_document_fields` (structured extraction), `get_shipment_details`, `diff_against_record`, `draft_customer_update`, `send_customer_update`, `flag_billing_discrepancy` |
| **Memory** | The shipment's full document trail, so it can reconcile a POD against the original BOL and the booking record, not just the single document in front of it |
| **Retrieval** | RAG over the account's tone/comms preferences and prior correspondence, so drafted updates match how that account is normally communicated with |
| **Guardrails** | Never auto-sends a customer-facing message about a cost or liability change without human sign-off. Weight/value discrepancies above a materiality threshold are flagged, not silently corrected |
| **Human-in-the-loop** | Every customer-facing draft is shown for edit/approve before send in the early phases; once accuracy is proven in evals (see `EVALS.md`), routine "your shipment is on schedule" updates graduate to auto-send while anything involving a discrepancy or delay stays human-approved indefinitely |

**Why it's agentic and not OCR:** the hard part isn't reading the document, it's reconciling it
against everything already known about the shipment and deciding, case by case, whether a human
needs to be in the loop before the customer hears about it.

---

## Shared infrastructure (not a 4th agent, but load-bearing)

- **Orchestrator** — a small graph (plan → act → observe → escalate-or-continue) shared by all
  three agents, so tool definitions, memory access, and the approval queue are consistent rather
  than reimplemented per agent.
- **Shipment memory store** — one source of truth per shipment that all three agents read/write
  to, so the Doc Agent's findings are visible to the Watch Agent and vice versa.
- **Human approval queue** — a single inbox for all three agents' escalations, with the option
  presented, the rationale, and the cost/risk delta, so an ops person can clear it in seconds.
- **Audit log** — every agent action (auto or approved) is logged with its inputs, tool calls, and
  rationale, so a non-engineer can reconstruct *why* the system did what it did (see `EVALS.md` and
  the JD requirement to "document agent behavior, failure modes, and operating procedures").
