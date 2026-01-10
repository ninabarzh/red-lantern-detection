# ROA poisoning correlation

## What this correlation detects

A targeted attempt to manipulate RPKI state to give an attacker’s prefix apparent legitimacy:

1. A malicious or compromised actor creates fraudulent ROAs for victim prefixes  
2. Independent validators report these ROAs as `valid`  
3. No actual BGP announcement occurs yet; routing infrastructure is unaware  

This is a **pre-attack reconnaissance / preparation phase**, focusing on trust corruption rather than traffic interception.

## Why this matters

ROA poisoning precedes hijacks. Detecting it early:

* Gives network operators days of warning  
* Highlights compromised registry credentials or insider threats  
* Provides context for later BGP-based hijacks  

Single RPKI validation logs may be insufficient; correlation across validators increases confidence.

## Observable signals

* ROA creation requests in RPKI validator logs  
* Validation state reported as `valid` by multiple validators  
* Authentication events associated with ROA creation (optional)

No BMP or router logs are required because no BGP announcement occurs yet.

## Required log sources

* **RPKI validator logs**  
  * `red_lantern_rpki_decoder`  
  * `red_lantern_rpki_details_decoder`  
  * Fields: `rpki.prefix`, `rpki.origin_as`, `rpki.state`, `validator_name`, `timestamp`  
* **Optional authentication logs** (registry or router admin access)  

## Correlation logic (human-readable)

1. Detect a ROA creation request from a user / admin  
2. Observe one or more independent validators marking the prefix as `valid`  
3. If multiple validators confirm, trigger a medium-to-high confidence alert for ROA poisoning  
4. Optionally, cross-reference authentication logs to identify the responsible actor  

This correlation **does not require BMP or router syslog**, because no actual route is announced.

## Temporal considerations

* ROA creation and validation confirmation may occur within minutes to hours  
* Correlation windows should support asynchronous validator updates  

## Assumptions and limitations

* Validators used are uncompromised  
* Some ROA changes may be legitimate; context matters  
* Does not detect active hijacks; only trust manipulation  

## Expected outcome

An alert indicates:

* A prefix may be fraudulently endorsed in RPKI  
* Further monitoring of BGP announcements for the same prefix is recommended  
* Investigation of registry/admin activity may be warranted
