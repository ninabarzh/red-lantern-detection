# Microsoft Sentinel

This folder contains KQL analytics rules for Red Lantern detections in Sentinel.

## Folder structure

- `analytics_rules_kql/` – each file is a separate analytics rule in KQL format.
  - Example files:
    - `sentinel_bgp_hijack.kql`
    - `sentinel_rpki_failure.kql`
    - `sentinel_multi_stage_attack.kql`

## Adding new detections

1. Make sure the logs are ingested into Sentinel and mapped to the fields referenced in the KQL.
2. Write a new KQL file for the detection.
3. Name files clearly, following `sentinel_<component>_<detection>.kql`.
4. Deploy the new analytics rule in Microsoft Sentinel.
