# Correlation input map (one‑page)

This page answers exactly one question: Which decoded fields feed which correlations, and from where?

## Log source → decoder → fields → correlation

### BMP (BGP Monitoring Protocol)

Purpose: Authoritative observation of BGP control‑plane behaviour. BMP is the *source of truth* for announcements and withdrawals.

Decoder(s):

* `red_lantern_bmp_announcement_decoder`
* `red_lantern_bmp_withdrawal_decoder`

Key decoded fields:

* `prefix`
* `prefix_length`
* `origin_as`
* `as_path`
* `next_hop`
* `peer_as`
* `bmp_event_type` (`announce` | `withdraw`)
* `timestamp`

Consumed by correlations:

| Correlation              | Fields used                                  |
|--------------------------|----------------------------------------------|
| `multi_stage_bgp_attack` | prefix, origin_as, bmp_event_type, timestamp |
| `rpki_cover_hijack`      | prefix, origin_as, bmp_event_type, timestamp |

Notes:

* BMP never goes through syslog
* No validation decisions here
* No intent, only facts

### RPKI validator logs

Purpose: Provide an independent trust signal for prefix–origin legitimacy.

Decoder(s):

* `red_lantern_rpki_decoder`
* `red_lantern_rpki_details_decoder`
* (Optional) per‑validator variants if formats differ

Key decoded fields:

* `rpki.prefix` – the announced prefix
* `rpki.origin_as` – origin AS of the prefix
* `rpki.state` – validation state (`valid` | `invalid` | `not_found`)
* `validator_name` – optional, from hostname if available
* `timestamp` – optional, from syslog timestamp

Consumed by correlations:

| Correlation              | Fields used                                             |
|--------------------------|---------------------------------------------------------|
| `roa_poisoning`          | rpki.prefix, rpki.origin_as, rpki.state, validator_name |
| `rpki_cover_hijack`      | rpki.prefix, rpki.origin_as, rpki.state                 |
| `multi_stage_bgp_attack` | rpki.prefix, rpki.origin_as, rpki.state                 |

Notes:

* Multiple validators increase confidence in assessment
* Disagreements between validators are meaningful signals, not errors
* These logs provide trust context, not routing events

### Router syslog (control‑plane state only)

Purpose: Operational context, not routing truth. Only neighbor session changes are captured here.

Decoder(s):

* `red_lantern_bgp_adjchange_decoder`
* `red_lantern_bgp_neighbor_state_decoder`

Key decoded fields:

* `router.neighbor_state` – state of the BGP session (`up` | `down`)
* `timestamp` – from syslog message
* `router_id` – optional, can be inferred from syslog hostname if needed

Consumed by correlations:

| Correlation              | Fields used                      |
|--------------------------|----------------------------------|
| `multi_stage_bgp_attack` | router.neighbor_state, timestamp |

Notes:

* No BGP announcements are included here
* No withdrawals are reported
* This source only confirms operational state or acceptance impact
* Optional; never authoritative

## Correlation → required inputs summary

### roa_poisoning

Requires RPKI validator logs

Explicitly does NOT require: BMP and Router syslog

Function: Detects trust manipulation without routing activity

### rpki_cover_hijack

Requires BMP and RPKI validator logs

Optional: Historical ROA change context

Explicitly does NOT require *router syslog*

Function: Detects active hijack under valid RPKI cover

### multi_stage_bgp_attack

Requires: BMP and RPKI validator logs

Optional: Router control‑plane syslog

Function: Detects full campaign lifecycle, including withdrawal

## What is *not* part of correlations

Explicitly excluded:

* Simulator training events
* Traffic payloads
* NetFlow/IPFIX
* Forwarding‑plane metrics
* Syslog‑based BGP parsing

If it is not authoritative, it does not drive correlation.

## Mental model (keep this)

* BMP = *What actually happened*
* RPKI logs = *Why it was trusted*
* Router syslog = *How the device reacted*
* Correlations = *When the story makes sense*
