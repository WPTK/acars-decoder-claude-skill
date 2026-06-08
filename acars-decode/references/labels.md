# ACARS Label Reference (ARINC 618/620)

Standards-defined labels (ARINC 620 spec). For user-defined labels in the `10`–`4T` range and carrier-specific patterns, see `airline-labels.md`. Source attribution: many empirical patterns documented across both files were aggregated from the [airframesio/acars-message-documentation](https://github.com/airframesio/acars-message-documentation) repository alongside the formal ARINC 620 spec.

## Table of Contents
1. [System Control Labels](#system-control)
2. [Service-Related Downlink Labels](#service-downlink)
3. [OOOI Labels and Field Parsing](#oooi)
4. [Weather Labels](#weather)
5. [ATS Labels](#ats)
6. [Peripheral Messages (H1)](#peripheral-h1)
7. [Other Labels](#other-labels)
8. [TEI Codes](#tei-codes)
9. [Position Report Decode (5R)](#position-report)
10. [Departure/Arrival Report Decode (Q1)](#departure-arrival)
11. [ETA Report Decode (Q2)](#eta-report)
12. [5Z Command Codes](#5z-commands)

---

## System Control Labels <a name="system-control"></a>

| Label | Dir | Description |
|-------|-----|-------------|
| `_d` (DEL) | Up/Dn | Frame acknowledgement (empty payload; `ack` field carries the `blk_id` being acknowledged). Originator retransmits after timeout if no `_d` received. |
| `_j` | Up | General response, polled mode |
| `54` | Up | ACARS frequency uplink / voice go-ahead |
| `54` | Dn | Voice contact request |
| `5P` | Dn | Temporary suspension (VHF switching to voice) |
| `5V` | Dn | VDL switch advisory |
| `Q4` | Up | Voice circuit busy |
| `Q5` | Dn | Unable to deliver uplinked message |
| `Q6` | Dn | Voice to data channel changeover advisory |
| `QX` | Dn | Intercept/unable to process |
| `QV` | Dn | Autotune reject |
| `SQ` | Up | Uplink squitter (ground station broadcast with frequency/service info) |
| `:;` | Up | Data transceiver autotune — payload is frequency in Hz/kHz (e.g., `130025`, `131550`, `1317251200`). |
| `::` | Up | DSP autotune broadcast uplink |
| `:}` | Up | POA to AOA retune |
| `KB` | Up/Dn | Loopback request/response (link test) |
| `F3` | Dn | Dedicated transceiver advisory |
| `CA`–`CF` | Dn | Cockpit printer status |

---

## Service-Related Downlink Labels <a name="service-downlink"></a>

| Label | Description | Key Fields |
|-------|-------------|------------|
| `00` | Emergency / hijack report | Flight ID, emergency indication. **Note**: most real-world Label 00 captures containing `HIJACK` text are training/test transmissions, not actual events. Real hijack alerts trigger immediate ground-side action far beyond the ACARS channel. Indicators of a test: routine performance requests, runway calc questions, or other normal crew text appended to the emergency boilerplate |
| `Q0` | Link test | Always empty payload — pure connectivity verification |
| `Q1` | Departure/arrival report (IATA) | Airport codes, times, fuel, pax, ETA |
| `Q2` | ETA report | Compact format: 3 spaces + HHMM ETA + space + 2-3 digit unknown + `/DS <dest ICAO>` |
| `Q3` | Clock update advisory | Time synchronization data |
| `Q7` | Delay message | Delay reason codes |
| `QA` | OUT report with fuel (IATA) | OUT time, departure airport, fuel on board |
| `QB` | OFF report (IATA) | Takeoff time, departure airport, ETA, fuel |
| `QC` | ON report (IATA) | Landing time, destination airport |
| `QD` | IN report with fuel (IATA) | IN time, destination airport, fuel remaining |
| `QE` | OUT/fuel/destination (IATA) | Combined OUT with destination |
| `QF` | OFF/destination (IATA) | Compact: 3-char origin + HHMM UTC + 3-char dest (e.g., `EWR2210ATL`). Preferred over QB by DSPs |
| `QG` | OUT/return IN (IATA) | Training/test flights returning to origin |
| `QH` | OUT report (IATA) | Minimal OUT event |
| `QK` | Landing report (IATA) | Landing event data |
| `QL` | Arrival report (IATA) | Full arrival information |
| `QM` | Arrival information (IATA) | Extended arrival details |
| `QN` | Diversion report (IATA) | Diversion with new destination |
| `QP` | OUT report (ICAO) | 4-char ICAO airport codes |
| `QQ` | OFF report (ICAO) | 4-char ICAO codes. May include position: `KTEBKJYO1528001FE23152852N4052.1W07403.0...` (orig+dest+HHMM+`001FE`+DD+HHMMSS+pos in DDMM.M) |
| `QR` | ON report (ICAO) | 4-char ICAO codes |
| `QS` | IN report (ICAO) | 4-char ICAO codes |
| `5D` | ATIS request | Requesting ATIS |
| `5R` | Aircrew position report | Full position data (see Position Report section) |
| `5U` | Weather request | Station weather request |
| `5Y` | ETA revision / diversion | Crew-initiated schedule change |
| `5Z` | Airline designated downlink | Airline-specific content |
| `7A` | Engine data / takeoff thrust | Engine parameters at takeoff |
| `7B` | Aircrew miscellaneous | Free text from crew |
| `80`–`8~` | Aircrew-addressed downlink | Messages to specific ground parties |
| `RB` | Command/response downlink | Response to RA uplink; user-defined format |
| `SA` | Media advisory | Link status change. Format: `<digit><2-char airline><HHMMSS><suffix>`. Suffix codes: V=VHF, V/=VHF preferred, VS=VHF+SAT, S=SATCOM, S/=SAT preferred, H=HF, G=GlobalStar, 2=VDL2, X=Inmarsat, I=Iridium. Example: `0LH151351VS` = priority 0, Lufthansa, time 15:13:51, VHF+SAT |
| `S3` | LRU config report | HW/SW part numbers for avionics |
| `E1` | Internet email (host service) | mailbox/domain/body format |
| `E2` | Internet email (DSP service) | Same format, DSP-routed |

### Service-Related Uplink Labels

| Label | Description |
|-------|-------------|
| `51` | Ground GMT update |
| `52` | Ground UTC update |
| `C0`–`C9` | Cockpit/cabin printer messages |
| `RA` | Command/response uplink |
| `S1` | Network statistics report request |
| `S2` | Network performance request |
| `S3` | LRU config report request |
| `H2` | Meteorological report command |
| `H4` | Meteorological report command (extended config) |
| `RE` | Refuel uplinks (ARINC 633) |
| `RF` | CG targeting refuel uplinks (ARINC 633) |
| `DI` | De-icing uplinks (ARINC 633) |
| `DL` | Data loading uplinks (ARINC 633) |

---

## OOOI Labels and Field Parsing <a name="oooi"></a>

**OOOI** = OUT-OFF-ON-IN, the four gate/flight events:
- **OUT** (QA/QE/QH/QP): Pushback from gate. Fuel recorded.
- **OFF** (QB/QF/QQ): Wheels up (takeoff). Fuel and ETA included.
- **ON** (QC/QR): Wheels down (landing).
- **IN** (QD/QS): Arrived at gate. Fuel remaining recorded.

### Label QA — OUT/Fuel (IATA)
```
Chars 1-4:    Standard header (originator, msg seq num, block seq char)
Chars 5-10:   Flight identifier (airline 2-char + flight number 4-char)
Chars 11-13:  Departure station (3-char IATA)
Chars 14-17:  OUT time (UTC hhmm)
Chars 18-19:  Day of month
Chars 20-25:  Fuel on board (hundreds of pounds, up to 6 digits)
Chars 26:     Metric indicator (if applicable)
Chars 27+:    Free text (airline-defined)
```

### Label QB — OFF Report (IATA)
```
Chars 1-4:    Standard header
Chars 5-10:   Flight identifier
Chars 11-13:  Departure station (IATA)
Chars 14-17:  OFF time (UTC hhmm)
Chars 18-19:  Day of month
Chars 20-25:  Fuel on board (hundreds of pounds)
Chars 26-28:  Destination station (IATA)
Chars 29-32:  ETA destination (UTC hhmm)
Chars 33+:    Free text
```

### Label QC — ON Report (IATA)
```
Chars 1-4:    Standard header
Chars 5-10:   Flight identifier
Chars 11-13:  Destination station (IATA)
Chars 14-17:  ON time (UTC hhmm)
Chars 18+:    Free text
```

### Label QD — IN/Fuel (IATA)
```
Chars 1-4:    Standard header
Chars 5-10:   Flight identifier
Chars 11-13:  Destination station (IATA)
Chars 14-17:  IN time (UTC hhmm)
Chars 18-19:  Day of month
Chars 20-22:  Fuel remaining (hundreds of pounds)
Chars 23+:    Free text
```

### Label QE — OUT/Fuel/Destination (IATA)
Includes departure station, OUT time, fuel, AND destination station with ETA.

### Label QF — OFF/Destination (IATA)
Preferred over QB because it includes both departure and destination — DSPs use this for frequency management.

### ICAO variants (QP/QQ/QR/QS)
Same structure as IATA equivalents but airport codes are 4 characters instead of 3. Shift subsequent field positions by +1 character per airport code.

---

## Weather Labels <a name="weather"></a>

| Label | Dir | Description |
|-------|-----|-------------|
| `31` | Dn | Weather report (payload often starts with `WXR` format code + station IDs) |
| `H2` | Dn | Meteorological report — AMDAR (versions 1–4, increasingly complex) |
| `H3` | Dn | Icing report (5-minute interval during icing events) |
| `H4` | Up/Dn | Meteorological report configuration (AWR tables) |
| `5U` | Dn | Weather request from crew |

### Weather request text format (Label 31 or 5U)
Text typically: `WXRnn` followed by station ICAO codes separated by commas.
- `WXR04` = weather report format 4
- Station codes: `KMCO,A0000000` = requesting weather for MCO (Orlando) with parameter flags

### Meteorological Report (H2) — Version 4 structure
AMDAR (Aircraft Meteorological Data Relay) reports with:
- Version number (chars 11-12): `04` for version 4
- Type: `A`/`C` = ascent, `E` = enroute, `D`/`P` = descent (C/P = pressure-triggered sampling)
- Compressed variants: `Q`/`R` = compressed ascent, `S` = compressed enroute, `U`/`V` = compressed descent
- Each sample: latitude (ADDMMT), longitude (ADDDMMT), time, altitude (tens of feet), static air temp (P/M + nnn = tenths °C), wind direction (degrees true), wind speed (knots), roll angle flag (G=steady, B=unsteady, W=max wind steady, U=max wind unsteady), water vapor/humidity

---

## ATS (Air Traffic Services) Labels <a name="ats"></a>

| Label/MFI | Dir | SMI | Description |
|-----------|-----|-----|-------------|
| `A0` | Up | AFN | ATS Facilities Notification |
| `A1` | Up | CLX | Oceanic clearance |
| `A3` | Up | CLD | Departure clearance (D-ATIS response) |
| `A4` | Up | FSM | Flight systems message |
| `A6` | Up | RAR | Request ADS reports |
| `A7` | Up | FTU | Free text from ATC |
| `A8` | Up | DDS | Departure slot delivery |
| `A9` | Up | DAI | ATIS report |
| `AA` | Up | ATC | ATC Communications / CPDLC |
| `AB` | Up | TWI | Terminal weather (TWIP) |
| `AC` | Up | PBC | Pushback clearance |
| `AD` | Up | ETR | Expected taxi clearance |
| `AF` | Up | CPR | CPC command/response |
| `B0` | Dn | AFN | ATS Facilities Notification CONTACT (FN_CON). Format: `/<ATSU>.AFN/FMH<callsign>,.<tail>,<icao>,<HHMMSS>/FPO<position>/FCOADS,<unk>/FCOATC,<unk><CRC>` |
| `B1` | Dn | RCL | Oceanic Clearance Request. Format: `/<FIR>.OC1/RCL <id>\n<callsign>-<entry>/<HHMM> M<mach>F<FL>\n-RMK/<remarks><CRC>` |
| `B2` | Dn | CLA | Oceanic Clearance Acknowledgement (full readback with track) |
| `B3` | Dn | RCD | Departure Clearance Request. Format: `/<airport>.DC1/RCD <id>\n<callsign>-<airport>-GATE <gate>-<dest>\nATIS <code>\n-TYP/<actype>\n-RMK/<remarks>` |
| `B4` | Dn | CDA | Departure Clearance Readback (CDA/DCR) |
| `B6` | Dn | FTD | Free text to ATC from crew. Also `/<...>.ADS.<tail><hex>` ADS-C report |
| `B9` | Dn | — | Position broadcast reply. Format: `/<airport>.TI2/<id><airport><CRC>` |
| `BA` | Dn | ATC | ATC Communications downlink / CPDLC. Format: `/<station>.<MTI>.<tail><hex>`. MTIs: DR1, AT1, CC1. Stations: USADCXA (US ATN), NYCODYA (NYC oceanic) |
| `BB` | Dn | TWI | Terminal weather request |
| `BC` | Dn | PBC | Pushback clearance request |
| `BD` | Dn | ETR | Expected taxi clearance request |
| `BF` | Dn | CPR | CPC downlink |
| `HX` | Dn | — | Undelivered Uplink Report. `RA FMT LOCATION <pos>` or `RA FMT 43 <airport> <code>` or `RA IV3 <HOWGOZIT>` |
| `MA` | Dn | — | A350 MIAM encoded payload (binary-in-ASCII, starts `T.0`/`T12`/`T32`, almost always multi-part, requires reassembly) |

For ATS messages sent to peripherals, Label becomes H1 with appropriate sublabel, and the original ATS label appears as the MFI in the supplementary address field.

---

## Peripheral Messages — Label H1 <a name="peripheral-h1"></a>

Label H1 is the catch-all for messages to/from avionics subsystems via the ACARS Management Unit (MU). The **sublabel** (2 chars after `#`) identifies the target peripheral.

### Common sublabels (with header)
| Sublabel | SMI | Peripheral |
|----------|-----|------------|
| `CF` | CFD | CMC / Central Fault Display |
| `DF` | DFD | DFDR / DFDAU (Digital Flight Data) |
| `MD` | FCL | FMC selected (Flight Management Computer) |
| `T1`–`T4` | TX1–TX4 | Cabin terminal |
| `EF` | EX1 | Electronic Flight Bag |

### Common sublabels (no header)
| Sublabel | SMI | Peripheral |
|----------|-----|------------|
| `CF` | CFX | CMC (no header variant) |
| `DF` | DFX | DFDR/DFDAU (no header variant) |
| `M1` | FML | FMC left |
| `M2` | FMR | FMC right |
| `M3` | FM3 | FMC #3 |
| `SD` | SDL/SDR/SDD | Satellite Data Unit |
| `HD` | HDL/HDR/HDD | HF Data Radio |
| `EG` | ENG | Engine Indicating System |
| `EF` | EF1/EF2 | Electronic Flight Bag |

### H1 without sublabel
OAT (Optional Auxiliary Terminal) or OAX format. OAT includes a header block with originator address, date/time, SMI, TEIs, and ground station ID. OAX is headerless with just `- ` followed by free text.

### H1 message content types
H1 carries the widest variety of content. Identify by the leading character pattern:

- **`#CFB`** — Crew Flight Bag (Airbus). Carries real-time failure reports (`FLR/FR<DDMMYY><HHMM> <ATA-subATA><phase> <system> <details>,<severity>`), ATA fault summaries, and snapshot reports. Severity is `HARD` or `INTERMITTENT`. Flight phase codes (the 2-digit number after the ATA chapter): `02` = engine start +3 min up to TO power, `03` = TO power up to 80 kt, `04` = 80 kt up to liftoff, `05` = climb, `06` = cruise, `07` = descent, `08` = touchdown up to 80 kt, `09` = 80 kt up to last engine shutdown
- **`#DFB`** / **`#EFB`** — Digital Flight Bag / Electronic Flight Bag
- **`CMC fault/event reports`** (sublabel CF) — see `references/cmc-reports.md`
- **`FPN/`** — Flight Plan. Format: `FPN/<status>[:<flightnum>:RP]:<KEY>:<VAL>:...<CRC>`. Statuses: `RI` (Route Inactive), `RP` (Route Planned). Keys: DA (dep airport), AA (arr airport), CR (current route), D (dep proc), A (arr proc), AP (approach), R (dep runway), F (first waypoint). Positions in millidegrees `N01234W123456` (divide by 1000)
- **`POS`** — ACMS position reports. Multiple variants. Alaska A320 fleet: `POSN<DDMMM>W<DDDMMM>,<wpt>,<HHMMSS>,<FL>,<next wpt>,<ETA>,<future wpt>,<OAT M/Pxx>,<fuel>,<GS>K,<TAS>K,<CRC>`. United variant has fewer fields. Southwest M1 variant has airport and route appended
- **`PRG/DT`** — Progress report (Airbus A330). Format: `PRG/DT<dest ICAO>,<arr runway>,<fuel LBS>,<unk>,<fuel landing KG>/FN<callsign>/TS<HHMMSS>,<DDMMYY><CRC>`
- **ACMS engine telemetry** (sublabel DF or no sublabel) — see `references/acms-telemetry.md`. Identified by text starting with aircraft type (`A321,`, `B737,`), format prefix (`ABS`, `B43A`, `A38/`), or containing `/REP`, `/CC`, `/CE` sections
- **Resource requests** — `RESREQ/AK` format, often paired with POS in multi-part messages
- **Weather responses** — `RESPWI/AC` format
- **Flight data events** — `FDE` prefix with timestamped parameter snapshots
- **CPDLC wrapped for peripheral delivery** — contains MFI (AA, A1, etc.) in supplementary address; payload may be hex-encoded
- **Cabin system messages** (sublabel T1-T4)
- **EFB data** (sublabel EF)
- **Qantas A330 ACMS** — `R<NN>/A33<NNN>,...` then `C1,.<tail>,<DDMMMYY>,<HH.MM.SS>,<orig>,<dest>,<callsign>,...` followed by C2-C6, N1-N2, S1-S2, T1-T2, V1-V2 rows of engine/system samples

---

## Other Labels <a name="other-labels"></a>

### Observed but not in ARINC 620 standard
For full carrier-specific patterns, preambles, and field parsing of labels in the `10`–`4T` range, see `references/airline-labels.md`.

| Label | Description | Notes |
|-------|-------------|-------|
| `10` | Multi-format | Southwest LDR01 landing data, SkyWest CSV, position blocks, MET01 requests, ACKMSG |
| `11` | In-range arrival report | `OS <ICAO> /QSL<HHMM>` (US carriers), POS-CAS block, or `<id>/INRANG/DS .../ETA .../FOB ...` |
| `12` | Position report | Spirit `POSN`, WestJet `N <lat>,W <lon>,<alt>,...` |
| `13` | Arrival gate request | `KPDX,KLAS,2170` or `ARR GATE REQ /AIR ID .../DEST .../ETA ...` |
| `14` | Three variants | BizJets `(2...(Z`, Frontier CSV with free text, United `/14 OFF EVENT` |
| `15` | Departure delay | First reports (FST01), BizJet positions, multi-line delay reports |
| `16` | Boeing position | Multiple format variants |
| `17` | METAR request | SkyWest `<flags>,<time>,<code>,METAR,<airport>`, Spirit concat format |
| `18` | POX / TEI flight event | Position with state vector |
| `19` | IN/post-flight fuel | TEI format or `/19 IN FUEL FOB RPT` (United) |
| `1L` | Multi-carrier | Lufthansa, Austrian, Condor, EgyptAir, Sunwing, Frontier with obfuscated counters + plain payloads |
| `1M` | ETA report | `/<flt>/ETA01/<date>/<orig>/<dest>/...` |
| `1R` | Initial report / Gate return | `/<flt>/INT01/<date>/...` or `/1R GATE RTN` (United) |
| `20` | POS report | `POSN<lat><lon>,...` extended fields, or `RST23A` reset event |
| `22` | Position+state | Frontier-style CSV with DDMMSS position |
| `26` | ETA report | Line-by-line: FUEL/TEMP/WDIR/WSPD/LATN/LONE/ETA/TUR/ALT (Aeroflot) |
| `27` | POS report with weather | Same line-by-line structure as Label 26, `POS01` prefix |
| `28` | State CSV | Republic Airways flag block |
| `30` | ETA (Alaska) | `/EA<HHMM>/DS<dest>/SK<code>` |
| `33` | Landing weight report | Multi-line ASCII-art with FLT NO, LDW, TIME, WIND, OAT, QNH |
| `37` | Proprietary/encrypted | Southwest (obfuscated), Republic FTX01 free text, Delta, Spirit, Endeavor |
| `39` | Weather request | `WXR01<flt>/<date><time><orig><dest>` listing stations |
| `3A` | Weather request with type | `WXR01.../TYP M/STA <icao>` |
| `3F` | UPS ETA downlink | `<DDHHMM> <DD><orig> <dest> <ETA HHMM>N` |
| `3P` | Position with performance | Full position+waypoint+ETA+OAT+fuel+CRC |
| `3R` | TOC/TOD | `TOC<date>,<flt>,.<tail>,<orig>,<dest>,...` |
| `3T` | METAR/TAF text | Multi-line standard METAR/TAF format |
| `4A` | Door events / various | `DOOR/<name> <OPEN\|CLSD> <HHMM>` or `/01-C`/`/01-D` (Cessna takeoff) |
| `4J` | Position+weather report | `4J<seq> POSWX <data>/<time> <orig>/<dest> .<tail>` with POS/OVR/ALT/TFW/TAS/SAT/WND/TRB/SKY TEIs |
| `4L` | Engine run events | `<HHMM><label> <orig> <dest>\n/FN <flt>\n/ENG 1 RUN <hhmmss>/ENG 2 RUN <hhmmss>` |
| `4N` | Position w/ airport pair | Header `<DD><MM>4N <orig> <dest><digit>` + body with DDMM.M position |
| `4T` | AGFSR (Air Canada) | Full pipe/slash-delimited position/ETA with phase, OAT, wind, GS, OUT time |
| `BA` | ATC Communications downlink | CPDLC downlink (hex-encoded payload) |
| `3]` | Cabin chimes | `CHIMES,<flt>,<orig>,<dest>,<date>,<time>,...` |

### Vendor-defined uplinks (V-series)
| Labels | Vendor |
|--------|--------|
| `VA`, `VB`, `VC` | Rockwell Collins |
| `VD`, `VE`, `VF` | Honeywell |
| `VG`, `VH`, `VI` | Teledyne |
| `VJ`, `VK`, `VL` | Airbus |
| `VM`, `VN`, `VO` | Universal |
| `VP`, `VQ`, `VR` | Boeing |
| `VS`–`VZ`, `V0`–`V9` | Other / any vendor |

### Other categories
| Labels | Dir | Description |
|--------|-----|-------------|
| `X1`–`X9` | Up | DSP-defined messages |
| `10`–`4~` | Up | User-defined messages |
| `RE` | Up/Dn | Refuel — administrative and general purpose (ARINC 633) |
| `RF` | Up/Dn | CG targeting refuel (ARINC 633) |
| `DI` | Up/Dn | De-icing (ARINC 633) |
| `DL` | Up/Dn | Data loading (ARINC 633) |
| `LT` | Up/Dn | Technical (cockpit) e-logbook |
| `LS` | Dn | Technical logbook request for data |
| `LC` | Up/Dn | Cabin logbook |
| `LB` | Dn | Cabin logbook request for data |
| `P7` | Up | AMS-protected ATS message |
| `P8` | Up | AMS-protected AMS-specific message |
| `P9` | Up | AMS-protected AOC message |

---

## TEI (Text Element Identifier) Codes <a name="tei-codes"></a>

TEIs are 2-character codes identifying structured fields in ACARS messages. Format: `TEI value` (separated by space). Multiple TEIs on one line separated by `/` with no spaces.

### Identification
| TEI | Description | Format |
|-----|-------------|--------|
| `AN` | Aircraft number (registration) | Up to 7 alphanumeric chars |
| `FI` | Flight identification | Up to 7 alphanumeric chars |
| `BC` | Billing code (operating on behalf of another carrier) | 3-4 alphanumeric chars |

### Airports and routing
| TEI | Description | Format |
|-----|-------------|--------|
| `DA` | Departure aerodrome | 3-char IATA or 4-char ICAO |
| `DS` | Destination station | 3/4-char + optional UTC hhmm |
| `AD` | Arrival aerodrome | 3-char IATA or 4-char ICAO |
| `GL` | Geographic locator (aircraft location) | 3/4-char airport/city code |
| `AP` | Aircraft at airport | 3/4-char airport code |
| `SA` | Alternate aerodrome(s) | 3-char codes separated by spaces |
| `RT` | Route information | Variable alphanumeric |
| `NP` | Next report point | Variable alphanumeric |
| `SP` | Significant point | Variable alphanumeric |
| `DV` | Diversion identification | Variable alphanumeric |
| `RD` | Departure runway | Variable alphanumeric |
| `AR` | Arrival runway | Variable alphanumeric |

### Times
| TEI | Description | Format |
|-----|-------------|--------|
| `OT` | OUT time | UTC hhmm |
| `OF` | OFF time | UTC hhmm |
| `ON` | ON time | UTC hhmm |
| `IN` | IN time | UTC hhmm |
| `EO` | Estimated time over | Location + UTC hhmm |
| `ED` | Estimated departure | Aerodrome + UTC hhmm |
| `AC` | Estimated approach clearance | UTC hhmm |
| `FC` | Estimated further clearance | UTC hhmm |
| `RI` | Return IN time | UTC hhmm |
| `RO` | Return ON time | UTC hhmm |

### Position and flight data
| TEI | Description | Format |
|-----|-------------|--------|
| `OV` | Present position | Location, UTC hhmm, altitude |
| `AL` | Altitude/flight level | `Axxx` = alt in 100ft, `Fxxx` = FL, `Mxxxx` = meters. Prefix C/D/L = climbing/descending/leaving |
| `CL` | Cruising level | Same format as AL |
| `CZ` | Cruising speed | `KTxxxx` = knots, `Mx.xx` = Mach |
| `HD` | Heading | 3 digits, degrees true |

### Weather and environment
| TEI | Description | Format |
|-----|-------------|--------|
| `TA` | Static air temperature | `MSxx` = negative °C, `PSxx` = positive °C |
| `TM` | Surface air temperature | Same as TA |
| `WV` | Wind | 3 digits direction (true) + 3 digits speed (knots) |
| `WX` | Weather observation | Wind info + optional position |
| `WI` | Weather info (no assigned TEI) | Variable alphanumeric |
| `SK` | Sky conditions | Variable alphanumeric |
| `DP` | Dew point | 2 digits °C (downlink only) |
| `QN` | Altimeter setting | `xx.xx` inches (suffix M = millibars) |
| `IC` | Aircraft icing | Variable alphanumeric |
| `TB` | Turbulence | Variable alphanumeric |
| `VR` | Runway visual range | Up to 3 digits, 30-60m increments |

### Fuel and weight
| TEI | Description | Format |
|-----|-------------|--------|
| `FB` | Fuel on board | Up to 6 digits, hundreds of pounds |
| `BF` | Boarded fuel | Up to 6 digits, hundreds of pounds |
| `FD` | Fuel over destination | Up to 6 digits, hundreds of pounds |
| `EN` | Fuel endurance | hhmm |
| `ZW` | Zero fuel weight | Variable numeric |

### Operations
| TEI | Description | Format |
|-----|-------------|--------|
| `PB` | Persons on board | Variable alphanumeric |
| `LA` | Landing officer ID | 1=Captain, 2=FO, 3-8=combinations |
| `LR` | Landing category | 1 digit |
| `NL` | Number of landings | Digits + F(full stop)/T(touch-and-go) |
| `AU` | APU status | Variable alphanumeric |
| `MN` | Maintenance | Variable alphanumeric |
| `CP` | Cargo/payload info | Variable alphanumeric |
| `LP` | Log page | 10 alphanumeric chars |

### Message handling
| TEI | Description | Format |
|-----|-------------|--------|
| `MA` | Message assurance | `nnnA` where nnn=seq 000-999, a=function (A=delivery, I=delivery+link ack, L=link ack, S=receipt, X=unsupported, F=untransmittable) |
| `DT` | Communication service info | DSP processing: `DSP site ddhhmm xxxx` |
| `TP` | Transmission path | VHF, SAT, IRD, HFD, VDL |
| `SL` | SELCAL code | 4 alpha chars |
| `SI` | Special comm addressing | Variable alphanumeric |
| `RM` | Remarks | Variable alphanumeric |
| `OS` | Other supplementary info | Variable alphanumeric |

### Free text indicator
`- ` (dash space) at start of a line marks the beginning of free text after structured TEI data.

---

## Position Report Decode — Label 5R <a name="position-report"></a>

```
Chars 1-4:    Standard header (originator, msg seq, block seq)
Chars 5-10:   Flight identifier
Chars 11-15:  Latitude (degrees + minutes + N/S, e.g., 3045N = 30°45'N)
Chars 16-21:  Longitude (degrees + minutes + E/W, e.g., 08130W = 081°30'W)
Chars 22-25:  Time over position (UTC hhmm)
Chars 26-29:  Altitude (Fxxx = flight level, Axxx = altitude x100ft)
Chars 30-35:  Next position (waypoint/fix name, up to 6 chars)
Chars 36-39:  ETA next position (UTC hhmm)
Chars 40-45:  Following position (waypoint/fix)
Chars 46-48:  Wind direction (degrees true)
Chars 49-51:  Wind speed (knots)
Chars 52-55:  Static air temperature (PSxx/MSxx)
Chars 56+:    Turbulence and free text
```

**Real-world variant**: many 5R captures (and equivalent H1 POS messages) don't follow the strict char-position layout above. They use a more flexible comma-separated form:
```
POSN38415W077203,,143722,430,GVE,144630,,N64,28348,39,354
```
Fields: `POS<lat dir><DDMMM><lon dir><DDDMMM>`, blank, UTC HHMMSS, FL or alt, current/next waypoint, ETA next, blank, OAT (`N` or `M` prefix = negative °C; `P` prefix = positive), fuel quantity, groundspeed, true airspeed. This same skeleton is used by H1 POS preambles from Alaska, United, Southwest (M1 sublabel), and others — when you see it, treat it as a position report regardless of which label carries it.

---

## Departure/Arrival Report — Label Q1 <a name="departure-arrival"></a>

Dual-purpose label used for both departure (after OFF event) and arrival (after IN event).
```
Chars 1-4:    Standard header
Chars 5-10:   Flight identifier
Chars 11-13:  Departure station (IATA)
Chars 14-17:  Departure day/time or arrival time (UTC hhmm)
Chars 18-20:  Destination station (IATA)
Chars 21-24:  ETA (UTC hhmm) or arrival time
Chars 25+:    Fuel, pax, additional data per airline
```

---

## ETA Report — Label Q2 <a name="eta-report"></a>

```
Chars 1-4:    Standard header
Chars 5-10:   Flight identifier
Chars 11-13:  Destination station (IATA)
Chars 14-17:  ETA (UTC hhmm)
Chars 18-21:  Fuel over destination (hundreds of pounds)
Chars 22+:    Free text
```

Also seen in compact form (no header): `   2002  99/DS KJFK` = 3 spaces + HHMM ETA + space + 2-3 digit unknown + `/DS <dest ICAO>`.

---

## 5Z Command Codes <a name="5z-commands"></a>

Label `5Z` (airline-designated downlink) uses two main variants:

### Variant 1: `OS <airport> /<command>[<data>]` (American/US Airways)

`OS KAUS /FTM` (free text message), `OS KPHX /CLR` (clearance request), `OS KSFO /IR KSFO0312` (in range KSFO @ 03:12 UTC), `OS KAUS /ALT00001521` (altimeter+ETA).

### Variant 2: `/<command code>[ <data>]` (United Airlines)

Command codes:

| Code | Meaning |
|------|---------|
| `B1` | Request Weight and Balance |
| `B3` | Request Departure Clearance |
| `CD` | Weight and Balance |
| `CG` | Request Pre-departure clearance (PDC) |
| `CM` | Crew Scheduling |
| `C3` | Off Message |
| `C4` | Flight Dispatch |
| `C5` | Maintenance Message |
| `C6` | Customer Service |
| `10` | PIREP |
| `C11` | International PIREP |
| `DS` | Late Message |
| `D3` | Holding Pattern Message |
| `D6` | From-To + Date |
| `D7` | From-To + Alternate + Time |
| `EO` | In Range |
| `PW` | Position Weather |
| `RL` | Request Release |
| `R3` | Request HOWGOZIT Message |
| `W1` | METAR |
| `ET` | Expected Time |

Format: `/<code> <description>          / <orig> <dest> <DD> <HHMMSS>[/<extra>]`. Example: `/R3 HOWGOZIT REQ   / KSFO KCMH 05 071630 8152 05 KSFO`.
