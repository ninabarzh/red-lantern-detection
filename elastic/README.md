# Elastic Security

This folder contains KQL detection rules for Elastic Security / Kibana.

## Folder structure

- `detections_kql/` – each file contains a KQL rule to detect Red Lantern events.
  - Example files:
    - `bgp_hijack_detection.kql`
    - `rpki_failure_threshold.kql`

## Adding new detections

1. Ensure the log source maps correctly to ECS fields used in the KQL query.
2. Create a new `.kql` file for the detection.
3. Name the file descriptively, e.g., `new_detection_type.kql`.
4. Update any Elastic detection engine rules to import the new KQL file.
