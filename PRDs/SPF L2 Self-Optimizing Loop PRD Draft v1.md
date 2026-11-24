---
tags: [prd, spf, image-validation, ai]
source: "SPF L2 Phase 1; Mohini Agent Optimizer PRD Draft v2; Custom Metrics research"
author: "Ayushmaan Pattnayak"
date_ingested: 2026-11-19
status: seed
confidence: medium
related:
  - "[[SPF L2 Phase 1]]"
  - "[[Mohini - Agent Optimizer PRD Draft v2]]"
  - "[[Custom Metrics — LLM Judge Accuracy & Calibration Research]]"
---

# SPF L2 Self-Optimizing Loop — PRD (Draft v1)

Nov 19, 2026 · Owner: Ayushmaan Pattnayak

**Status.** Draft. Builds on [[SPF L2 Phase 1]] (the image-validation system this loop improves). Loop structure is adapted from [[Mohini - Agent Optimizer PRD Draft v2]] (one cluster, one fix; simulate before showing; owner approves; nothing auto-applies). Numbers not found in the inputs are in Open Items, not invented.

## Open Items for This Draft

| # | Item | Why it matters |
|---|---|---|
| O1 | Is the TnS final decision logged **per comparison** (SPF vs Catalog, OBD vs Catalog, flyer, damage), or only per claim? | Without per-comparison labels there is no ground truth for the loop (US1). |
| O2 | Cost of a false Approve (payout leak) vs a false Reject (seller harm, escalation), in ₹. | Needed to set the precision floor and weight errors (US8). |
| O3 | Auto-decision rate and accuracy targets for go-live. Phase 1 PRD §7 leaves them undefined. | Defines when shadow mode ends (US10). |
| O4 | SLA: Phase 1 says 3 hrs per 6k batch; 18 Nov MoM notes a 3 → 6 hrs change after recent SPF L2 changes. Which holds? | Sets the compute and retry budget (US11). |
| O5 | Who is the Owner who approves changes? Assumed TnS lead. | US7. |
| O6 | Who sets "Actual Reason for Claim", and when? | Usable as model context only if sellers set it upfront. |
| O7 | Do sellers or agents author the SPF images' remarks? Is Seller OBD remarks reliably populated? | Affects what context the model can use. |
| O8 | Gold-set size per workflow and per category needed for stable accuracy estimates. | Data science to size (US2). |
| O9 | Handling of RTOCV-missing cases and RVP WPR CX MM flow (both open in 18 Nov MoM). | Out of Phase 1 scope; confirm they stay out of the loop. |
| O10 | Retry cap per cluster (3 in the Mohini optimizer, DS to set ₹ cap). | US4. |
| O11 | Retention and audit requirements for images, model outputs and labels. | Phase 1 PRD lists audit/logging as undefined. |
| O12 | Mapping of each FMLM Task form question to a pipeline comparison (§10.2 is my reading, unconfirmed). | Wrong mapping = wrong labels. |
| O13 | Once the form is retired after go-live, where do labels come from? Proposed: permanent sampled TnS audit (§10.5). | Label source would otherwise dry up. |
| O14 | `spf_flyer_status`: Phase 1 says one flyer present = `skipped`, AIS says it is marked a mismatch. Which is logged in Traces? | Decides how the loop scores flyer-path cases. |
| O15 | Does MART (L1) output reach the pipeline as a usable signal for Non-GC SPF vs Catalog (Phase 1 decision 6)? | Only source of a real verdict for that node. |

## 1. Context

SPF L2 receives about 10,000 claims a day (6k morning, 4k noon). Mix: RTO 30%, WPR 55%, Damage 15%. Each claim carries up to ~8 images (catalog and SPF 4–6, Seller OBD up to 2). Phase 1 automates image comparison to hit 3 hrs for a 6k batch (~33 claims/min) with SPF vs Catalog at >90% accuracy / >85% recall and Seller OBD vs Catalog at >85% / >80%.

Human reviewers err 2–3%. At 10k claims a day that is roughly 200–300 wrong decisions a day in the baseline (derived, not measured).

## 2. Problem Statement

Phase 1 ships a fixed set of prompts, thresholds and models. Shadow mode will show where the AI and TnS disagree, but nothing turns those disagreements into improvement. Today, if the AI is wrong on a pattern, someone has to notice it on a dashboard, work out why and hand-tune it, and that does not scale across three workflows, many categories and a catalog that changes daily. Four compounding problems:

1. **No trustworthy ground truth.** Humans err 2–3%, so "AI ≠ human" is not the same as "AI wrong". Evidence: 41 of 2,326 cases (1.8%) took the wrong path (RVP vs Catalog despite a flyer present) and were approved when the flyer IDs matched. Optimising against raw human decisions would learn the human mistakes.
2. **Failures are patterned, not random.** Category (Lifestyle first), image quality, angle, brand, combo or quality-disguised-as-damage fail in different ways; a global tweak fixes one and breaks another.
3. **Errors are asymmetric.** A false Approve leaks payout; a false Reject hurts a seller. Plain accuracy hides this.
4. **Manual-review queue is the only release valve.** Inconclusive cases go to TnS, and nothing makes that queue shrink over time.

## 3. Summary

**A closed loop that turns every AI-vs-human disagreement in SPF L2 into a tested, owner-approved improvement.** Shadow-mode decisions are compared to TnS decisions per comparison; disagreements are adjudicated into a gold set, clustered, and each cluster gets one proposed change (rubric, exemplars, preprocessing, model choice or confidence threshold). The change is replayed on the gold set, then run as challenger against the current champion on the same live claims in shadow mode, and the Owner accepts or rejects it. Example: a cluster "Lifestyle, Seller OBD vs Catalog, false Match on same-brand different-colour" gets a rubric change that adds a colour/variant check; it passes the gold-set replay and beats the champion on 2 weeks of shadow claims, and the Owner accepts.

## 4. Right to Win

Quantified where inputs allow:

- Baseline human error of 2–3% on ~10k claims/day (≈200–300 decisions/day).
- Known systematic human miss: 41 of 2,326 (1.8%) flyer-ID cases.
- MART non-GC mismatch handling: 4/650 (reported as 0.24%, arithmetically 0.62%; see [[SPF L2 Phase 1]] open items).
- Manual turnaround today is 4–5 hrs for the morning batch against a 3 hr target.

**Not yet sized:** ₹ payout leaked per false Approve, ₹ cost of TnS time per manual review, and the value of each extra point of auto-decision rate (O2). Sizing these is the first task before leadership review.

## 5. Goals

**Phase 1 (shadow mode)**
1. Produce a per-comparison, adjudicated gold set from shadow-mode disagreements.
2. Show the Owner every failure cluster with evidence and an accuracy / recall / confusion matrix per comparison and category.
3. Propose one tested change per cluster; never apply a change without Owner approval.
4. Guarantee no change reaches a live decision: all changes run in shadow only.

**Phase 2 (after go-live decision)**
5. Raise the auto-decision rate within a precision floor by moving the Inconclusive threshold per category.
6. Detect drift and roll back automatically.
7. Consider auto-apply for low-risk changes, earned from the Owner's accept history (as in Mohini v0: nothing auto-applies in v0).

## 6. Who Uses It

**TnS lead (Owner).** Knows the SOP, the claim categories and what a good decision looks like; is not an ML engineer and cannot read a prompt diff. Must be able to judge a proposal from evidence (failing cases, before/after confusion matrix, cost of the change) without depending on engineering for each decision. Secondary users: Data Science (sets gold-set sizes, retry cost cap), AIS engineering (owns the pipeline and Traces).

## 7. Success Metrics

| Type | Metric | Why it belongs here |
|---|---|---|
| Input | Share of shadow-mode comparisons with a logged TnS decision | No label, no loop (O1). |
| Input | Gold-set size per workflow × category | Accuracy estimates are meaningless on small samples (O8). |
| Output | Clusters detected / proposals made / accepted / rejected | Shows the loop is producing usable changes; accept rate is the signal for future auto-apply. |
| Output | Proposals that pass gold-set replay but lose in shadow ("Sims miss") | Measures how well offline replay predicts live behaviour. |
| Outcome | Accuracy and recall per comparison vs adjudicated gold (not raw human) | The real quality measure; targets from Phase 1 §8. |
| Outcome | Auto-decision rate at the precision floor | Whether manual queue shrinks. |
| Outcome | False Approve rate and false Reject rate, separately | Asymmetric cost (O2). |
| Outcome | Batch completion time vs SLA | The loop must not break speed (O4). |
| Guardrail | Cost per claim (model + retries + replay) | Optimiser cost counts against savings. |

## 8. Definitions

| Term | Meaning |
|---|---|
| Comparison | One image check: SPF vs Catalog, Seller OBD vs Catalog, Flyer text match, or Damage yes/no. The unit of evaluation. |
| Verdict | Match, Mismatch or Inconclusive output of a comparison. |
| Champion | The configuration currently producing shadow decisions. |
| Challenger | A proposed change run beside the champion on the same claims. |
| Disagreement | A comparison where the AI verdict differs from the TnS form answer. |
| Adjudicated label | The label after a disagreement is resolved by a second look (stronger model plus TnS spot check). |
| Gold set | Frozen adjudicated comparisons, never shown to the optimizer except through scoring. |
| Held-out set | Part of the gold set the optimizer never sees, used for the final gate. |
| Cluster | A group of failures with a shared cause (workflow, category, image condition). |
| Change | One edit to rubric, exemplars, preprocessing, model or threshold, targeting one cluster. |
| Sims miss | A change that passed replay but lost in shadow. |
| Auto-decision rate | Share of claims decided without manual review. |
| Owner | TnS lead who accepts or rejects changes. |

## 9. User Stories & Acceptance Criteria

### US1: Capture per-comparison shadow decisions and labels

As the Owner, I want every comparison to store the AI verdict and the TnS answer, so that disagreements can be measured.

| AC | Description |
|---|---|
| 1 | For each claim, the system stores per comparison: claim ID, workflow (RTO / RVP-WPR / Damage), return type, category, image references, AI verdict, confidence, Traces entry and the TnS answer from the agent form. |
| 2 | Where the TnS answer exists only at claim level, the story is blocked and flagged against O1; the system does not infer per-comparison labels from the final Approve/Reject. |
| 3 | A comparison skipped by the model (for example flyer status `skipped`, or a broken OBD link that fell back to the flyer API) is stored with that reason, so it is not counted as a miss. |
| 4 | Sensitive fields (seller, customer identifiers) are masked in the loop's working store; only claim ID is retained. Open: retention period (O11). |

### US2: Adjudicate disagreements into a gold set

As the Owner, I want disagreements resolved before they are used, so that the loop does not learn human mistakes.

| AC | Description |
|---|---|
| 1 | Every disagreement goes to a second look by a stronger model with a different family from the production model, to avoid same-family bias (per the Custom Metrics research). |
| 2 | Where the second look disagrees with TnS, a TnS reviewer who did not make the original decision makes the final call; that call is the adjudicated label. |
| 3 | A random sample of agreements is also adjudicated each week so the gold set is not only hard cases and accuracy is not biased. Sample size is set by Data Science (O8). |
| 4 | Gold-set entries are immutable and versioned; a label change creates a new version, and every past score stays attributable to the version used. |
| 5 | The gold set is split into a development part the optimizer can learn from (exemplars) and a held-out part it never sees. |

### US3: Cluster failures

As the Owner, I want failures grouped by cause, so that one fix addresses one pattern.

| AC | Description |
|---|---|
| 1 | Clusters are shown per comparison with workflow, category, failure type (false Match, false Mismatch, wrong Inconclusive), example cases and the confusion matrix. |
| 2 | A cluster becomes eligible for a proposal only after it passes a minimum sample and error-rate bar, set by Data Science (O8), so noise is not chased. |
| 3 | A cluster whose cause is not an image-comparison fault (bad catalog image, missing metadata, upstream data) is labelled as such and handed back with evidence; no proposal is generated. |
| 4 | Each cluster shows false Approve and false Reject separately, because their costs differ (O2). |

### US4: Propose one change per cluster

As the Owner, I want one targeted change per cluster, so that its effect can be measured on its own.

| AC | Description |
|---|---|
| 1 | A change touches exactly one lever for one cluster: rubric/prompt text, retrieved exemplars, image preprocessing, model choice for the category, or the Inconclusive threshold. |
| 2 | Each attempt receives the previous attempt's replay failures as input. After the cap (3, with a ₹ cap per cluster; O10), the cluster is handed back with everything tried. |
| 3 | A change that was rejected by the Owner is not re-proposed for the same cluster; the rejection reason is recorded and used in the next proposal. |
| 4 | Prompt/rubric changes carry a section ID, so two changes to the same section are visible together, with the cluster each one targeted. |
| 5 | A change that removes or loosens a check is allowed only after it passes replay without any increase in false Approve. |

### US5: Offline verification gate

As the Owner, I want proposals tested before I see them, so that I only review changes that are not obviously harmful.

| AC | Description |
|---|---|
| 1 | Every change is replayed on the cluster's failing cases, the full regression suite and the held-out set, for the target comparison and for any comparison sharing the same prompt section. |
| 2 | A proposal reaches the Owner only if the failing cases improve and no category's accuracy on the regression set drops beyond a tolerance set by Data Science (O8). |
| 3 | The proposal card shows evidence counts (failing cases fixed, regression passes, held-out result), not a single confidence percentage. Replay proves "does not break", not uplift. |
| 4 | Latency and cost of the changed configuration are measured in replay; a change that would breach the throughput budget (O4) is rejected before the Owner sees it. |

### US6: Champion vs challenger in shadow

As the Owner, I want to see a change beat the current system on real claims with no live risk, so that I can accept it with evidence.

| AC | Description |
|---|---|
| 1 | The challenger runs on the same live claims as the champion; no traffic split is needed because both run in shadow. |
| 2 | Results compare both against adjudicated labels once TnS decisions arrive, and show the difference in accuracy, recall, false Approve rate, false Reject rate and Inconclusive rate per cluster. |
| 3 | The run lasts until a minimum number of labelled comparisons is reached for the cluster, set by Data Science (O8); it does not stop on a lucky early lead. |
| 4 | If the challenger is worse on any guardrail (false Approve rate, throughput), the run stops automatically and the change is recorded as a Sims miss. |

### US7: Owner accepts, edits or rejects

As the Owner, I want explicit control, so that nothing changes without my action.

| AC | Description |
|---|---|
| 1 | The proposal card shows what it fixes, what changes, why, replay results, shadow results, cost change and anything that got worse. |
| 2 | Accept promotes the challenger to champion for shadow decisions; Reject requires a reason; Edit sends the change back through replay and shadow, since an edited change is untested. |
| 3 | Every promotion is versioned and reversible in one action; the previous champion is kept. |
| 4 | In Phase 1 no accept reaches a live decision. Live use depends on the go-live decision (US10). |

### US8: Confidence threshold and auto-decision coverage

As the Owner, I want the loop to widen automation safely, so that the manual queue shrinks without raising false Approves.

| AC | Description |
|---|---|
| 1 | For each comparison and category, the system reports the trade-off between auto-decision rate and false Approve / false Reject rates at each threshold. |
| 2 | The Owner sets a precision floor per workflow; the loop only proposes thresholds that keep the floor on the held-out set. |
| 3 | Combo cases and quality-disguised-as-damage always route to manual review, as in Phase 1 §6.3; the loop cannot change this. |
| 4 | RVP vs Catalog and RTOCV image comparisons remain outside the loop until they are in Phase 1 scope (O9). |

### US9: Manual-review outcomes feed back

As the Owner, I want TnS decisions on Inconclusive cases to become labels, so that the loop learns from the hardest cases for free.

| AC | Description |
|---|---|
| 1 | Every Inconclusive case routed to TnS stores the AI evidence and the TnS decision as a labelled comparison. |
| 2 | A cluster of Inconclusive cases that TnS resolves the same way is surfaced as an automation candidate, with the proposed threshold or rubric change. |
| 3 | Labels from manual review go to the gold set only after the same adjudication rules as US2, since TnS also errs 2–3%. |

### US10: Drift monitoring and go-live decision

As the Owner, I want to know when quality moves and when the AI is good enough to take over decisions, so that I can decide from evidence.

| AC | Description |
|---|---|
| 1 | A weekly view shows accuracy, recall and the confusion matrix per comparison against adjudicated gold, with the human-vs-gold baseline alongside. The AI is judged against that baseline, not against 100%. |
| 2 | An alert fires when a category's accuracy drops beyond a tolerance set by Data Science (catalog, seasonal or new-seller drift). |
| 3 | The go-live decision (Phase 2) needs accuracy at or above the Phase 1 §8 targets and at or above the human baseline, and the auto-decision target from O3. |
| 4 | After go-live, any promoted change takes effect first on a small share of live claims with automatic rollback. The share is open for the Owner and TnS to set. |

### US11: Cost and throughput guard

As the AIS engineer, I want the loop to stay inside the processing budget, so that SLA is not harmed.

| AC | Description |
|---|---|
| 1 | Replay, adjudication and shadow challengers run off the critical path; they must not delay the morning batch (6k in the SLA window; ~33 claims/min at the 3 hr target, subject to O4). |
| 2 | Cost per claim, per cluster and per accepted change is shown, so the Owner sees what each change earned against what it cost. |
| 3 | A weekly budget cap on adjudication and retries is configurable; reaching it pauses proposals, not decisions. |

### US12: History and audit

As the Owner, I want to see what changed, why and what happened, so that decisions are defensible.

| AC | Description |
|---|---|
| 1 | Each comparison's history lists every change to its prompt section, the cluster each one fixed, the date, the approver and the result. |
| 2 | Every shadow decision stores the champion version used, so any past decision can be reproduced. |
| 3 | If clusters keep reopening each other, or a section changes more than N times in M days, the section is frozen for proposals and the Owner is asked to decide, with the choice written as an explicit rule. N and M are open. |

## 10. Integration with Current SPF L2 Workflows

The loop sits beside the Phase 1 pipeline (routing → comparisons → SOP decision → Traces). It reads Traces and TnS form answers and changes only comparison nodes. It never edits routing or SOP decision rules (Approve/Reject mapping), which stay owned by TnS.

### 10.1 What the loop can tune, by workflow

| Workflow | Node | Tunable by the loop? | Lever | Notes |
|---|---|---|---|---|
| RVP-WPR GC | SPF vs Catalog | Yes | Rubric, exemplars, preprocessing, model, threshold | Phase 1 decision 2 adds this node to GC; it is the first node to baseline. Match → Reject; Mismatch → continue to OBD. |
| RVP-WPR Non-GC | SPF vs Catalog | No | n/a | Marked Mismatch for all, with no image comparison (agreed by Engineering, Product, TnS). Excluded from scoring. MART's L1 output is the only real signal (O15). |
| RVP-WPR (GC and Non-GC) | Seller / Forced / On-demand OBD vs Catalog | Yes | As above | Target >85% accuracy, >80% recall. Match → Reject; Mismatch → Approve. |
| RVP-WPR | FM vs LM flyer text | Partly | OCR/extraction quality and text normalisation only | The match itself is a string comparison. Scored on extraction accuracy. Status per `spf_flyer_status` (O14). |
| RTO | Flyer ID / serialisation | Partly | Same as above | One flyer present counts as mismatch. |
| RTO | Seller OBD vs Catalog | Yes | As above | Three outputs: no issues → Reject, has issues → Approve, Inconclusive → manual. |
| Damage | Damage in product or packaging, with severity | Yes | Rubric (especially primary packaging vs Flipkart packaging), exemplars, threshold | Missing SPF images → Reject by rule; combo and quality-disguised-as-damage → manual always (loop cannot change). |
| All | Routing (return type, claim code, reason) | No | n/a | Deterministic. Loop only flags misroutes. |
| Out of scope | RVP vs Catalog, RTOCV, RVP Missing/Misshipment, video | No | n/a | Not in Phase 1 (O9). |

### 10.2 Label source: FMLM Task form to comparison

Agents answer 15–17 questions per claim, appended to the bulk CSV. The in-scope questions give per-comparison labels. My mapping, unconfirmed (O12):

| Form question (in scope) | Candidate comparison |
|---|---|
| Seller image = Catalog image | SPF vs Catalog |
| Seller image = Seller OBD image | Seller OBD vs SPF image (confirm if it feeds the OBD vs Catalog check) |
| CX OBD / CX image = Catalog; = RVP; RVP = Catalog; RVP = Seller image | Not in Phase 1 automation or partly; confirm |
| Brand/design on image is part of seller's listing | Brand check; out of Phase 1 |
| Damage questions | Damage (not in the list above, confirm form fields) |

Label join key: incident ID, matching the Traces column in the output sheet.

### 10.3 Where the loop runs in the daily cycle

| Time | Phase 1 event | Loop action |
|---|---|---|
| 8:00 AM | Morning batch (6k) assigned | Pipeline runs with the champion version pinned for the whole batch; challengers run in shadow on the same claims, off the critical path. |
| By ~11 AM | AI results due (3 hr target; see O4) | Inconclusive cases reach TnS inside the window. |
| 2:00 PM | Noon batch (4k) | Same version pinned; same shadow challengers. |
| Evening | TnS manual forms and CSV complete | Labels joined to Traces; disagreements queued for adjudication. |
| Weekly | n/a | Clustering, proposals, owner review. Promotion takes effect between batches, never mid-batch. |

### 10.4 Selection effect

SPF L1 rejections never reach TnS (Phase 1 decision 1). The gold set therefore describes the L2 population only: L1-approved plus Detailed Investigation cases. Accuracy and thresholds from the loop apply to L2 and must not be quoted for all SPF claims. If L1 behaviour changes (for example Damage is withdrawn from MART), the mix shifts and the loop must treat that week as drift (US10).

### 10.5 After go-live, labels dry up

Today the form creates labels as a by-product of manual work. When the AI takes decisions, the form stops, and with it the label stream. So the loop needs, permanently:

- A sampled TnS audit of auto-decided claims at a rate that keeps each category's gold-set growth above the minimum (O8).
- Full TnS labels on every Inconclusive case (US9).
- A higher audit rate on a category for two weeks after any promoted change.

### 10.6 Human path deviations as a signal

The 41 flyer cases show agents can take a different path from the SOP. The loop should log claims where the agent's path differs from the pipeline's route (for example flyer present but the agent used RVP vs Catalog) as a separate cluster type, so that SOP drift and training gaps are visible and are not mistaken for model errors.

### 10.7 Added acceptance criteria

| AC | Description |
|---|---|
| US1-6 | Each stored comparison records the workflow node from §10.1; Non-GC SPF vs Catalog is stored as "not evaluated" and never scored. |
| US1-7 | The loop joins the form answers and Traces on incident ID; a claim with a form answer but no Traces entry (or the reverse) is counted and shown, not dropped. |
| US7-5 | A promotion is pinned to batch boundaries; every claim in a batch uses one champion version. |
| US10-5 | Reports state that figures apply to the L2 population (§10.4). |
| US9-4 | After go-live, the audit sampling in §10.5 is on by default and cannot be set to zero. |

## 11. Dependencies and Risks

| Item | Note |
|---|---|
| Per-comparison TnS labels (O1) | Hard dependency. If missing, add a short form capture or sampled re-labelling before building the loop. |
| Judge reliability | Second-look model and TnS reviewers are themselves imperfect. Accuracy of the second look must be measured against TnS on a labelled sample (see Custom Metrics research; Annotation Reliability Gradient: concrete checks like "same brand" are more reliable than subjective ones like "quality disguised as damage"). |
| Criteria drift | TnS may redefine what counts as a match once they see model outputs. Labelling and rubric edits must interleave, and past labels may change (US2 AC4). |
| Overfitting to the gold set | Held-out set and fresh weekly samples guard against it. |
| Optimiser cost | Retries and replay cost money; US11 caps them. |
| Upstream data issues | Bad catalog images and broken OBD links are not model faults (US3 AC3). |

## 12. Out of Scope

- Video-based adjudication (future scope in Phase 1).
- RVP vs Catalog, RTOCV comparison, RVP Missing / Misshipment flows.
- Merging L1 and L2.
- Auto-applying changes (Phase 2, earned from accept history).

<!-- auto-related:start (generated by _agent/scripts/link_notes.py — edits inside are overwritten) -->
## Related
- [[PRDs/SPF L2 Phase 1|SPF L2 Phase 1]]
- [[Self-optimizing Loop/Outputs/Auto Evals Phase 2 - PRD Draft v1|Auto Evals Phase 2 - PRD Draft v1]]
- [[Self-optimizing Loop/Outputs/Auto Evals Phase 2 - PRD Approved Buildout|Auto Evals Phase 2 - PRD Approved Buildout]]
- [[Self-optimizing Loop/Outputs/Self-Optimizing Loop — Leadership POV|Self-Optimizing Loop — Leadership POV]]
- [[Self-optimizing Loop/Outputs/Self-Optimizing Loop — Product Strategy Narrative|Self-Optimizing Loop — Product Strategy Narrative]]
<!-- auto-related:end -->
