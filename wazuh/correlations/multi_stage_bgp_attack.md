# Multi-stage BGP attack correlation

## What this correlation detects

A deliberate, multi‑phase BGP control‑plane campaign in which an adversary:

1. Establishes or exploits a *valid* RPKI state for a prefix
2. Announces a (sub‑)prefix that passes RPKI validation
3. Has the route accepted by routing infrastructure
4. Withdraws the route once objectives are met

This is not a noisy hijack. It is a patient, trust‑abusing operation designed to blend into normal routing behaviour.

Single signals are weak and misleading:

* Route leaks happen
* Operators make mistakes
* Validators can temporarily disagree

None of these, in isolation, justify escalation.
This attack only becomes visible when independent control‑plane signals converge: routing behaviour over time, validation state, and (optionally) operational acceptance.

Treating any one of these as decisive either misses the attack or floods analysts with false positives.

## Observable signals

Detectable without packet inspection, probing, or traffic capture:

* BGP announcements and withdrawals for monitored prefixes (via BMP)
* Route attributes (origin AS, AS_PATH, next hop)
* Independent RPKI validator validation state
* Router‑reported control‑plane acceptance timing (optional)

No single signal is sufficient. The correlation depends on *sequence* and *consistency* across sources.

## Required log sources

* *BMP (BGP Monitoring Protocol)*, Authoritative source for announcements and withdrawals
* *RPKI validator logs*, Authoritative source for prefix–origin validation state (one or more validators)
* *Router syslog (optional)*, Operational context only (session stability, acceptance timing)

Optional but not required: authentication logs, flow telemetry.
Router logs are not a source of routing truth.

## Correlation logic (human‑readable)

1. Observe a BGP announcement for a monitored prefix or sub‑prefix via BMP
2. (Optional) Router logs indicate acceptance in the control plane
3. One or more RPKI validators report the prefix–origin pair as `valid` within a bounded time window
4. Observe a subsequent withdrawal of the same prefix via BMP

Raise a high‑confidence alert when all steps occur in sequence.

This is sequence correlation, not thresholding.

## Temporal considerations

* Announcement → RPKI validation: seconds to minutes
* Withdrawal: minutes to hours (sometimes days)
* Correlation windows must be asymmetric

This reflects deliberate operator behaviour, not bursty automation.

## Assumptions and limitations

Assumptions:

* BMP visibility exists for relevant peers
* RPKI validator logs reflect authoritative state
* Router syslog, if present, reflects acceptance timing only

Limitations:

* Withdrawals may be delayed or absent
* Partial visibility lowers confidence, not correctness
* Absence of router logs reduces context, not validity

## Evasion considerations

An adversary may attempt to evade detection by:

* Using a single validator to avoid consensus signals
* Extending activity over longer time windows
* Announcing prefixes just outside monitored ranges
* Avoiding withdrawal to appear stable

All of these increase operational cost and exposure.

## Expected outcome

A triggered correlation indicates:

* A suspicious prefix was announced
* It was accepted by routing infrastructure
* Validators endorsed the announcement
* The route lifecycle matches deliberate campaign behaviour

Reality check:

This is *actionable intelligence*, not a warning light.

Typical actions include prefix filtering, traffic rerouting, trust‑anchor review, and incident escalation.
