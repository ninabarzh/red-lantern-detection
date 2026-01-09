# Splunk

This folder contains Splunk configurations and detection queries for Red Lantern events.

## Folder structure

- `inputs.conf` – defines monitored files and sourcetypes (ingestion).
- `savedsearches.conf` – scheduled alerts.
- `detection_spl/` – SPL queries as `.spl` files for interactive searches or alerts.

## Adding new detections

1. Add log inputs or sourcetype mappings in `inputs.conf` if a new log source is introduced.
2. Write a new SPL query for the detection and save it in `detection_spl/`.
3. Optionally, add a scheduled search in `savedsearches.conf` referencing the SPL query.
4. Name SPL files clearly, e.g., `bgp_hijack_detection.spl`.
