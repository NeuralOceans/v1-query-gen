---
name: v1-query-gen
description: Generate validated Trend Vision One custom filter queries from vendor log documentation. Use when creating detection models from syslog samples, security event logs, or vendor documentation.
disable-model-invocation: true
---

# Vision One Custom Filter Query Generator

**Copyright (c) 2026 Scarlett Menendez. All Rights Reserved.**

Licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

## Purpose

Generate validated Trend Vision One custom filter queries from any vendor's log documentation. This skill is vendor-agnostic and processes syslog samples, security events, or log documentation to create detection model queries using the `vendorParsed` field format.

## Input

Provide log documentation containing:
- Product/Vendor name
- Event types
- Event descriptions
- Sample syslog messages
- Severity levels (optional)

**Example Input:**
```
Product: Cisco ASA Firewall
Event Type: %ASA-4-106023
Description: Denied IP packet
Sample: %ASA-4-106023: Deny tcp src inside:192.168.1.100/12345 dst outside:10.0.0.1/443 by access-group "inside_access_in"
Severity: High
```

## Process

### Step 1: Parse Log Documentation
Extract from the provided documentation:
- **Product/Vendor name** (e.g., "Meraki MX Security Appliance", "Palo Alto Firewall")
- **Event types** (e.g., "ids-alerts", "security_event", "firewall")
- **Event descriptions** (e.g., "IDS signature matched", "Malicious file blocked")
- **Sample syslog messages** (raw log examples)
- **Severity levels** (Critical, High, Medium, Low, Informational)

### Step 2: Analyze Sample Syslog Messages
For each sample syslog message, identify:

**Key=Value pairs** (query directly):
| Syslog Pattern | Vision One Query |
|----------------|------------------|
| `type=value` | `vendorParsed.type: value` |
| `action=block` | `vendorParsed.action: block` |
| `direction=ingress` | `vendorParsed.direction: ingress` |
| `disposition=malicious` | `vendorParsed.disposition: malicious` |
| `decision=blocked` | `vendorParsed.decision: blocked` |
| `state=start` | `vendorParsed.state: start` |

**Free-form text** (no key=value structure):
- Use `vendorParsed.msg: "*keyword*"` format
- Combine: `vendorParsed.msg: "*keyword1*" AND vendorParsed.msg: "*keyword2*"`

### Step 3: Build Vision One Queries

**Syntax Rules:**
```
vendorParsed.<fieldname>: <value>
```

**Operators:**
- `AND` - Both conditions must match
- `OR` - Either condition matches

**Wildcards:**
- `*` - Matches any characters
- MUST have at least one defined value (not all wildcards)

**String Matching:**
- Partial: `vendorParsed.field: *value*`
- Exact: `vendorParsed.field: "exact value"`
- Contains: `vendorParsed.msg: "*contains this*"`

### Step 4: Validate Queries

Before presenting, verify:
- [ ] At least one field has a defined value (not just wildcards)
- [ ] Field names match syslog key names exactly
- [ ] Wildcards used appropriately for partial matching
- [ ] Correct AND/OR operators
- [ ] Severity matches security impact

## Output Format

Present results in this exact table format:

```
| Product | Event Type | Description | Custom Filter Query | Severity | Sample Syslog Message |
|---------|------------|-------------|---------------------|----------|----------------------|
| [Vendor Product] | [event_type] | [Description] | [vendorParsed query] | [Severity] | [sample log] |
```

## Field Mapping Reference

| Syslog Key | Vision One Field |
|------------|------------------|
| `type=` | `vendorParsed.type` |
| `action=` | `vendorParsed.action` |
| `decision=` | `vendorParsed.decision` |
| `direction=` | `vendorParsed.direction` |
| `disposition=` | `vendorParsed.disposition` |
| `state=` | `vendorParsed.state` |
| `protocol=` | `vendorParsed.protocol` |
| `src=` | `vendorParsed.src` |
| `dst=` | `vendorParsed.dst` |
| `pattern=` | `vendorParsed.pattern` |
| `priority=` | `vendorParsed.priority` |
| `signature=` | `vendorParsed.signature` |
| `spi=` | `vendorParsed.spi` |
| `identity=` | `vendorParsed.identity` |
| Free-form text | `vendorParsed.msg` |

## Query Pattern Examples

| Log Structure | Query Pattern |
|---------------|---------------|
| Has `type=eventname` | `vendorParsed.type: eventname` |
| Has `action=value` | `vendorParsed.action: value` |
| Has `decision=blocked` | `vendorParsed.decision: blocked` |
| Has two key=value pairs | `vendorParsed.field1: value AND vendorParsed.field2: value` |
| Free-form with keywords | `vendorParsed.msg: "*keyword1*" AND vendorParsed.msg: "*keyword2*"` |
| Multiple event types | `vendorParsed.type: event1 OR vendorParsed.type: event2` |

## Notes

- Queries are for Trend Vision One Third-Party Log Collection
- Uses `vendorParsed` field containing parsed syslog data
- Always test in Vision One "Validate Query" before deployment
- For logs without key=value pairs, use `vendorParsed.msg` with wildcards

---

**Author:** Scarlett Menendez
**Version:** 1.0.0
**Created:** 2026-02-27
