# TA-hexward — Splunk add-on for Hexward Labs

Search, alert and report on findings from [Hexward Labs](https://hexwardlabs.com)
tools — RuleHawk, Loglight, CertLight, Decoy, DmarcWatch, Patchlight,
TenantWatch, Attack Surface Monitor, AuditLight, TopoLight — in Splunk.

The products send the [Hexward Finding Schema](https://github.com/nizartuanku/hexward-core/blob/main/SCHEMA.md)
(`hexward.finding/1`) with sourcetype `hexward:finding`. This add-on parses
it, maps it to CIM field names, and ships ready-made searches and a
dashboard. Free in every edition of every Hexward product.

## What it adds

- **Sourcetype `hexward:finding`**: JSON parsing (`KV_MODE = json`), timestamp
  from `ts` (UTC), one event per line.
- **CIM fields (Alerts data model):** `dest` (target), `signature` (title),
  `signature_id` (check), `id` (fingerprint), `src` (source IP when known),
  `severity` (`critical` … `informational`), `description` (remediation),
  `app`, `vendor_product`, `type`. Opened findings carry the `alert` tag.
- **Event types:** `hexward_finding`, `hexward_finding_opened`,
  `hexward_finding_resolved`.
- **Saved searches** (not scheduled, no actions — enable what you want):
  open critical and high findings, new findings in the last 24 hours, mean
  time to resolve by product.
- **Dashboard "Hexward findings"**: open count, opened by severity, opened vs
  resolved over time, latest findings.

## Install

Install on your search heads and indexers (or Splunk Cloud), like any
add-on: *Apps → Install app from file* with the `.tgz` from the
[releases](https://github.com/nizartuanku/TA-hexward/releases), or copy the
`TA-hexward` folder into `$SPLUNK_HOME/etc/apps/` and restart.

## Send findings

### HTTP Event Collector (recommended)

Create a HEC token (*Settings → Data inputs → HTTP Event Collector*), allow
the index you want, save the token to a file, then start the product with:

```sh
rulehawk -hec-url https://splunk.example:8088 -hec-token-file /etc/hexward/hec.token -hec-index hexward
```

The product sets `sourcetype=hexward:finding` and `source=hexward:<product>`
itself. Use `-hec-ca-file` when Splunk uses a private CA; certificates are
always verified.

### Universal Forwarder

```sh
rulehawk -export-file /var/log/hexward/rulehawk.ndjson
```

```ini
# inputs.conf on the forwarder
[monitor:///var/log/hexward/*.ndjson]
sourcetype = hexward:finding
index = hexward
```

### Syslog

Send `-syslog-format json` to your syslog input and set the input's
sourcetype to `hexward:finding`. `-syslog-format cef` also works with any
CEF-aware pipeline.

## Load what already exists

Live export carries changes. To load current findings once:

```sh
curl -s http://127.0.0.1:8426/api/findings/export?format=hfs > snapshot.ndjson
```

and upload it with sourcetype `hexward:finding` (or append it to the
monitored file).

## Tested

Splunk Enterprise 10.6.1 (Docker), October 2026: RuleHawk 0.1.3 sending ten
findings over HTTP Event Collector; every event parsed with its CIM fields,
event types and `alert` tag; all three saved searches and the dashboard
searches return the expected results, including mean time to resolve after
a `resolved` event.

The add-on maps field names for the CIM Alerts data model. It has not been
run through Splunk AppInspect or listed on Splunkbase yet.

## License

Apache-2.0. © Nizar Tuanku — Cybersecurity.
