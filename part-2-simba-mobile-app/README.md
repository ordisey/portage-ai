# Part 2 — Singapore Telco Mobile App (SIMBA)

**Top 3 features, with rationale and prioritization.** Kept deliberately short — the brief asks
for a quick read on product thinking, not a second POC.

## Context

SIMBA's proposed acquisition of M1 didn't go through — IMDA suspended its review in May 2026 amid
a spectrum-licensing investigation, and the deal was terminated. That leaves SIMBA where it's
actually been most effective: a lean, digital-first challenger competing on price, transparency,
and simplicity rather than network scale. That's the lens below — not a merged-entity roadmap, but
what a standalone challenger's app needs to do well.

## 1. Usage & billing you can trust at a glance — **P0**

Real-time data/talktime/rollover balance, proactive low-balance and bill-change alerts, and a
plain-language breakdown of every charge.

**Why:** SIMBA's entire value proposition — no contract, no hidden fees, free 6-cycle rollover —
is a trust claim. If the app doesn't make that promise instantly legible (current My SIMBA reviews
mention it can be slow to load and has crashed), the brand's core differentiator is invisible at
the exact moment it matters: the moment someone checks whether they're about to pay more. This is
the cheapest, most foundational way to reinforce trust — it's data the company already has, just
not yet legible.

**Priority:** first. It protects existing revenue (retention/churn) and requires no network or
retail investment — only exposing data SIMBA already has, reliably.

## 2. Instant self-serve for everything — **P0 / P1**

One-tap eSIM activation and porting, in-app plan switching, and simple multi-line/family
management — no store visit, no call.

**Why:** SIMBA has no retail footprint like Singtel or StarHub — the app *is* the storefront.
Every friction point in it (a plan change that needs a call, an eSIM activation that isn't
instant) either sends the customer to a competitor with a shop, or into a support queue that costs
more per interaction than a lean challenger can absorb. It's also the highest-leverage growth
channel: "switch to SIMBA, it took three minutes" is the acquisition engine for a no-storefront
brand.

**Priority:** build in parallel with #1, ranked second only because the scope is larger (eSIM and
porting integration work).

## 3. A support assistant that actually resolves things — **P2**

An in-app assistant that answers billing/roaming/outage questions instantly, walks a customer
through eSIM or roaming setup, and escalates to a human only for what genuinely needs one.

**Why:** this is where SIMBA's cost structure and its customers' expectations meet. Routine
questions — why did my bill change, how do I turn on roaming, is there an outage — are the
highest-volume, lowest-value support tickets, and every one handled by a live agent is a
challenger telco spending scale-player money it doesn't have. Done well, it also reinforces trust
(fast, clear answers) instead of the hold-the-line frustration that erodes it.

**Priority:** sequenced last, deliberately. It only works well once usage/billing data (#1) is
clean and self-serve actions (#2) exist for it to actually *do*, not just explain. Building it
first would ship an assistant that can describe a problem but can't fix it.

## How this was prioritized

Three questions, in this order: does it protect the trust the brand is already promising to
customers; does it reduce cost-to-serve or friction in the one channel a no-storefront challenger
has; and what depends on what. Feature 3 is arguably the most strategically interesting of the
three — but sequencing it last, behind the data and actions it depends on, is itself the
prioritization call this exercise is testing.
