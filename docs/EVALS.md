# Evaluation & guardrail plan

The JD asks directly for "evaluation pipelines to measure agent quality, reliability, and cost — and
iterate until agents are production-trustworthy." Here's what that actually looks like for Portage AI.

## What I'd measure, per agent

| Metric | Lane Agent | Watch Agent | Doc & Comms Agent |
|---|---|---|---|
| **Correctness** | Did it pick a carrier that satisfies every routing rule and stays within the rate-variance threshold? | Did the diagnosed cause match the real one, and was the proposed remediation actually available? | Did the extracted fields match ground truth, and did it correctly flag (or not flag) a discrepancy? |
| **Escalation calibration** | Did it escalate exactly the cases it should — not over- or under-escalating? | Same | Same |
| **Latency** | Time from request to booked or escalated | Time from event to a proposed remediation | Time from document received to a draft being ready |
| **Cost** | Tokens + tool calls per shipment booked | Tokens + tool calls per exception resolved | Tokens + tool calls per document processed |
| **Safety** | Never books outside compliance minimums | Never auto-executes above the cost/risk threshold | Never auto-sends a message that touches cost or liability |

## How I'd build it

1. **Golden scenario sets, per agent, ~20-30 to start.** Hand-built at first, then drawn from
   anonymized real cases once we're in pilot. Each one has an expected outcome — the correct action,
   or the correct escalation and why.
2. **Rule-based checks first.** Compliance violations, threshold breaches, schema validity — these
   are checked deterministically. No judge model needed, no ambiguity to argue about.
3. **LLM-as-judge for the rest.** Diagnosis quality on the Watch Agent, and tone/accuracy of drafted
   comms on the Doc & Comms Agent, get graded by a separate model call against a rubric, spot-checked
   by a human periodically so judge drift doesn't go unnoticed.
4. **A regression gate.** The full eval set runs on every prompt or model change before it ships.
   This is what stops "it got worse when we upgraded the model" from reaching production quietly.
5. **Production shadow metrics.** Once we're piloting, every real approve/reject from a human ops
   user becomes a labeled example — approve means the agent was right, reject or counter means it
   wasn't. This is the highest-signal eval data there is, and it comes for free as a byproduct of the
   HITL design, not as a separate research effort.

## Guardrails as something a non-engineer can actually see

Every threshold in `docs/AGENTS.md` — rate variance %, cost-impact $, discrepancy materiality — is
meant to be a config value an ops lead can see and adjust, not a constant buried in a prompt. The JD
is explicit that non-engineers need to be able to run and monitor these agents safely, and the
prototype's Guardrails panel is my stand-in for what that config surface would look like.

## Model & orchestration choices for this POC

- **Model:** Claude, for all three agents — strong structured tool-use and instruction following,
  and I built this POC with Claude Code, so the model choice matches how I actually built it, not
  just what I'm pitching.
- **Orchestration:** a small, explicit plan → act → observe → escalate-or-continue loop per agent,
  rather than reaching for a heavyweight framework for a three-agent POC. It maps directly onto a
  LangGraph-style state machine if the system needs more complex branching once it's in Next.
- **Context strategy:** each agent only gets the shipment record, the relevant tool outputs, and the
  narrow slice of RAG content relevant to its own task — not the full account history. Keeps cost and
  latency predictable, and keeps irrelevant context from degrading tool-call accuracy.
