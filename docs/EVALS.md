# Evaluation & guardrail plan

The JD asks explicitly for "evaluation pipelines to measure agent quality, reliability, and cost —
and iterate until agents are production-trustworthy." This is what that looks like for Portage AI.

## What gets measured, per agent

| Metric | Lane Agent | Watch Agent | Doc & Comms Agent |
|---|---|---|---|
| **Correctness** | Did it pick a carrier that satisfies all routing rules and is within the rate-variance threshold? | Did the diagnosed cause match the real cause, and was the proposed remediation actually available? | Did extracted fields match ground truth, and did it correctly flag/not-flag a discrepancy? |
| **Escalation calibration** | Did it escalate exactly the cases it should (not over- or under-escalating)? | Same | Same |
| **Latency** | Time from request to booked/escalated | Time from event to remediation proposed | Time from document received to draft ready |
| **Cost** | Tokens + tool calls per shipment booked | Tokens + tool calls per exception resolved | Tokens + tool calls per document processed |
| **Safety** | Never books outside compliance minimums | Never auto-executes above the cost/risk threshold | Never auto-sends a cost/liability-impacting message |

## How it's built

1. **Golden scenario sets (per agent, ~20–30 to start):** hand-built and drawn from anonymized
   real cases once in pilot. Each scenario has an expected outcome (correct action, or correct
   escalation) and, for escalation cases, the expected reason.
2. **Rule-based checks first:** compliance violations, threshold breaches, and schema validity are
   checked deterministically — no LLM judge needed, no ambiguity.
3. **LLM-as-judge for the rest:** correctness of diagnosis (Watch Agent) and tone/accuracy of
   drafted customer comms (Doc & Comms Agent) are graded by a separate model call against a rubric,
   spot-checked by a human periodically to catch judge drift.
4. **Regression gate:** the full eval set runs on every prompt or model change before it ships —
   this is what prevents "it got worse when we upgraded the model" from reaching production
   silently.
5. **Production shadow metrics:** once piloting, every real approve/reject decision from a human
   ops user becomes a labeled example — approve = agent was right, reject/counter = agent was
   wrong. This is the highest-signal eval data and it's collected for free as a byproduct of the
   HITL design, not a separate research effort.

## Guardrails as a first-class, non-engineer-legible thing

Every threshold referenced in `docs/AGENTS.md` (rate variance %, cost-impact $, discrepancy
materiality) is designed to be a **config value an ops lead can see and adjust**, not a constant
buried in a prompt — because the JD is explicit that non-engineers need to safely run and monitor
these agents. The prototype's "Guardrails" panel is a stand-in for that config surface.

## Model & orchestration choices for this POC

- **Model:** Claude (Anthropic) for all three agents — strong structured tool-use and instruction
  following, and this POC is being built with Claude Code, so the model choice and the build
  process are consistent with how the role expects agents to be built day to day.
- **Orchestration:** a small explicit plan → act → observe → escalate-or-continue loop per agent
  rather than a heavyweight framework for a 3-agent POC — the same pattern maps directly onto
  LangGraph-style state machines if/when the system needs more complex branching in the Next phase.
- **Context strategy:** each agent gets only the shipment record + relevant tool outputs + the
  narrow slice of RAG content relevant to its task, not the full account history — keeps cost and
  latency predictable and avoids irrelevant context degrading tool-call accuracy.
