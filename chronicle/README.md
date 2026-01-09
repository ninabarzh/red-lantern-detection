# Google Chronicle

This folder contains YARA-L detection rules for Red Lantern events.

## Folder structure

- `yara_l_rules/` – each file is a single YARA-L detection rule.
  - Example files:
    - `bgp_hijack_detection.yaral`
    - `rpki_validation_failure.yaral`
    - `multi_event_correlation.yaral`

## Adding new detections

1. Write a new YARA-L rule in its own `.yaral` file.
2. Name it descriptively, e.g., `<component>_<detection>.yaral`.
3. Ensure the event fields referenced in the rule match the Chronicle ingestion schema.
4. Deploy the rule into Chronicle ruleset.
