# Correlation input map

This page answers exactly one question: Which decoded fields feed which correlations, and from where?

*Note: Fields are marked as either decoded (explicitly parsed by Wazuh decoders) or derived/inferred (available via Wazuh timestamps, hostname, or correlation logic).*

## Log source → decoder → fields → correlation

### BMP (BGP Monitoring Protocol)

Purpose: Authoritative observation of BGP control‑plane behaviour. BMP is the *source of truth* for announcements and withdrawals.

Decoder(s):

* `red_lantern_bmp_announcement_decoder`
* `red_lantern_bmp_withdrawal_decoder`

Decoded fields:

* `bmp.prefix`
* `bmp.as_path`
* `bmp.next_hop`
* `bmp.origin_as`

Derived/inferred fields (for correlation):

* event type (`announce` | `withdraw`)
* prefix length
* timestamp

Consumed by correlations:

| Correlation              | Fields used                                      |
|--------------------------|--------------------------------------------------|
| `multi_stage_bgp_attack` | bmp.prefix, bmp.origin_as, event type, timestamp |
| `rpki_cover_hijack`      | bmp.prefix, bmp.origin_as, event type, timestamp |

Notes:

* BMP never goes through syslog
* No validation decisions here
* No intent, only facts

### RPKI validator logs

Purpose: Provide an independent trust signal for prefix–origin legitimacy.

Decoder(s):

* `red_lantern_rpki_decoder`
* `red_lantern_rpki_details_decoder`

Decoded fields:

* `rpki.prefix`
* `rpki.origin_as`
* `rpki.state`

Derived/inferred fields:

* validator name (from hostname if available)
* timestamp (from syslog/Wazuh event)

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

Decoded fields:

* `router.neighbor_state`

Derived/inferred fields:

* timestamp (from syslog)
* router ID (from syslog hostname)

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

* Requires RPKI validator logs
* Does not require BMP or Router syslog
* Function: Detects trust manipulation without routing activity

### rpki_cover_hijack

* Requires BMP and RPKI validator logs
* Optional: Historical ROA change context
* Does not require Router syslog
* Function: Detects active hijack under valid RPKI cover

### multi_stage_bgp_attack

* Requires BMP and RPKI validator logs
* Optional: Router syslog
* Function: Detects full campaign lifecycle, including withdrawals

## What is *not* part of correlations

* Traffic payloads
* NetFlow/IPFIX
* Forwarding‑plane metrics
* Syslog-based BGP parsing

*If it is not authoritative, it does not drive correlation.*

## Mental model

* BMP = *What actually happened*
* RPKI logs = *Why it was trusted*
* Router syslog = *How the device reacted*
* Correlations = *When the story makes sense*

