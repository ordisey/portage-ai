# Part 2 — Singapore Telco Mobile App (SIMBA)

Top 3 features, with my rationale and how I'd prioritize them. Keeping this short on purpose — the
brief itself asks for a quick read on product thinking, not a second POC.

## Context

SIMBA's proposed acquisition of M1 didn't go through — IMDA suspended its review in May 2026 over a
spectrum-licensing investigation, and the deal was terminated. That leaves SIMBA where it's actually
been most effective: a lean, digital-first challenger competing on price, transparency, and
simplicity rather than network scale. That's the lens I used below — not a merged-entity roadmap,
just what a standalone challenger's app needs to get right.

## 1. Usage & billing you can trust at a glance — P0

Real-time data/talktime/rollover balance, proactive low-balance and bill-change alerts, and a
plain-language breakdown of every charge.

**Why:** SIMBA's entire pitch — no contract, no hidden fees, free 6-cycle rollover — is a trust
claim. If the app doesn't make that promise instantly legible, and current My SIMBA reviews mention
it can be slow to load and has crashed, the brand's whole differentiator disappears at the exact
moment it matters: the moment someone checks whether they're about to pay more. This is the
cheapest, most foundational move available, since it's data SIMBA already has — it just isn't
legible yet.

**Priority:** first. It protects existing revenue through retention, and it needs no network or
retail investment to ship.

## 2. Instant self-serve for everything — P0/P1

One-tap eSIM activation and porting, in-app plan switching, and simple multi-line/family
management — no store visit, no call.

**Why:** SIMBA doesn't have a retail footprint like Singtel or StarHub — the app is the storefront.
Every point of friction in it, a plan change that needs a call, an eSIM activation that isn't
instant, either sends the customer to a competitor with a shop, or into a support queue that costs
more per interaction than a lean challenger can afford. It's also the biggest growth lever they
have: "switch to SIMBA, it took three minutes" is the whole acquisition engine for a brand with no
storefront.

**Priority:** build alongside #1. I'd rank it second only because the scope — eSIM and porting
integration — is bigger.

## 3. A support assistant that actually resolves things — P2

An in-app assistant that answers billing, roaming, and outage questions instantly, walks a customer
through eSIM or roaming setup, and escalates to a human only when it genuinely needs one.

**Why:** this is where SIMBA's cost structure and its customers' expectations meet. Routine
questions — why did my bill change, how do I turn on roaming, is there an outage — are the
highest-volume, lowest-value tickets a support team gets, and every one a live agent handles is
scale-player money a challenger doesn't have. Done well, it also builds trust instead of eroding
it — fast, clear answers instead of hold-the-line frustration.

**Priority:** last, on purpose. It only works once the usage/billing data from #1 is clean and the
self-serve actions from #2 actually exist for it to use. Build it first and you'd ship an assistant
that can explain a problem but can't fix it.

## How I prioritized

Three questions, in this order: does it protect the trust the brand is already promising, does it
cut cost-to-serve or friction in the one channel a no-storefront challenger actually has, and what
depends on what. Feature 3 is arguably the most interesting of the three strategically — but putting
it last, behind the data and the actions it needs to be useful, is itself the call this exercise is
testing.
