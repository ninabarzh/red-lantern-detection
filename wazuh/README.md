# Wazuh

This folder contains Wazuh decoders and rules for Red Lantern detections.

The detections process realistic BGP, BMP, RPKI, and router syslog events produced by the Red Lantern simulator and comparable real-world sources.

## Folder structure

* `decoders/` – Wazuh decoders (`<decoder>` elements). Each file corresponds to one log source or protocol family:
  * `red_lantern_syslog.xml` – router syslog (BGP session state, updates, withdrawals)
  * `red_lantern_bmp.xml` – BGP Monitoring Protocol (BMP) events
  * `red_lantern_rpki.xml` – RPKI validation and ROA-related logs

* `rules/` – Wazuh rules (`<group>` and `<rule>` elements). Each file defines single-event detections for one category:
  * `red_lantern_syslog.xml` – syslog-based BGP events
  * `red_lantern_bmp.xml` – BMP route announcements and withdrawals
  * `red_lantern_rpki.xml` – RPKI validation and ROA events

* `correlations/` – documentation and encodings for multi-event and multi-stage attacks. These are *not executed by Wazuh rules* and are provided for correlation engines, SIEMs, or analytical reference.

## Naming and IDs

* *Decoder names* follow the pattern: `red_lantern_<component>_decoder`
* *Rule groups* align with decoder scope, for example: `red_lantern.bgp`, `red_lantern.bmp`, `red_lantern.rpki`
* *Rule IDs* are allocated in the range *10000–13999*, which is safe for local custom rules and avoids collisions with Wazuh core rules.

## Detection scope

* These rules perform *single-event detection only*.
* No correlation, suppression, or multi-event logic is implemented in Wazuh rules.
* All patterns reflect *realistic operational logs*, not simulator-only artefacts.

## Testing and validation

Any tests present serve only as *regex and structure sanity checks*.
Authoritative validation requires loading the decoders and rules into Wazuh and replaying log data.

## Adding new detections

1. Add a decoder in `decoders/` if a new log source or message format is introduced.
2. Add rules in `rules/` referencing the decoder with `<decoded_as>`.
3. Allocate rule IDs in the next available block within the 91000–93999 range.
4. Validate behaviour in a running Wazuh environment.

