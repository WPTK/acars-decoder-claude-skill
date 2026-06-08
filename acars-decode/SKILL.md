---
name: acars-decode
description: >
  Use this skill whenever the user wants to decode, interpret, analyze, or understand an ACARS message,
  VDL Mode 2 (VDLM2) message, CPDLC message, or any aviation datalink message. Triggers include:
  any mention of 'decode this ACARS', 'what does this message mean', 'interpret this VDL2', 'ACARS message',
  or when the user pastes raw ACARS/VDL2 text, or any structured aviation
  datalink output and wants to understand what it says. Also trigger when the user shares output from any ACARS/VDL2 decoder tool.
  This skill covers ALL ACARS message types. It handles OOOI (out/off/on/in),
  position reports, weather requests, free text, ATS/CPDLC, departure/arrival, ETA, meteorological,
  engine data, media advisory, and any other ARINC 620/618 label. It also handles VDL Mode 2 framing
  metadata from decoder tools. Do NOT use for aircraft ownership research (use aircraft-research skill)
  or ADS-B position data without an ACARS/datalink message attached.
---

# ACARS & VDL Mode 2 Message Decoder

You are decoding aviation datalink messages. The user will paste a raw ACARS or VDL Mode 2 message and your job is to produce a clear, structured breakdown of every field — what it means, what the aircraft is doing, and any operationally significant context.

## Reference Files

This skill uses reference files for detailed decode tables. **Read the relevant reference file(s) before decoding:**

| Reference | When to read |
|-----------|-------------|
| `references/labels.md` | **Always** — contains the complete ARINC 620 label table, TEI codes, 5Z command codes, and field formats for standards-defined message types |
| `references/airline-labels.md` | When the label is in the `10`–`4T` range (user-defined). Contains carrier-specific patterns, preambles, and field parsing for non-standard but widely-observed labels |
| `references/acms-telemetry.md` | When the message is an H1 with ACMS/engine telemetry data (dense comma-separated values, REP headers, /C section prefixes, ABS prefix, ITT triggers) |
| `references/cmc-reports.md` | When the message is a CMC/maintenance fault report (Label H1 sublabel CF, or standalone CMC format with RPT/Line A/Line B/Line R structure) |
| `references/cpdlc.md` | When the message contains CPDLC content (Label AA, Label BA hex-encoded `/<station>.<MTI>.<tail><hex>`, or decoded CPDLC block from the decoder tool) |

Empirical patterns documented across these references draw heavily from the [airframesio/acars-message-documentation](https://github.com/airframesio/acars-message-documentation) research repository alongside the formal ARINC 620 spec.

## Understanding the two layers

**ACARS** is the application-layer messaging protocol (ARINC 618 air-ground format, ARINC 620 ground system/message content). Messages contain a label, optional sublabel, and text payload.

**VDL Mode 2** is one transport layer — a digital data link on VHF frequencies (118.000–136.975 MHz) using D8PSK modulation at 31.5 kbit/s per ETSI EN 301 841-1. When a decoder tool captures a VDL2 frame, it wraps the ACARS payload in VDL2 framing metadata.

**Key distinction:** VDL2 is the pipe; ACARS is the message inside. Decode both layers when present. Users colloquially call all of it "ACARS" — decode everything they give you.

Other ACARS transports include: plain VHF (POA — 2400 baud AM on 131.550 MHz and other frequencies), SATCOM (Inmarsat, Iridium), and HF Data Link (HFDL). The user's receiver is an adsb.im stack at KJAX capturing both plain ACARS (130.025, 131.55, 131.725 MHz) and VDL2 (136.650, 136.800, 136.975 MHz). Messages from airframes.io federated stations may also appear with different station names and frequencies.

## Decode workflow

### Step 1: Identify the transport layer
If VDL2 framing metadata is present, decode it first:
- **Frequency** — which VDL2 channel (common: 136.650, 136.725, 136.775, 136.800, 136.825, 136.875, 136.925, 136.975 MHz)
- **Ground station / To Address Station ID** — which ARINC ground station handled it
- **Signal level** — reception quality (closer to 0 dB = stronger; below -20 dB = weak)
- **ICAO hex** — 24-bit aircraft address; cross-reference to tail if not shown
- **On Ground** — operational context for phase of flight
- **Block ID** — multi-block tracking; `0` = standalone single-block message. For multi-block, the ID cycles through `1`–`9` then `A`–`Z` (36 values total). Same value across the blocks of a single multi-block transmission
- **Message Number** — ends in `A` for first block, `B`/`C`/etc. for continuation
- **ACK** — `!` = NAK (requesting retransmission); other values acknowledge prior blocks
- **Mode** — `2` = VDL Mode 2

### Step 2: Identify the ACARS message type via Label
The **Label** is the primary type identifier. Read `references/labels.md` for the complete standards-defined table. If a sublabel or MFI (Message Function Identifier) is present, it further narrows the type.

For labels in the `10`–`4T` range (user-defined per ARINC 620), the **preamble** (first 3-8 characters of the payload) is usually the strongest signal for what report type and which airline. Read `references/airline-labels.md` and match the preamble. Common preambles: `POS`/`POSN` (position), `OFF`/`OFFRP` (off event), `OS ` (origin-prefixed message), `INRANG` (in-range), `WXR01` (weather), `ETA01` (ETA), `FUELRP` (fuel), `LDR01` (Southwest landing), `AGFSR` (Air Canada), `DAT` (Jetstar), `DOOR/` (door event), `CHIMES` (cabin call), `BRAKE/` (brake event), `FTX01` (free text).

### Step 3: Decode the text payload
Apply format rules for the identified label from the reference files. For fixed-format messages (OOOI, position reports, ETA), parse character-by-character per ARINC 620 field definitions. For free-text or peripheral messages (H1), identify the sublabel and embedded structure.

### Step 4: Provide operational context
After decoding fields, explain what the message means operationally: phase of flight, what the aircraft is reporting or requesting, whether anything is unusual or noteworthy.

## Input formats

### 1. adsb.im / EROS local station output
The user's primary source. Two sub-formats:

**VDL2 capture** — header line `VDL-M2EROS-KJAX-VDL2` (or `VDL from station EROS-KJAX-VDL2`), then labeled metadata fields:
```
Tail:N565JB / Flight: United Airlines 0559 / ICAO:A73A47
Message Label: H1 Message To/From Terminal
Frequency: 136.800 MHz / Signal Level: -17.1 dB
Mode: 2 / Block ID: 5 / Message Number: F54A / ACK: ! / On Ground: No
```
Then `Text:` followed by the ACARS payload. Multi-part messages show `Message Parts: F54A F55A` and may include both `Decoded Text` and `Non-Decoded Text` sections.

**Plain ACARS capture** — header line `ACARSEROS-KJAX-ACARS`, otherwise similar metadata structure but on POA frequencies (130.025, 131.55, 131.725 MHz).

### 2. airframes.io federated station output
Same format but station names vary: `PhillyRox-VDLM`, `BS-KAUS-VDL2`, `CW-KTPA-ACARS`, `KC3ZYT-KIDI-IMSL` (Inmarsat), etc. May use shortened metadata format:
```
Tail N124DU ICAO A062EC Flight DL2958
F: 136.65 M: 2 L: H1 B: 1 M#: 801
```
Where F=frequency, M=mode, L=label, B=block ID, M#=message number.

### 3. Raw ACARS text (from ARINC ground systems)
Starts with routing headers like `/BLVBOCR.` followed by structured lines. No VDL2 framing.

### 4. Decoded CPDLC blocks
Tool-decoded CPDLC with Msg Type, Gs Addr, Air Addr, message elements. See `references/cpdlc.md`.

### 5. CMC/maintenance reports
Fault or event reports with RPT/Line structure. See `references/cmc-reports.md`.

## Airline-specific notes

- **Delta / Northwest legacy**: Delta ACARS systems transmit using legacy Northwest IATA code `NW` in the raw callsign field of the ACARS message (e.g., `NW2723` for DL2723, `NW2351` for DL2351). Confirmed across A321, A319, and 737 fleet. Post-merger system artifact, still active as of April 2026. Parser caveat: airframesio.io and some federated stations auto-translate the raw `NW` to the displayed IATA code (`DL`) in their parsed output; adsb.im local captures typically show the raw `NW` field. The flight is the same Delta operation either way — note which form your source uses.
- **Delta ACMS**: Delta A321 fleet uses REP239 report format with inline position/flight data. Delta 737 fleet uses `/REP019` format with `/CC`, `/C0`, `/C1`, `/CE`, `/EC`, `/EE` section prefixes. Both confirmed in real captures. See `references/acms-telemetry.md`.
- **American Airlines ACMS**: A321 fleet uses `A38/A321` prefix with `/C1` through `/C5` climb/cruise profile sections containing lat, lon, altitude, speed, time-to-go data. Different structure from Delta.
- **United Airlines ACMS**: 737 fleet uses `ABS026AA` prefix for ACMS climb profiles with position/altitude/temp/wind samples tagged with flight phase (CL=climb, ER=enroute, DE=descent). Also uses `B43A` prefix format for landing/approach reports with `OFFOFFAUT` autoland flags.
- **United event reports**: Label 14 `/14 OFF EVENT` format and Label 1E `/1E BRAKE AFTER ENG` are diagnostic event markers; the origin/destination shown can be stale from a previous leg until the crew updates FMS.
- **Southwest**: High-volume. Label 37 messages are encrypted/compressed proprietary content — do not attempt to decode the payload, just note it. Label 10 uses `LDR01` landing-data CSV. POS reports use extended format with full route decode (`:DA:origin:AA:dest..waypoints:D:SID:F:route:A:STAR`). 737-MAX fleet ACMS uses `++<6-digit seq>,<tail>,B7378MAX,<YYMMDD>,<flight>,<orig>,<dest>,<leg>,<ACMS serial>` H1 sublabel-DF format with position samples below — see `references/acms-telemetry.md`. All-737 fleet.
- **Atlas Air (GTI)**: Wet-lease operator. GTI = Atlas house flights, 5Y/GTI = Atlas cargo, GSS = charter.
- **FedEx (FDX)**: ACARS on 131.725 MHz. Verbose reports from 767/777/MD-11 fleet.
- **UPS**: Heavy user of labels `3B`, `3C`, `3F`, `3G`, `3I`, `3J`, `3K`, `3M`, `3N`, `3S`, `3U`, `3W` for operational reports (ETA, mechanical issues, manifest, crew dispatch). Format pattern: `<DDHHMM> <DD><orig IATA> <dest IATA> <HHMM><extra>`. **SDF** = Louisville Worldport (main hub), **MHR** = Sacramento Mather (West Coast sort hub), **ONT** = LA Ontario gateway.
- **Air Canada / Rouge**: Uses `AGFSR` prefix in position/ETA reports (Label 4T) with structured fields: flight/day/origin/dest/time/heading/position/altitude/fuel/temp/wind.
- **WestJet**: Label 12 position reports with simple `N lat,W lon,altitude,time,speed,.tail,fuel` format.
- **JetBlue**: A320 fleet. Combined RESREQ + POS messages in single multi-part H1 transmissions.
- **Alaska Airlines**: Label 16 `N <decimal lat>,W <decimal lon>,<alt>,<unk>, <unk>` simple positions. Label 30 `/EA<HHMM>/DS<ICAO>/SK<code>` ETA format. H1 POS variant: `POSN<lat>W<lon>,<wpt>,<HHMMSS>,<FL>,<next>,<ETA>,<future>,<OAT>,<fuel>,<GS>K,<TAS>K,<CRC>`.
- **Spirit (NK)**: Label 12 `POSN <DDMMSS>W <DDMMSS>,...` positions with FOB/ETA/origin/destination appended. Label 17 origin+dest+ETA concatenated.
- **Frontier (F9)**: Label 14 variant 2 with comma-separated fields including freetext spanning fields 10-13. Label 1L obfuscated counter variant. Labels 28/29 state CSVs.
- **Aeroflot/Russian carriers**: Labels 26 and 27 use distinctive multi-line `ETA01`/`POS01` format with FUEL/TEMP/WDIR/WSPD/LATN/LONE/ETA/TUR/ALT keyed lines.
- **Jetstar (JST/JQ)**: Label 2L with `DAT` preamble for departure data (REG, FLT, GWT, ZFW, FOB, CAP, FO, LOG, LDR, DRT lines). Label 3L `S <lat>/E<lon> /UTC <HHMM>` short positions.
- **Qantas A330**: H1 messages with `R<NN>/A33<NNN>` ACMS records — long multi-block with C1-C6, N1-N2, S1-S2, T1-T2, V1-V2 telemetry rows.
- **Airbus A350**: Label MA messages start with `T.0`, `T12`, or `T32` — these are MIAM-encoded (binary-in-ASCII), almost always multi-part. Treat as opaque encoded data unless reassembly + decryption is performed; the message bounds are `T.0&` ... `|`.
- **Air France**: Label 2N/2O takeoff reports with `AF<flight>/<DDHHMM><origICAO><destICAO>` format. Label 2U `ECD/<tail>\nLOADSHEET FINAL EDN<NN>\n<flight>/<DD>/<DDMMMYY>/<HHMM>\n<orig+dest>\nTOW <weight>` loadsheet finals.
- **BizJets (ARINCDirect)**: Label 14/15 messages prefixed `(2` are free text from FBO/dispatch to crew. Must end with `(Z` to be complete — missing `(Z` = multi-part fragment, ignore or reassemble. Common origins: NetJets fleet (QS callsigns), various Cessna/Gulfstream/Falcon owners.
- **Lufthansa Group (LH/OS/DE/EW)**: Label 1L obfuscated counter format for state reports. Condor (DE) uses readable `DE<flight>,<HHMM>,<airport>,<flag>` event format.

## Output format

Produce a structured breakdown with these sections:

**1. Transport Layer** (if VDL2 metadata present)
Brief summary: frequency, ground station, signal quality, ICAO hex.

**2. Header / Flight Info**
Tail number, flight number/callsign, airline, date/time, origin/destination if known.

**3. Message Type**
Label, sublabel (if any), plain-English description.

**4. Payload Decode**
Field-by-field breakdown. For fixed-format messages, parse every field. For free text, interpret. For binary/hex-encoded data, decode what's possible and note what needs additional context.

**5. Operational Context**
What this tells us about the flight's current state.

## Important rules

- Always convert positions to decimal degrees alongside the raw format
- Identify aircraft type from the tail number when recognizable, and report it as the ICAO type designator (e.g., B738, A21N)
- For CMC reports, always flag REAL vs TEST vs FAULT status
- Weather requests starting with `WXR`: digits after WXR are format code, followed by station identifiers
- Multi-block messages (Block ID not `0`) are fragments — note this and decode what's present
- Use Fahrenheit and imperial units for the user's benefit, but show original metric values too
- Do not speculate on fault severity beyond what the message explicitly states — note what the data shows and let the user draw conclusions
