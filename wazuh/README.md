# Wazuh

This folder contains Wazuh decoders and rules for Red Lantern detections.

## Folder structure

- `decoders/` – Wazuh decoders (`<decoder>` elements). Each file is one log source:
  - `red_lantern_syslog.xml` – syslog BGP events
  - `red_lantern_bmp.xml` – BGP monitoring
  - `red_lantern_rpki.xml` – RPKI validation logs

- `rules/` – Wazuh rules (`<group>` + `<rule>` elements). Each file is a category:
  - `red_lantern_syslog.xml` – syslog BGP events
  - `red_lantern_bmp.xml` – BGP monitoring
  - `red_lantern_rpki.xml` – rules for RPKI events

## Naming and IDs

- Decoders: `red_lantern_<component>_decoder`
- Rules: `<group>` names match decoders, e.g., `red_lantern.bgp`
- Rule IDs: 91000–93999 range to avoid collisions with Wazuh core rules

## Adding new detections

1. Add a decoder in `decoders/` if parsing a new log source.
2. Add rules in `rules/` referencing the decoder with `<decoded_as>`.
3. Assign new rule IDs in the next free 100-range.
4. Update tests under `tests/wazuh/` if applicable.
