# The top 3 agentic functions

Portage AI runs three agents across a shipment's lifecycle. They share one orchestration layer, one
shipment memory store, and one human-approval queue — I didn't want three separate chatbots that
happen to sit in the same product.

Each section below follows the structure the JD asks for directly: tool use, memory, planning,
retrieval, guardrails, and human-in-the-loop checkpoints.

---

## 1. Lane Agent — booking & carrier selection

**Job:** turn a shipment request — a structured form, an email, a forwarded PO — into a booked,
carrier-confirmed shipment, at the best available rate that satisfies the account's routing rules.

| Dimension | Design |
|---|---|
| **Trigger** | New shipment request lands (form submission, an inbound email parsed to structured intent, or an API call from an upstream ERP) |
| **Planning** | Parse the request → identify candidate lanes/carriers → pull live rates → apply routing rules (preferred carrier, insurance minimums, transit-time SLA) → rank options → select or escalate |
| **Tools** | `get_shipment_details`, `lookup_carrier_rates` (TMS/carrier-rate API), `check_carrier_compliance` (insurance/certifications), `book_shipment` (TMS write), `notify_customer` |
| **Memory** | Short-term: the shipment record it's building. Long-term: customer/lane preferences learned over time — e.g. "this account always excludes Carrier X after a claim" |
| **Retrieval** | RAG over the account's routing-rules doc and carrier contracts, for the rules that aren't captured as structured data — "no owner-operators for reefer freight over $50k," that sort of thing |
| **Guardrails** | Hard stop if no carrier meets compliance minimums. A rate-variance threshold — auto-books within policy, escalates if the cheapest compliant option is more than 15% above the lane's trailing average |
| **Human-in-the-loop** | Auto-books routine shipments. Routes to an ops approval queue — one-click approve/counter/reject — when the threshold above is crossed, or the customer account is flagged high-touch |

I'd call this agentic rather than a form because "best carrier" isn't a lookup — it's a constrained
optimization over live rates, compliance, and account-specific rules that change per shipment. And
the agent has to know when its own confidence is too low to decide alone.

---

## 2. Watch Agent — exception & delay resolution

**Job:** watch in-transit shipments continuously, catch problems before the customer does, and
resolve or escalate them.

| Dimension | Design |
|---|---|
| **Trigger** | Tracking/EDI webhook event — delay, customs hold, temperature excursion, failed delivery attempt — or a missed-milestone check (no scan in the expected window) |
| **Planning** | Detect the anomaly → pull shipment and event context → diagnose the likely cause → generate one or two remediation options with cost/time tradeoffs → act or escalate |
| **Tools** | `get_tracking_events`, `get_shipment_details`, `lookup_alt_routing` (rebook/reroute options), `estimate_cost_impact`, `execute_remediation` (rebook/expedite), `create_ops_ticket`, `notify_customer` |
| **Memory** | Shipment history — has this lane been delay-prone before? — plus a running memory of the specific exception across multiple events, since a customs hold might update twice before it resolves |
| **Retrieval** | RAG over customs/regulatory playbooks and carrier SLA docs, to interpret exception codes and know what recourse actually exists for a given carrier and lane |
| **Guardrails** | A cost-impact threshold for auto-remediation — auto-approves a reroute under $250 extra cost and less than a day added transit; anything above that needs a human. It never auto-cancels or auto-claims without one |
| **Human-in-the-loop** | Low-risk fixes execute automatically with a logged rationale. High-cost or high-risk exceptions land in the approval queue with the two ranked options, the cost/time deltas, and a one-click approve/deny |

The interesting work here isn't detecting the delay — that's a webhook firing. It's diagnosing why,
ranking the real remediation options against cost, time, and customer impact, and knowing which of
those calls it's actually allowed to make on its own.

---

## 3. Doc & Comms Agent — document intelligence + customer communication

**Job:** validate shipment paperwork against the shipment record, catch discrepancies before they
turn into invoicing or customs problems, and keep the customer proactively informed in plain
language.

| Dimension | Design |
|---|---|
| **Trigger** | A document arrives — BOL, POD, commercial invoice, customs declaration — via upload, an inbound email attachment, or a carrier API |
| **Planning** | Extract structured fields from the document → reconcile against the shipment record → flag or resolve discrepancies → decide what, if anything, needs to reach the customer → draft (and, once it's earned trust, send) the update |
| **Tools** | `extract_document_fields` (structured extraction), `get_shipment_details`, `diff_against_record`, `draft_customer_update`, `send_customer_update`, `flag_billing_discrepancy` |
| **Memory** | The shipment's full document trail, so it can reconcile a POD against the original BOL and the booking record — not just the one document in front of it |
| **Retrieval** | RAG over the account's tone and comms history, so a drafted update matches how that account is normally spoken to |
| **Guardrails** | Never auto-sends a customer-facing message about a cost or liability change without human sign-off. Weight/value discrepancies past a materiality threshold get flagged, not silently corrected |
| **Human-in-the-loop** | Every customer-facing draft is shown for edit/approve before it sends, early on. Once accuracy holds up in evals (`docs/EVALS.md`), the routine "your shipment is on schedule" updates can graduate to auto-send — anything involving a discrepancy or delay stays human-approved indefinitely |

The hard part here isn't reading the document — it's reconciling it against everything already known
about the shipment, and deciding, case by case, whether a human needs to see it before the customer
does.

---

## Shared infrastructure (not a fourth agent, but it's load-bearing)

- **Orchestrator** — a small graph (plan → act → observe → escalate-or-continue) shared by all three
  agents, so tool definitions, memory access, and the approval queue stay consistent instead of being
  reimplemented per agent.
- **Shipment memory store** — one source of truth per shipment that all three agents read and write
  to, so something the Doc Agent finds is already visible context for the Watch Agent's next
  diagnosis.
- **Human approval queue** — a single inbox for every agent's escalations, with the option, the
  rationale, and the cost/risk delta presented the same way each time, so ops can clear it in
  seconds instead of context-switching between three different views.
- **Audit log** — every action, auto or approved, is logged with its inputs, tool calls, and
  rationale, so a non-engineer can reconstruct why the system did what it did. That's the JD's own
  bar: document agent behavior and failure modes so non-engineers can run and monitor this safely.
