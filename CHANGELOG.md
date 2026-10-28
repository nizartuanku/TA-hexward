# Changelog

## v0.1.0 — 2026-10-28

First release. Sourcetype `hexward:finding` (JSON, timestamp from `ts`), CIM field aliases for the Alerts data model, event types `hexward_finding`, `hexward_finding_opened`, `hexward_finding_resolved` with the `alert` tag on opened findings, three saved searches (open critical/high, new in 24 h, mean time to resolve by product) and the "Hexward findings" dashboard. Tested on Splunk Enterprise 10.6.1 with RuleHawk 0.1.3 sending over HTTP Event Collector.
