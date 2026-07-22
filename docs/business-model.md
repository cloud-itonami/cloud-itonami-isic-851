# Business Model: Pre-primary and Primary Education (Administrative Coordination)

## Classification
- Repository: `cloud-itonami-isic-851`
- ISIC Rev.4: `851` — pre-primary and primary education, narrowed to back-office
  administrative COORDINATION (never grading, curriculum, discipline, or
  custody)
- Social impact: student-safety administrative-coordination family-engagement
  equitable-access

## Customer

- primary/pre-primary school office administrators and coordinators (public
  or private) looking to replace paper/spreadsheet/group-chat coordination
  with a governed, auditable workflow
- small-school operators (the school itself, or a school-management IT/
  volunteer lead) who cannot justify a full district-scale SIS/communication
  suite

## Problem

Back-office school coordination — attendance/pickup logistics, parent/
guardian meeting scheduling, classroom supply requests, staff shift
proposals, and wellbeing/safety-concern flagging — is usually run on paper,
spreadsheets or ad hoc group chat, with no immutable record of who decided
what and no structural guarantee that a safety concern (suspected abuse/
neglect, bullying) is escalated to a human rather than silently logged.
Commercial alternatives (ParentSquare, Brightwheel, Procare) bundle this
into a much larger, sales-gated communication/SIS/payments suite with no
public pricing at all (see Revenue below) and no governor-style hard block
on ever auto-committing a safety flag.

## Offer

- `log-attendance-note` — attendance/pickup logistics logging
- `schedule-parent-guardian-meeting` — meeting scheduling coordination
- `coordinate-supply-request` — non-instructional consumable supply requests
- `schedule-staff-shift-proposal` — administrative shift proposals (never a
  final binding assignment)
- `flag-safety-concern` — wellbeing/safeguarding concerns; **always**
  escalates to a human, at every rollout phase, no exceptions
- an independent Governor with three permanent HARD checks (student must be
  registered/verified; every proposal effect must be `:propose`, never a
  direct actuation; grading/curriculum/discipline/custody/safety-authority
  content is permanently blocked) — see ADR-2607152900
- an append-only audit ledger: every decision (commit/hold/escalate, by
  whom, on what basis) is a query over an immutable log

## Funnel (demo → fork → certified operator)

1. **Demo** — the nightly build-time-regenerated operator console
   (`docs/samples/operator-console.html`, `.github/workflows/regenerate.yml`)
   shows the governed proposal flow through the real actor code, not a mockup.
2. **Fork / self-host** — AGPL-3.0-or-later; run the actor for one school.
3. **itonami.cloud certification** (optional) — same trust ladder as every
   cloud-itonami venture; see Operator Trust Levels below.

## Revenue

Operators can sell:

- self-host setup: one-time implementation fee (student directory import,
  rollout-phase sign-off, staff onboarding)
- managed hosting: flat monthly platform fee per school
- compliance package: audit-ledger export for a district/board review

| Package | Customer | Price shape (example) |
|---|---|---|
| Self-host starter | school IT/office lead | setup ¥100k–250k + optional support |
| Managed School Ops (Starter) | one school, unlimited staff seats | ¥25,000/月 flat |

**Market-anchored (2026-07-22)**: benchmarked against 5 real competitor
products (`90-docs/pricing-intelligence/pricing-intelligence-ledger.edn`,
`run-id "pricing-intel-20260722-01"`). Only 2 of the 5 publish real numbers —
**Bloomz** ($3/student/year schoolwide, published on its own site) and
**SchoolCues** ($75/mo base + up to $1/mo/student for additional modules,
per its own blog). The other 3 — **ParentSquare**, **Brightwheel**,
**Procare** — disclose nothing publicly; every one of them routes to a
sales-quote form, a starker opacity pattern than the sibling flagships found
in HR/recruiting/CRM SaaS. Converting the 2 published anchors at ~¥150/$ for
an illustrative 300-student primary school: Bloomz lands at ~¥11,000/月;
SchoolCues at full modular buildout lands at ~¥56,000/月. **¥25,000/月 sits
inside that real range**, biased toward the low end because this actor is
coordination-only (no parent broadcast messaging, no tuition/payment
processing, no gradebook/SIS) — but it also carries something neither
comparator does structurally: every proposal passes an independent Governor
that permanently blocks non-`:propose` effects and out-of-scope content, and
a safety-concern flag can never auto-commit at any rollout phase.

This is **deliberately not** the ¥50,000–150,000/月 portfolio-uniform range
used by `cloud-itonami-isic-6399`/`6310`/`7810`/`5820` — that range was
anchored on HR/recruiting/CRM per-seat SaaS competitors, which have no
evidenced relationship to K-12 school administrative-coordination pricing.
Carrying that number over here without market evidence would be exactly the
kind of unfounded figure this ledger exists to prevent.

**Subscribe (2026-07-22)**: a live Stripe Payment Link for the Managed
School Ops (Starter) tier (¥25,000/月 flat) is available now —
[**subscribe to Managed School Ops — Starter**](https://buy.stripe.com/bJe3cu74ncW46qs5c3bMQ0h).
This is a no-code Stripe-hosted checkout; nothing in this repo's actor code
changed. After subscribing, contact gftdcojp via an [operator-interest
issue](https://github.com/cloud-itonami/cloud-itonami-isic-851/issues/new?template=operator-interest.yml)
to arrange managed-tenant setup (manual fulfillment today, no automated
onboarding yet). **No school has claimed or subscribed to this tier yet —
this is a live, working checkout with zero paid tenants, not a claim of
existing revenue.**

## Unit Economics (worked example, illustrative)

One managed school (≤500 students):

- infrastructure: actor runtime + audit ledger ≈ ¥3k–8k/月 (no per-search
  compute; the actor runs at proposal time, not continuously)
- LLM cost: proposals only, at coordination-event time — bounded because
  nothing here is a continuous background process
- human approval labor: the real cost driver until phase 3 auto-commit is
  earned — a school office coordinator reviewing proposals, budget ~2–4
  h/月 once the rollout phase stabilizes
- support + incident: budget 2–3 h/月

At ¥25,000/月, gross margin is thin for a single school in isolation — the
business scales with number of schools onboarded per operator (a district,
diocese, or franchise group), not with proposal volume, which is why the
self-host + certified-operator ladder (rather than gftdcojp running every
school directly) is the intended growth path.

## Open Participation

Anyone may fork, run the demo, self-host, submit patches, and build a local
operator business. itonami.cloud certification is required before an
operator is listed, receives leads, or runs managed tenants under the
platform brand.

## Operator Trust Levels

| Level | Capability |
|---|---|
| Contributor | patches, docs, examples |
| Self-host operator | runs their own school's instance, no platform endorsement |
| Certified operator | listed on itonami.cloud after review |
| Managed operator | may receive leads and operate customer (school) tenants |
| Core maintainer | can approve changes to governor, security and governance |

## Trust Controls

- a proposal for an unregistered/unverified student is never committed or
  even escalated
- any proposal whose effect is not `:propose`, or whose content touches
  grading/curriculum/discipline/custody/safety-authority territory, is
  permanently blocked — not merely low-confidence
- a safety-concern flag always reaches a human; it can never auto-commit,
  at any rollout phase
- every commit, hold, escalation and approval path is auditable
- sensitive student/family data stays outside Git

## Non-Negotiables

- Do not commit real student/family PII to this repository.
- Do not bypass the Governor for production commits.
- Do not expand the proposal-op allowlist to grading, academic assessment,
  curriculum, discipline, special-education/IEP, or custody decisions
  without a new ADR.
- Do not market an uncertified deployment as an itonami.cloud certified
  operator.
