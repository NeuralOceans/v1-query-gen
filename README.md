# Vision One Query Generator

A Claude Code skill that automatically creates security detection queries from log documentation.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## What Does This Skill Do?

When you have security logs from devices like firewalls, switches, or access points, you need to create detection rules to alert on suspicious activity. This skill:

1. Reads your vendor's log documentation (syslog samples, event descriptions)
2. Analyzes the log structure to identify searchable fields
3. Generates validated queries for Trend Vision One
4. Outputs a ready-to-use table you can import into your security platform

No coding required. Just paste your log documentation and get working queries.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Example Prompts](#example-prompts)
- [Expected Output](#expected-output)
- [Troubleshooting](#troubleshooting)
- [Sources](#sources)
- [License](#license)

## Prerequisites

**What You Need:**
1. Claude Code CLI
2. A terminal/command prompt
3. Log documentation from your security vendor (Meraki, Cisco, Palo Alto, etc.)

**Installing Claude Code:**

Windows (PowerShell):
```powershell
npm install -g @anthropic/claude-code
```

Mac/Linux:
```bash
npm install -g @anthropic/claude-code
```

Verify installation:
```bash
claude --version
```

## Installation

**Step 1: Download the Skill**

Clone from GitHub:
```bash
git clone https://github.com/NeuralOceans/v1-query-gen.git
```

Or download the ZIP from the GitHub repository.

**Step 2: Copy to Claude Skills Directory**

Windows:
```powershell
xcopy v1-query-gen %USERPROFILE%\.claude\skills\v1-query-gen\ /E /I
```

Mac/Linux:
```bash
cp -r v1-query-gen ~/.claude/skills/
```

**Step 3: Verify Installation**

Start Claude Code and type `/v1-query-gen`. If installed correctly, Claude will recognize the skill.

## Quick Start

**Step 1:** Start Claude Code
```bash
claude
```

**Step 2:** Invoke the skill with your log data
```
/v1-query-gen

Product: My Firewall
Event Type: blocked_connection
Description: Blocked inbound connection
Sample: timestamp=1234567890 action=block src=192.168.1.100 dst=10.0.0.1 protocol=tcp
Severity: High
```

**Step 3:** Copy the generated query into Trend Vision One

## Example Prompts

### Example 1: Simple Firewall Log

**Input:**
```
/v1-query-gen

Product: Cisco ASA Firewall
Event Type: %ASA-4-106023
Description: Denied IP packet
Sample: %ASA-4-106023: Deny tcp src inside:192.168.1.100/12345 dst outside:10.0.0.1/443 by access-group "inside_access_in"
Severity: High
```

**Output:**

| Product | Event Type | Description | Custom Filter Query | Severity | Sample Syslog Message |
|---------|------------|-------------|---------------------|----------|----------------------|
| Cisco ASA Firewall | %ASA-4-106023 | Denied IP packet | `vendorParsed.msg: "*Deny*" AND vendorParsed.msg: "*access-group*"` | High | %ASA-4-106023: Deny tcp src... |

### Example 2: Structured Key=Value Log

**Input:**
```
/v1-query-gen

Product: Meraki MX Security Appliance
Event Type: security_event
Description: Malicious file blocked
Sample: url=http://malware.com/bad.exe disposition=malicious action=block sha256=abc123
Severity: Critical
```

**Output:**

| Product | Event Type | Description | Custom Filter Query | Severity | Sample Syslog Message |
|---------|------------|-------------|---------------------|----------|----------------------|
| Meraki MX Security Appliance | security_event | Malicious file blocked | `vendorParsed.disposition: malicious AND vendorParsed.action: block` | Critical | url=http://malware.com/bad.exe... |

### Example 3: Multiple Events at Once

**Input:**
```
/v1-query-gen

Please generate queries for these events:

Product: Palo Alto Firewall

Event: threat-detected
Description: Threat signature matched
Sample: type=THREAT subtype=virus action=block severity=critical
Severity: Critical

Event: traffic-denied
Description: Traffic denied by policy
Sample: type=TRAFFIC action=deny rule="Block-Malware" src=1.2.3.4
Severity: High
```

**Output:**

| Product | Event Type | Description | Custom Filter Query | Severity | Sample Syslog Message |
|---------|------------|-------------|---------------------|----------|----------------------|
| Palo Alto Firewall | threat-detected | Threat signature matched | `vendorParsed.type: THREAT AND vendorParsed.subtype: virus` | Critical | type=THREAT subtype=virus... |
| Palo Alto Firewall | traffic-denied | Traffic denied by policy | `vendorParsed.type: TRAFFIC AND vendorParsed.action: deny` | High | type=TRAFFIC action=deny... |

### Example 4: Using Vendor Documentation

```
/v1-query-gen

Here's documentation from my vendor. Please generate queries for all events:

[Paste your vendor's syslog documentation here]
```

Claude will parse the entire documentation and generate queries for each event type found.

## Expected Output

Every query is returned in a consistent table format:

| Column | Description |
|--------|-------------|
| Product | The vendor/product name |
| Event Type | The event identifier from the log |
| Description | Human-readable description |
| Custom Filter Query | The Vision One query |
| Severity | Critical, High, Medium, Low, or Informational |
| Sample Syslog Message | The original log sample |

### Query Types

**Direct Field Match** (for key=value logs):
```
vendorParsed.type: event_name
```

**Multiple Field Match**:
```
vendorParsed.action: block AND vendorParsed.direction: inbound
```

**Message Content Match** (for free-form text):
```
vendorParsed.msg: "*keyword*" AND vendorParsed.msg: "*another keyword*"
```

## Using Generated Queries in Vision One

1. Copy the query from the "Custom Filter Query" column
2. Navigate to: **Detection Model Management > Custom Filter Settings > Add Custom Filter**
3. Fill in the form:
   - Filter name: Use the Description
   - Event type: Select `THIRD_PARTY_LOG`
   - Query: Paste the Custom Filter Query
   - Severity: Select from the table
4. Click **Validate Query** to confirm
5. Click **Save** to create the detection rule

## Troubleshooting

**"Skill not found"**
- Check the skill is in `~/.claude/skills/v1-query-gen/`
- Ensure `SKILL.md` exists in that folder
- Restart Claude Code

**"Query invalid" in Vision One**
- Query needs at least one defined value (not just wildcards)
- Test queries in Vision One's Search App first
- Check `vendorParsed` field structure for your specific logs

**"No key=value pairs found"**
- Claude will use `vendorParsed.msg` for free-form logs
- This is normal for logs without structured fields

## Tips for Best Results

1. Include sample logs - Real samples give better results than descriptions alone
2. Specify severity - If you don't specify, Claude will estimate
3. Batch multiple events - Process entire vendor documentation at once
4. Validate before deploying - Always test in Vision One first
5. Keep samples handy - Save your syslog messages for reference

## Sources

### Trend Micro Documentation

- [Trend Vision One Search Syntax](https://docs.trendmicro.com/en-us/documentation/article/trend-vision-one-search-syntax)
- [Trend Vision One Search App](https://docs.trendmicro.com/en-us/documentation/article/trend-vision-one-search-app)
- [Third-Party Log Collection](https://docs.trendmicro.com/en-us/documentation/article/trend-vision-one-third-party-log-collection)
- [Custom Filter Settings](https://docs.trendmicro.com/en-us/documentation/article/trend-vision-one-custom-filter-settings)
- [Detection Model Management](https://docs.trendmicro.com/en-us/documentation/article/trend-vision-one-detection-model-management)

### AI Assistant

- **Model:** Claude Opus 4.6 (claude-4.5-opus)
- **Platform:** Claude Code CLI
- **Provider:** Anthropic

All queries were manually validated in a live Trend Vision One environment.

## License

MIT License - Copyright (c) 2026 Scarlett Menendez

See [LICENSE](LICENSE) for full details.

## Author

**Scarlett Menendez**

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-02-27 | Initial release |
