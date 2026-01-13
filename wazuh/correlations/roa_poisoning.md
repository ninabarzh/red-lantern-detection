# ROA poisoning correlation

## What this correlation detects

A suspected attempt to corrupt RPKI trust by introducing *new or altered ROAs* that legitimise an unexpected prefix–origin pairing before any BGP announcement occurs.

This correlation focuses on trust manipulation, not traffic interception.

## Why this matters

ROA poisoning is often a preparatory step:

* It precedes stealthy sub-prefix hijacks
* It can indicate compromised registry credentials or insider misuse
* It creates a false sense of legitimacy for later routing events

Once a poisoned ROA exists, downstream routing decisions may already be biased in the attacker’s favour.

## Observable signals

This correlation relies only on RPKI validation evidence, not routing activity.

Observable via logs:

* RPKI validators reporting a prefix–origin pair as `valid`
* Agreement across multiple independent validators
* Change in validation state compared to historical baseline (implicit)

Not observed here:

* No BGP announcements
* No withdrawals
* No router acceptance or rejection

If routing is involved, this is no longer ROA poisoning; it becomes a hijack correlation.

## Required log sources

Mandatory

* RPKI validator logs

  * `red_lantern_rpki_decoder`
  * `red_lantern_rpki_details_decoder`
  * Fields used:

    * `rpki.prefix`
    * `rpki.origin_as`
    * `rpki.state`
    * `validator_name`
    * `timestamp`

Optional (context only)

* Authentication or registry access logs
  (used for enrichment, never as a trigger)

## Correlation logic (human-readable)

1. Observe one or more RPKI validators reporting a prefix–origin pair as `valid`
2. Confirm that the same prefix–origin pair is reported as `valid` by multiple independent validators within a bounded time window
3. Verify that no corresponding BGP announcement for the prefix has been observed via BMP during the same window

Raise an alert when validator consensus exists without routing activity.

This is correlation by *absence of routing*, not by presence.

## Temporal considerations

* Validator updates may be asynchronous
* Consensus windows should allow minutes to hours
* Absence of BMP events must be evaluated over a longer window than validator convergence

This is slow, quiet groundwork — not a race condition.

## Assumptions and limitations

Assumptions:

* Validators provide trustworthy views of RPKI state
* BMP coverage exists for the monitored routing domain (to confirm absence)

Limitations:

* Legitimate ROA creation or updates may trigger alerts
* Historical baseline is required for strong confidence
* Cannot identify the actor without external registry logs

This correlation detects suspicious trust changes, not guilt.

## Expected outcome

A triggered correlation indicates:

* A prefix has gained RPKI legitimacy unexpectedly
* Multiple validators agree on the endorsement
* No routing activity yet reflects that endorsement

Recommended actions:

* Flag the prefix for heightened BMP monitoring
* Review recent ROA changes with the registry or RIR
* Prepare filters or mitigations in advance of possible hijack activity

Reality check:

This correlation fires rarely, and that is the point.
If it fires often, the baseline is wrong or validators are too noisy.

It is an early-warning sensor, not an alarm bell.
