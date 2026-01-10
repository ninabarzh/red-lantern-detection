# Multi-stage BGP attack correlation

## What this correlation detects

A deliberate, multi-phase BGP control-plane campaign in which an adversary:

1. Manipulates RPKI state to appear legitimate
2. Announces a sub-prefix that passes RPKI validation
3. Achieves traffic redirection or interception
4. Withdraws the route after objectives are met

Unlike noisy hijacks, this is a patient, trust-abusing operation rather than a blunt routing disruption.

## Why this matters

Single alerts are often ambiguous:

* Route leaks occur
* Misconfigurations happen
* Validators may disagree

This attack only becomes visible when multiple signals converge: BGP behaviour, operational acceptance, and validator consensus. Single-signal alerts either miss the attack or generate false positives.

## Observable signals

Detectable without packet inspection or probing:

* BGP announcements and withdrawals for monitored prefixes
* Route attributes (origin AS, AS_PATH, next hop)
* Router-reported acceptance or rejection of routes
* Independent RPKI validator confirmation of validation state

No single signal suffices; combined, they reveal a full control-plane compromise.

## Required log sources

* BMP (BGP Monitoring Protocol) – authoritative announcement and withdrawal events
* RPKI validator logs – authoritative validation state from one or more validators
* Router syslog (optional) – operational context only (session stability, acceptance timing)

Optional: authentication logs, flow telemetry. Router logs are **not** a source of routing truth.

## Correlation logic (human-readable)

1. Observe BGP announcement for a monitored prefix/sub-prefix via BMP
2. Router logs indicate acceptance in the control plane (optional)
3. One or more RPKI validators confirm the prefix and origin AS as `valid` within a short time window
4. Observe subsequent withdrawal of the prefix

Raise a high-confidence alert if all steps occur in sequence.

## Temporal considerations

* Announcement → validation: seconds–minutes
* Withdrawal may occur minutes → hours → days
* Support asymmetric correlation windows

This is a short, deliberate campaign, not a burst of events.

## Assumptions and limitations

* BMP coverage exists for relevant peers
* Validator results reflect authoritative RPKI state
* Router logs indicate timing of acceptance (optional)

Limitations:

* Withdrawals may be delayed or missing
* Partial visibility reduces confidence

## Evasion considerations

Adversaries may:

* Use only a single validator to avoid consensus signals
* Spread activity across longer windows
* Announce prefixes just outside monitored ranges
* Avoid withdrawal to appear stable

These actions increase operational cost for the attacker.

## Expected outcome

A triggered correlation indicates:

* A suspicious prefix was announced
* Routing infrastructure accepted it
* Validators endorsed the announcement

Actionable: reroute traffic, filter prefixes, audit trust anchors.

## Related playbooks

* Playbook 1: RPKI presence establishment (baseline)
* Playbook 2: ROA poisoning and validation mapping
* Playbook 3: Sub-prefix hijack under RPKI validation cover
