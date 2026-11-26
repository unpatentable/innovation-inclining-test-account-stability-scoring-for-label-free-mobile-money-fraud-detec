# Inclining-Test Account Stability Scoring for Label-Free Mobile Money Fraud Detection

> **Note:** This README is an ultra-condensed summary of the research reports published by Unpatentable.org. To view the full innovation documentation and explore the complete research, use the links at the bottom of this document.

**Domain:** [Digital & Information Technologies](https://unpatentable.org/innovation/?domain=digital-information-technologies)
**Subdomain:** [Unsupervised Learning](https://unpatentable.org/innovation/?domain=digital-information-technologies&subdomain=unsupervised-learning)

---

### Why We Invented This Technology
Mobile money users in Africa, hundreds of millions of accounts held mostly by low-income people, lose savings to SIM-swap, social-engineering and agent fraud; second-hand reports of INTERPOL's 2026 assessment put continental losses above $4 billion a year, and a single loss can wipe out a household's working capital. Label-free behavioural anomaly detection is the natural defence because labelled fraud data barely exists there, but the practical methods (per-account behavioural profiling, peer-group outlier scoring, graph-based collusion detection) are widely believed to be covered by dense patent portfolios held by large card networks and fraud-analytics vendors. My one search did not confirm the specific patents or their coverage in African jurisdictions, so the enclosure claim is a hypothesis to verify, and vendor pricing and data scarcity are probably co-causes alongside patents. If nothing changes, small operators, SACCOs and microfinance institutions keep running rule-based checks while losses grow with transaction volume.

### The Innovation
## Abstract
This method treats each mobile money account as a floating vessel and detects fraud by comparing two independent estimates of its metacentric height. The first is calculated passively from transaction logs: habitual, long-dwelling value acts as ballast, fresh pass-through value as top-weight, and the peer stratum as the displaced fluid. The second is measured actively, as naval architects do in an inclining experiment. A small, randomised, known friction is applied to an exit transaction and the behavioural "heel" is recorded on the server side of an ordinary USSD session. The log-ratio of observed to predicted heel is the anomaly statistic. No labels, client software or licensed platform are needed, and reactions slow accounts but never block them automatically.

## Background & Prior Work
Label-free fraud detection is dominated by passive comparison: peer group analysis (Bolton & Hand), Local Outlier Factor, Isolation Forest, and commercial behavioural-profiling platforms. These observe but never test, so a taken-over or mule account whose marginal statistics resemble its peers stays invisible.

Active methods exist only at the user-interface layer. BioCatch's "invisible challenge" patents inject cursor or drag-and-drop interference and compare the corrective reaction with a user-specific model ([US10474815B2](https://patents.google.com/patent/US10474815B2/en), [US10776476B2](https://patents.google.com/patent/US10776476B2/en)). They presuppose client-side instrumentation and earlier reactions from the same user, neither of which exists on feature-phone USSD.

The present method differs in three ways. It perturbs at the transaction layer, observes entirely server-side, and predicts each account's response from passively computed hydrostatics. An account can therefore be scored on its first probe.

## Competitive Landscape & Strategic Edge
From recall, FICO Falcon, Featurespace ARIC, NICE Actimize, SAS and BioCatch serve banks with card and app channels at enterprise prices. USSD-specific vendor material describes coarse signals such as session frequency, repeated authentication attempts and PIN changes ([Neuralt](https://www.neuralt.com/news-insights/ussd-fraud-detection-prevention-mobile-money)).

There are three blind spots:
- **Friction as instrument:** friction is treated as a cost, or as an action taken after scoring, never as a measurement.
- **Randomisation:** it yields a causal control group without labels, yet is unused.
- **Free timing data:** USSD application servers already see every menu step and can timestamp it.

A recent preprint estimates that transaction features alone barely detect socially engineered payments (AUC about 0.53), while session behaviour lifts this to 0.78–0.85 ([arXiv 2605.31257](https://arxiv.org/pdf/2605.31257)). This suggests the signal lies in behaviour under interaction.

## Core Concept
**Strata and hydrostatic ledger.** Accounts are partitioned into peer strata s by account type (customer, agent, merchant), throughput quantile and tenure band (<30, 30–180, >180 days). Neighbouring strata are merged until each holds at least 200 accounts. Over a rolling window (28 days default; 7–90), four quantities are kept per account i with O(1) updates per event:
- **Displacement δ_i:** throughput (value in plus out) divided by the stratum median.
- **Centre of gravity KG_i:** the value-weighted mean flow height. A flow f has height h_f = 1 − r_f·d_f.
  - r_f is the fraction of the previous six windows in which its counterparty or agent appeared.
  - d_f = min(1, dwell_f/D_s), with dwell from first-in-first-out lot matching (queue capped at 64 lots) and D_s the stratum median dwell.
  - Habitual, rested value sits low (ballast); fresh value leaving to new counterparties sits high (top-weight).
- **Beam b_i:** exp(H_i) divided by the stratum median, clipped to [0.1, 3]. H_i is the Shannon entropy of value shares across recurring counterparties (r ≥ 0.5).
- **Slack fraction q_i:** the share of outgoing value with dwell under 10 minutes.

**Calculated stability.** Box-barge hydrostatics give draft = δ/b and BM = b²/(12·draft), hence:

GM_calc,i = KB_s + b_i³/(12·δ_i) − KG_i − λ·q_i

KB_s is the stratum median of KG (the centroid of the displaced peer medium). λ = 0.5 (range 0.2–1) is the free-surface correction for sloshing pass-through value. A heavily loaded account on a narrow habitual base, carrying top-weight and slack value, computes as "tender". GM_calc ≤ 0 is itself reportable.

**Inclining probe.** Eligible events are irreversible exits (agent cash-out, off-net or bank transfer) and first-time-payee transfers. Eligibility is fixed before randomisation.
- **Baseline rate:** probing probability p0 = 0.02 (range 0.01–0.05).
- **Refit rate:** as ships are re-inclined after refit, the first three exits within 72 h of a refit trigger are probed with probability 0.8, leaving 20% as controls.
- **Refit triggers:** SIM swap, PIN reset, handset change where visible, reactivation after more than 60 days dormant, or a day-on-day GM_calc drop beyond 2 robust standard deviations.
- **Probe P1:** one added USSD screen showing the registered recipient or agent name with confirm/cancel. It is inserted only if less than half the gateway session timeout has elapsed, because completion is reported to collapse once waits consume the session budget ([arXiv 2607.07650](https://arxiv.org/pdf/2607.07650)).
- **Probe P2:** a post-session settling hold of 20 or 45 s before the exit settles, announced by SMS and consuming no session time.

Each probe type has a known intensity φ.

**Heel measurement and score.** For every probed and control event the server logs a response vector y:
- log latency on the last interactive screen, relative to the account's own median;
- cancellation;
- re-dials, balance enquiries, and duplicate or split exit attempts within 10 minutes;
- agent or channel switch;
- log time-to-next-exit.

Heel is the robust Mahalanobis distance θ = sqrt((y − μ_s)ᵀ Σ_s⁻¹ (y − μ_s)), floored at 0.1. μ_s and Σ_s (median/MAD scaling, shrinkage covariance) come from that stratum's randomised controls.

The inclining relation tanθ = w·d/(Δ·GM) supplies the prediction:

θ_pred = κ_{s,p}·φ / (δ_i·max(GM_calc,i, 0.05))

One scale κ per stratum and probe type is fitted by median regression on probed events; isotonic regression of θ on GM_calc is an alternative. The anomaly statistic is S_i, the median over the account's probes of ln(θ_obs/θ_pred). It is standardised to z_i using the stratum median and 1.4826·MAD, then shrunk by n/(n+2) for n probes.

Three signatures follow:
- **Tender (z > 3):** retry storms, splitting or agent-hopping under mild friction, indicating a time-critical exit.
- **Dead ship (z < −3 where θ_pred is high):** no disruption where history predicts one, indicating a new controller or a script.
- **Loll:** no return to within two robust deviations of baseline in 24 h.

**Graded reaction from human-robot interaction.** The probe is the financial analogue of contingency detection, in which a robot emits a signal and tests whether behaviour changes in response. The reaction borrows speed-and-separation monitoring from collaborative robotics (ISO/TS 15066), which slows the robot as separation shrinks and does not halt it outright. The exit-rate cap is:

C_i = C_s·clip((GM_eff − GM_min)/(GM_ref − GM_min), 0.25, 1)

Here GM_eff = min(GM_calc, GM_meas) and GM_meas = κφ/(δ·θ_obs). GM_ref is the stratum median and GM_min its 1st percentile. The 0.25 floor (range 0.1–0.5) means no wallet is frozen automatically. Freezes need an analyst, who receives a daily top-k list with the three largest contributing response components. Probe messages state plainly why the step appears.

**Embodiments.**
- **Zero-friction tier:** exogenous gateway timeouts and session drops serve as natural inclining moments. This assumes outages are independent of account intent within a cell and hour.
- **Optional stronger tier:** holds up to 90 s on refit-triggered exits. Its risk is customer harm and conflict with instant-payment service levels, so it needs regulator consent.
- **Footprint:** state is under about 1 kB per account, roughly 1 GB per million accounts (estimate), on PostgreSQL plus a Python service.

## Physics & Logical Feasibility
The method is computational. Hydrostatic equations supply model structure, with no claim that accounts obey fluid mechanics. The fitted κ absorbs any mismatch. If GM_calc has no relation to heel, θ_pred collapses to a stratum constant and the method degrades gracefully into peer-relative probe-response outlier detection.

Randomisation fixed before treatment gives unbiased control distributions without labels.

Prerequisites are per-step USSD timestamps, the ability to add one screen, and the ability to delay settlement. Per-step timing is available wherever the operator's application server answers each menu request, though network jitter adds noise.

Sample size is the main constraint. As an arithmetic estimate, 50,000 eligible exits a month at p0 = 0.02 gives about 1,000 probes. Across 12 merged strata that is roughly 83 each per month, so about three months to exceed 200 per stratum.

## Implementation Readiness & Key Constraints
- **Months 0–3:** build the hydrostatic ledger and run it in shadow mode on historical logs.
- **Months 3–6:** pilot randomised P1 probes under disclosed terms and regulator or sandbox notification.
- **Months 6–12:** fit κ and compare flags with later-confirmed complaints and reversals. These are used for evaluation only, never for training.
- **Months 12–24:** publish a reference implementation and adapters for common USSD gateways and open payment switches.

Dependencies and challenges:
- MFIs using aggregators cannot see SIM-swap or handset events, so they fall back to PIN-reset and dormancy triggers.
- Small strata limit calibration.
- Analyst capacity sets k.
- Settlement holds must fit scheme rules.
- Customer communication must be worded for low-literacy users.

## Utility & Purpose
Small operators lack labelled fraud data, data-science teams and licence budgets, and their customers are harmed disproportionately by both fraud and wrongful blocking. This method gives them a detector that manufactures its own evidence through tiny randomised experiments. It targets fraud types that passive statistics miss:
- SIM-swap takeovers;
- mule cash-outs racing a reversal;
- coached scam victims;
- scripted agent commission fraud.

Beneficiaries are mobile money operators, microfinance institutions, savings cooperatives, their agents, and low-income users. Users keep access under a slowed but never zero exit rate while a human reviews the case.

## Potential Applications & Impact
- **Operators:** a core fraud engine for small wallets, or a second opinion alongside rule engines at larger ones.
- **Shared infrastructure:** a module for open payment switches and regulator-run monitoring utilities.
- **Research:** a methodology for label-free evaluation of friction policies.
- **Regulators:** the calculated-versus-measured stability pair gives an interpretable vocabulary (ballast, top-weight, slack value, heel) for explaining decisions to non-specialist staff.

More broadly, treating friction as a sensor could transfer to other settings where labels are scarce but small randomised interventions are cheap. Examples are agent-network liquidity monitoring, airtime-credit abuse, benefit-payment integrity and account-recovery flows. Open publication keeps transaction-layer probing with model-predicted response available to public-interest providers.

## Market/Adoption Feasibility (If Applicable)
Cost is dominated by integration: one commodity server and, as an estimate, a few engineer-months to connect the ledger and gateway hooks. There are no licence fees.

Enabling factors already exist:
- USSD application programming interfaces deliver per-step callbacks.
- Name-confirmation screens are familiar to users, as with Safaricom's Hakikisha (from recall).
- Regulatory sandboxes operate in several African markets.

Barriers are regulator comfort with randomised friction, scheme rules on settlement time, and the calibration period for very small institutions. Pooling the κ parameters across cooperating operators, without sharing customer data, could shorten that period.

## Risk & Counterarguments
- **Unproven separability:** it has not been shown that fraud and legitimate behaviour separate under probes. No field data exist, and the tender and dead-ship signatures are hypotheses.
- **Fairness:** low-literacy, elderly or shared-phone users may heel atypically. Stratification and own-baseline latency mitigate this but may not remove it.
- **Friction cost:** added friction can reduce completion, especially near USSD timeouts.
- **Adversarial adaptation:** fraudsters may rehearse calm responses. Two-sided scoring, unpredictable assignment and the cost of building ballast raise their effort, but adaptation remains possible.
- **Exogeneity:** natural-outage probes rest on an exogeneity assumption.
- **Regulation:** randomised treatment of customers needs disclosure, data-protection review and consumer-protection clearance.

## Open Questions & Future Research Directions
- Does the box-barge exponent (b³/δ) fit behavioural data better than a learned monotone form?
- What minimum probe intensity yields usable heel?
- How should probe types be composed into one moment scale?
- Can roll period (events until recovery) give a third stability estimate via the relation that period varies as 1/√GM?
- Can the method extend to app, SIM-toolkit and voice-response channels, or to agent tills?
- Can cross-operator sharing of calibration parameters be made privacy-preserving?
- What sequential-testing rules would stop probing an account once it is confidently stable?
- How should disparate impact be audited formally across literacy, age and region?

## Novelty Threshold Justification
**Medium Novelty**: a cross-domain synthesis whose central mechanism was not found in the prior art. The closest art is BioCatch's invisible-challenge family, which injects interface-level interference and compares reactions with a user-specific model ([US10474815B2](https://patents.google.com/patent/US10474815B2/en)). Peer group analysis and density-based outlier scoring are the closest passive art.

Three elements are new relative to these:
- randomised transaction-layer probes observed purely server-side on USSD;
- response predicted from a passively calculated metacentric height, not from the user's earlier challenge reactions;
- the calculated-versus-measured discrepancy as the anomaly statistic, with refit-triggered re-inclining and a floored exit-rate reaction.

Three searches found no publication of this mechanism. That is not proof of absence.

## References & Prior Art Checks
From recall:
- Bolton & Hand, "Unsupervised Profiling Methods for Fraud Detection" (2001).
- Breunig et al., Local Outlier Factor (SIGMOD 2000).
- Liu, Ting & Zhou, Isolation Forest (ICDM 2008).
- De Luca et al., collision detection and safe reaction (IROS 2006).
- Lee, Chao, Bobick & Thomaz, multi-cue contingency detection (Int. J. Social Robotics, 2012).
- ISO/TS 15066:2016.

Found by search:
- [US10474815B2](https://patents.google.com/patent/US10474815B2/en)
- [US10776476B2](https://patents.google.com/patent/US10776476B2/en)
- [Biometric Update on BioCatch patents](https://www.biometricupdate.com/201612/biocatch-obtains-9-new-patents-in-2016)
- [arXiv 2607.07650](https://arxiv.org/pdf/2607.07650)
- [arXiv 2605.31257](https://arxiv.org/pdf/2605.31257)
- [Neuralt](https://www.neuralt.com/news-insights/ussd-fraud-detection-prevention-mobile-money)

**Prior art searched:**
1. `fraud detection patent injected interference "invisible challenge" user reaction to perturbation behavioral response BioCatch` — used US10474815B2, US10776476B2 and the Biometric Update article.
2. `USSD session keystroke latency timing mobile money fraud detection unsupervised randomized delay probe response` — used arXiv 2607.07650 and the Neuralt page. No hit on the mechanism.
3. `randomized transaction hold OR "active probing" payment fraud detection response to friction as anomaly signal retry behavior cash-out mule unsupervised "metacentric" OR "perturbation response"` — used arXiv 2605.31257. No direct hits.

---

### About the Reports
Each innovation on Unpatentable.org is published as a three-part open documentation set:
- **Innovation Report** – the master blueprint describing the first principals concept, mechanism, and rationale behind the invention.
- **Implementation Guide** – a step-by-step, practical manual detailing how to build or prototype the innovation. Includes estimated cost breakdown.
- **Industry Impact Report** – an analytical overview exploring the potential economic, industrial, and societal effects of the innovation.

Together, these documents form a complete public disclosure package — enabling researchers, developers, and policymakers to understand, replicate, and build upon each invention.

---

### Explore this Innovation
- [Innovation Report](https://unpatentable.org/innovation/inclining-test-account-stability-scoring-for-label-free-mobile-money-fraud-detection)
- [Implementation Guide](https://unpatentable.org/innovation/implementation-guide-inclining-test-account-stability-scoring-for-label-free-mobile-money-fraud-detection)
- [Industry Impact Report](https://unpatentable.org/innovation/industry-impact-report-inclining-test-account-stability-scoring-for-label-free-mobile-money-fraud-detection)

### Download PDFs
- [Innovation Report (PDF)](https://unpatentable.org/wp-content/uploads/2026/11/Inclining-Test-Account-Stability-Scoring-for-Label-Free-Mobile-Money-Fraud-Detection.pdf)
- [Implementation Guide (PDF)](https://unpatentable.org/wp-content/uploads/2026/11/Implementation-Guide-Inclining-Test-Account-Stability-Scoring-for-Label-Free-Mobile-Money-F.pdf)
- [Industry Impact Report (PDF)](https://unpatentable.org/wp-content/uploads/2026/11/Industry-Impact-Report-Inclining-Test-Account-Stability-Scoring-for-Label-Free-Mobile-Money.pdf)

---

**Timestamps:**
- [View Main Innovation Report on Arweave](https://arweave.net/NfjGuk3XnjyO-9859zjx7Y5lhxmcIurNcUd-KlPj1DY)
- [View Implementation Guide on Arweave](https://arweave.net/dlF8HpZ5JNYlYx0vvp65JYtuX8fc5BHY2YfWmnYSTMA)

**Suggested Citation:**
> Unpatentable.org (2026). *Inclining-Test Account Stability Scoring for Label-Free Mobile Money Fraud Detection.* Defensive Publication Series.
> https://unpatentable.org/innovation/inclining-test-account-stability-scoring-for-label-free-mobile-money-fraud-detection

---

```json
{
  "domain": "Digital & Information Technologies",
  "subdomain": "Unsupervised Learning",
  "published": "2026-11-26",
  "arweave_main_tx": "NfjGuk3XnjyO-9859zjx7Y5lhxmcIurNcUd-KlPj1DY",
  "license": "CC BY 4.0",
  "arweave_implementation_tx": "dlF8HpZ5JNYlYx0vvp65JYtuX8fc5BHY2YfWmnYSTMA",
  "value_unlocked": "$41B"
}
```

Unpatentable.org is a public library of timestamped defensive publications —
open innovations released into the commons to prevent monopolization, democratize knowledge, and accelerate human progress.

License: CC BY 4.0
