# RPKI cover hijack correlation

## What this correlation detects

An active BGP hijack or interception attempt in which an adversary:

1. Announces a prefix or sub-prefix via BGP
2. Ensures the prefix–origin pair appears RPKI-`valid`
3. Achieves routing stability under that validation
4. Later withdraws or alters the route without a clear operational cause

This is a trust-abusing hijack: the route looks legitimate because the trust layer has been shaped to allow it.

## Why this matters

Most hijack detections rely on obvious failure modes:

* `invalid` RPKI state
* sudden origin changes
* large-scale route leaks

This attack avoids all three.

If detection is limited to single-signal alerts, it is either blind or unbearably noisy. Only correlation across authoritative routing events and independent trust signals exposes the behaviour.

## Observable signals

Observable without packet inspection or traffic telemetry:

* BGP announcements and withdrawals observed via BMP
* Stable route presence for a limited duration
* RPKI validator results indicating `valid` for the observed prefix–origin pair

Explicitly not observed:

* Payloads
* Traffic volume
* User impact
* Forwarding-plane metrics

This correlation is entirely control-plane driven.

## Required log sources

Mandatory

* BMP (BGP Monitoring Protocol)

  * Authoritative announcements and withdrawals
  * Fields:

    * `prefix`
    * `origin_as`
    * `bmp_event_type` (`announce` | `withdraw`)
    * `timestamp`

* RPKI validator logs

  * Validation outcome for the same prefix–origin pair
  * Fields:

    * `rpki.prefix`
    * `rpki.origin_as`
    * `rpki.state`
    * `validator_name`
    * `timestamp`

Optional (context only)

* Historical ROA state or ownership metadata
  (used for enrichment, not for triggering)

Router syslog is not required and does not provide routing truth.

## Human-readable correlation logic

1. Observe a BGP announcement for a monitored prefix or sub-prefix via BMP
2. Confirm that one or more RPKI validators report the prefix–origin pair as `valid` within a bounded time window
3. Observe continued presence of the route (no immediate withdrawal)
4. Observe a subsequent withdrawal or route change for the same prefix–origin pair

Raise a high-confidence alert when a route is:

* RPKI-valid
* Operationally stable
* Then deliberately removed

This pattern is inconsistent with accidental misconfiguration and consistent with tactical use.

## Temporal considerations

* Announcement and validation typically occur within seconds to minutes
* Route presence may last minutes to hours (or longer)
* Withdrawal may be delayed and is not required to be symmetric with announcement timing

Correlation windows must support *long-tail withdrawals*.

## Assumptions and limitations

Assumptions:

* BMP coverage exists for relevant peers
* RPKI validators provide independent confirmation

Limitations:

* Hijacks that remain indefinitely may not trigger withdrawal-based logic
* Partial BMP visibility reduces confidence, not correctness
* Detection is intentionally late-stage to reduce false positives

This correlation trades speed for certainty.

## Evasion considerations

An adversary may attempt to evade detection by:

* Maintaining the route indefinitely
* Announcing only exact prefixes
* Restricting propagation to limit BMP visibility

Each evasion increases operational risk and reduces attacker flexibility.

## Expected outcome

A triggered correlation indicates:

* A prefix was announced under valid RPKI cover
* Routing infrastructure accepted it as legitimate
* The route was used tactically rather than operationally

Recommended actions:

* Treat as a control-plane security incident
* Apply immediate prefix filtering or de-preference
* Audit ROAs and trust anchors
* Increase monitoring for related prefixes and AS paths

This is not an anomaly.
This is a hijack that knew the rules.
