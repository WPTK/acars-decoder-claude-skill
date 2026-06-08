# ACARS & VDL Mode 2 Message Decoder

A Claude skill that decodes ACARS, VDL Mode 2, and CPDLC messages into a clear, field-by-field breakdown.

## Overview

ACARS and VDL Mode 2 messages are the short datalink traffic aircraft and ground systems trade constantly: position reports, out/off/on/in times, weather requests, free-text notes between crew and dispatch, climb and cruise engine telemetry, automated maintenance fault reports, and controller-pilot clearances (CPDLC). Most are terse and coded.

Paste one in, from any receiver or decoder, and the skill names the message type, decodes every field, converts positions to decimal degrees, and explains what the aircraft is doing or reporting.

## Installation

An Agent Skill is a plain-Markdown format: a `SKILL.md` file (decode instructions plus a short YAML header) and optional reference files. The format was introduced by Anthropic, but it is not Claude-specific. Nothing in this skill calls a model-specific feature; it is instructions and decode tables, so any agent or harness that loads `SKILL.md` skills can use it.

To install, download or clone this repository and place the `acars-decode/` folder in the skills directory your agent reads. The path depends on the tool:

```
~/.claude/skills/acars-decode/     Claude Code / Claude Desktop, personal (all projects)
.claude/skills/acars-decode/       Claude Code, project-scoped (committed to a repo)
.cursor/skills/acars-decode/       Cursor, project-scoped
```

Other tools that support the format use their own skills directory; the folder contents are identical in every case. `SKILL.md` has to sit at the top level of the `acars-decode/` folder, not nested deeper (`skills/acars-decode/SKILL.md`, not `skills/acars-decode/something/SKILL.md`). One folder too many is the usual install mistake.

Once the folder is in place, the agent loads the skill on its own when a message matches the description, with nothing to invoke manually. For the Claude implementation specifically, the [Claude Code skills documentation](https://code.claude.com/docs/en/skills) covers the full model.

## Activation

The skill triggers when the user wants to decode, interpret, or understand an ACARS / VDLM2 / CPDLC message, pastes raw datalink text, or shares output from any ACARS/VDL2 decoder tool. It covers all ARINC 620 message types: OOOI, position reports, weather requests, free text, ATS/CPDLC, departure/arrival, ETA, meteorological, engine data, media advisory, and the full user-defined label range.

It does **not** handle aircraft ownership research or bare ADS-B position data with no datalink message attached.

## Structure

```
acars-decode/
├── SKILL.md                      Decode workflow, input formats, airline notes, output spec
├── README.md                     This file
└── references/
    ├── labels.md                 Complete ARINC 620 label table, TEI codes, 5Z commands
    ├── airline-labels.md         Carrier-specific patterns for user-defined labels (10 to 4T)
    ├── acms-telemetry.md         H1 ACMS / engine telemetry parsing
    ├── cmc-reports.md            CMC / maintenance fault report structure
    └── cpdlc.md                  CPDLC content decoding
```

### When each reference is read

| Reference | Read when |
|-----------|-----------|
| `labels.md` | Always. The baseline label table and field formats for standards-defined types. |
| `airline-labels.md` | Label falls in the user-defined `10` to `4T` range. Matched by payload preamble. |
| `acms-telemetry.md` | H1 message carrying dense ACMS/engine telemetry (REP headers, `/C` sections, ABS prefix). |
| `cmc-reports.md` | CMC or maintenance fault report (H1 sublabel CF, or standalone RPT/Line structure). |
| `cpdlc.md` | CPDLC content (Label AA, hex-encoded Label BA, or a decoded CPDLC block). |

Two label-range figures appear in the skill, and they are not in conflict. ARINC 620 defines the full user-defined label range as `10`–`4~`. The documented, decode-actionable subset (labels for which `airline-labels.md` actually carries field formats) is `10`–`4T`. The skill text and this README use `10`–`4T` because that is the range it can parse; `airline-labels.md` references the wider `10`–`4~` because that is the spec's outer bound. Labels above `4T` with no documented structure are listed as opaque so the decoder does not chase a format that isn't there.

## Decode workflow

1. **Identify the transport layer.** If VDL2 framing is present, decode it first: frequency, ground station, signal level, ICAO hex, on-ground flag, block ID, message number, ACK, mode.
2. **Identify the message type via Label.** The label is the primary type identifier; a sublabel or MFI narrows it further. For user-defined labels, the payload preamble (`POS`, `OFF`, `INRANG`, `WXR01`, `ETA01`, `FUELRP`, and others) is the strongest signal for report type and carrier.
3. **Decode the payload.** Apply the format rules for the identified label. Fixed-format messages (OOOI, positions, ETA) parse character by character; free-text and peripheral H1 messages are interpreted by sublabel and embedded structure.
4. **Provide operational context.** What the message says about phase of flight, what the aircraft is reporting or requesting, and anything unusual.

## Output format

Decodes are returned in five sections:

1. **Transport Layer** (when VDL2 metadata is present): frequency, ground station, signal quality, ICAO hex.
2. **Header / Flight Info**: tail, flight/callsign, airline, date/time, origin and destination if known.
3. **Message Type**: label, sublabel, plain-English description.
4. **Payload Decode**: field by field.
5. **Operational Context**: what it tells us about the flight's current state.

## Conventions

- Positions are converted to decimal degrees alongside the raw coordinates.
- Aircraft type, when recognizable from the tail number, is reported as the ICAO type designator (B738, A21N, and so on).
- Numeric values are given in Fahrenheit and imperial units, with the original metric values shown alongside.
- A single captured block of a multi-block message is decoded as a fragment and labeled as such, rather than guessed past.
- Maintenance (CMC) reports are flagged REAL, TEST, or FAULT, so a test transmission is not read as a live fault.
- Fault severity is never inferred beyond what the message explicitly states.

## Carrier coverage

Carrier-specific report formats, preambles, and parsing notes for the user-defined label range live in `references/airline-labels.md`.

## Sources

Empirical decode patterns draw heavily from the [airframesio/acars-message-documentation](https://github.com/airframesio/acars-message-documentation) research repository, cross-referenced against real captures. The underlying standards:

- [ARINC Specification 618](https://aviation-ia.sae-itc.com/standards/arinc618-9-618-9-air-ground-character-oriented-protocol-specification), Air/Ground Character-Oriented Protocol, defines the ACARS air-ground format.
- [ARINC Specification 620](https://aviation-ia.sae-itc.com/standards/arinc620-10-620-10-datalink-ground-system-standard-interface-specification-dgss), Datalink Ground System Standard and Interface Specification, defines the ground-system message content (labels, sublabels, field formats).
- [ETSI EN 301 841-1](https://www.etsi.org/deliver/etsi_en/301800_301899/30184101/01.04.01_60/en_30184101v010401p.pdf), VHF air-ground Digital Link (VDL) Mode 2, specifies the D8PSK physical layer.

ARINC specifications are published by SAE ITC and must be purchased. The ETSI standard is a free download. Both ARINC links point to the current supplement (618-9, 620-10) at the time of writing.
