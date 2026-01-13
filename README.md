# Red Lantern detection

This repository contains detection logic for the Red Lantern threat simulation.  
It is designed to be modular, orthogonal, and easy to extend with new log sources or SIEMs.

## Wazuh

```
wazuh/
├── decoders/   # Wazuh decoders for parsing logs
├── rules/      # Wazuh rules referencing decoders
```

- Decoders parse logs into fields.  
- Rules generate alerts.  
- Rule IDs are in the 91000–93999 range to avoid collisions.  

## Splunk

```
splunk/
├── inputs.conf         # log ingestion / sourcetype definitions
├── savedsearches.conf  # scheduled alerts
└── detection_spl/      # SPL queries for interactive searches or alerts
└── correlations/       # SPL queries for interactive searches or alerts
```

- SPL queries detect BGP hijacks, RPKI validation failures, and multi-stage attacks.  
- Each detection type has its own `.spl` file.  

## Elastic Security

```
elastic/detections_kql/
```

- KQL queries for Red Lantern events.  
- Each detection is a separate `.kql` file.  
- Queries assume correct ECS mapping for log fields.  

## Microsoft Sentinel

```
sentinel/analytics_rules_kql/
```

- KQL analytics rules for Sentinel.  
- Each rule is a separate `.kql` file.  
- Covers BGP hijack, RPKI failure, and multi-stage attack correlations.  

## Google Chronicle

```
chronicle/yara_l_rules/
```

- YARA-L detection rules.  
- Each rule is in its own `.yaral` file.  
- Rules cover BGP hijack, RPKI failures, and multi-event correlations.

## Adding new detections

1. Determine the SIEM and log source.  
2. Add decoders or ingestion configuration if required (Wazuh or Splunk).  
3. Add detection rules or queries in the appropriate folder.  
4. Follow naming conventions for clarity:
   - Wazuh decoders: `red_lantern_<component>.xml`
   - Wazuh rules: `red_lantern_<component>.xml`
   - SPL/KQL/YARA-L: `<component>_<detection>.spl/kql/yaral`
5. Update tests and/or saved searches if applicable.  

## Notes

- Wazuh is the only platform using a decoder layer.  
- All other SIEMs rely on their native ingestion/parsing mechanisms.
