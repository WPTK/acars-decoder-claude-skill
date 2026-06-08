# ACMS (Aircraft Condition Monitoring System) Telemetry Reference

## Overview

ACMS data appears in H1 messages containing dense, structured sensor readings from the aircraft's monitoring systems. Format varies significantly by aircraft type, airline, and ACMS software configuration. This reference documents patterns observed in real captures.

ACMS messages are typically multi-block (Block ID not `0`) due to their size. Always note if you're seeing a fragment.

---

## Identifying ACMS messages

ACMS messages are H1-label with these distinguishing characteristics:
- Text starts with aircraft type identifier: `A321,` `A319,` `B737,` `B738,` etc.
- Or starts with format prefix: `ABS`, `B43A`, `AGFSR`, `A38/`
- Or starts with `++<6-digit seq>,` (Southwest 737-MAX format)
- Contains `/REP` header or dense comma-separated numeric data
- Contains section prefixes like `/C0`, `/C1`, `/CE`, `/EC`, `/EE`, `/P`, `/S`, `/A`
- Contains `ITT_#` (Inter-Turbine Temperature) trigger identifiers

---

## Southwest Airlines 737-MAX ACMS — `++` Prefix Format

Confirmed on N8713M, N8717M and other 737-8/737-8 MAX tails. Sublabel: `DF`.

```
++86501,N8713M,B7378MAX,220618,WN655 ,KOAK,KHOU,1175,SMX47-2102-0000
6
N2954.0,W09611.6,180618,14449, 00.5, 65,009,DC,00000,0,
N2953.2,W09605.2,180619,12625, 05.0,148,001,DC,00000,0,
N2952.5,W09558.9,...
```

**Header line:** `++<6-digit seq>,<tail>,<aircraft type>,<YYMMDD>,<flight>,<orig ICAO>,<dest ICAO>,<leg counter>,<ACMS unit serial>`

| Field | Example | Description |
|-------|---------|-------------|
| `++` | literal | Format marker |
| `86501` | Report sequence number | |
| `N8713M` | Tail | |
| `B7378MAX` | Aircraft type literal | `B7378MAX` = 737-8 MAX; also seen `B7378` |
| `220618` | Date | YYMMDD |
| `WN655 ` | Flight | Padded to 6 chars |
| `KOAK` | Origin | ICAO |
| `KHOU` | Destination | ICAO |
| `1175` | Leg counter | Cumulative airframe legs |
| `SMX47-2102-0000` | ACMS unit serial / software version | Format `SMX<NN>-<NNNN>-<NNNN>` |

**Body:** A single line with the sample count (e.g., `6`), then that many position-sample rows. Each row is comma-separated:

| Pos | Example | Description |
|-----|---------|-------------|
| 1 | `N2954.0` | Latitude direction + DDMM.M (29°54.0' N) |
| 2 | `W09611.6` | Longitude direction + DDDMM.M (096°11.6' W) |
| 3 | `180618` | Date+time DDhhmm (day 18 at 06:18 UTC) — wait, this is more likely DDHHMM. Cross-reference against header date |
| 4 | `14449` | Altitude (ft) |
| 5 | ` 00.5` | Pitch angle (°) |
| 6 | `65` | Heading? Or VSI tenths |
| 7 | `009` | Vertical speed × 100 ft/min |
| 8 | `DC` | Phase flag: `CL`=climb, `DC`=descent, `CR`=cruise, `TO`=takeoff, `LD`=landing |
| 9 | `00000` | Unknown (often zeros) |
| 10 | `0` | Unknown trailing flag |

Note the header `flight` field captures the leg the ACMS unit is currently logging, but the message may be transmitted on a LATER flight after the unit dumps cached data. Always check whether the header `KOAK,KHOU` route matches the live flight number/route in the VDL2 frame header; if not, this is a delayed dump from a prior leg.

---

## Delta Air Lines ACMS Formats

### Delta A321 — REP239 (Enroute/Position Report)
Confirmed on N125DN, N120DN, N391DN.

```
A321,126066,1,1,TB000000/REP239,00,00,4/239N125DN2723040126114751779N30999W 82562369-33-59259 23AF0501 17880 253 459 000323050      KTPAKLGA
```

Field breakdown:
| Field | Example | Description |
|-------|---------|-------------|
| Aircraft type | `A321` | Airbus A321 |
| Report serial | `126066` | Sequential report number |
| `1,1` | Page 1 of 1 | |
| `TB000000` | Turbulence block (zeros = none) | |
| `/REP239` | Report type 239 | Position/status report |
| `00,00,4` | Report parameters | |
| `/239` | Report type repeated | |
| Tail | `N125DN` | Registration |
| Flight | `2723` | Flight number |
| Date | `040126` | DDMMYY (April 1, 2026) |
| Time | `114751` | UTC hhmmss |
| Heading | `779` | Tenths of degrees? Or magnetic heading encoded |
| Position | `N30999W 82562` | Lat 30.999°N, Lon 82.562°W (degrees + thousandths) |
| Flight level | `369` | FL369 = 36,900 ft |
| Temperature | `-33` | OAT in °C |
| Unknown | `-59` | Possibly wind or deviation |
| Ground speed | `259` | Knots |
| Wind | `23AF0501` | Encoded wind data |
| Fuel/weight | `17880` | Likely fuel remaining or gross weight |
| Speed params | `253 459` | IAS and TAS (knots) |
| Deviation | `000323050` | Navigation/track parameters |
| Route | `KTPAKLGA` | Origin KTPA, destination KLGA |

### Delta 737/A319 — REP019 (Engine/Cruise Report)
Confirmed on N520DE (737), N340NB (A319).

```
A321,016696,1,1,TB000000/REP019,84,01,4/CCN520DE,APR01,115523,KMCO,KJFK,0872
/C0TWP04005030002
/C106,04727,9991,09,0010,0,0100,09,X
/CE-348,39000,241,780,1437,265,W52503
/EC800396,00114,00027,73,15,031
/EE771921
```

Section prefixes:
| Prefix | Description | Observed fields |
|--------|-------------|-----------------|
| `/CC` | Flight identification | Tail, date (MMMdd), time (hhmmss), origin, dest, flight number |
| `/C0` | Configuration | Software/config identifier string |
| `/C1` | Parameter block 1 | Numeric parameters, comma-separated |
| `/CE` | Cruise/engine snapshot | Temperature(°C tenths), altitude(ft), IAS, TAS, fuel flow, wind, wind encoding |
| `/EC` | Engine condition | N1/N2/EGT/fuel flow/oil data per engine |
| `/EE` | Extended engine | Additional engine parameters |

**`/CE` field decode (observed):**
- `-348` = static air temp: -34.8°C
- `39000` = altitude: 39,000 ft
- `241` = indicated airspeed (knots)
- `780` = true airspeed (knots Mach-derived?) or ground speed
- `1437` = fuel flow (lbs/hr per engine or total)
- `265` = heading or wind direction
- `W52503` = wind encoded (W = west, 525 = direction?, 03 = speed component?)

### Delta Engine Telemetry — ITT Trigger Reports
Confirmed on N124DU. Extremely long messages with ITT (Inter-Turbine Temperature) event triggers.

```
R641V0106,170300,, N124DU,5318410-07,736086,736222,07245,KMSP,KDCA,12:37:21,2026:04:01,2958
```

Header fields:
| Field | Example | Description |
|-------|---------|-------------|
| Report ID | `R641V0106` | Report type and version |
| Unknown | `170300` | Parameter count or config |
| Tail | `N124DU` | Registration |
| Serial/config | `5318410-07` | ACMS software config |
| Engine serials | `736086,736222` | Left and right engine serial numbers |
| Fuel/weight | `07245` | Fuel state |
| Route | `KMSP,KDCA` | Origin, destination |
| Time | `12:37:21` | UTC |
| Date | `2026:04:01` | |
| Flight | `2958` | Flight number |

Following the header, data is organized into snapshot blocks triggered by `ITT_#nnnnn(hh:mm:ss)` markers, each followed by `/P` (performance), `/S` (status), and `/A` (aerodynamic) sections containing hundreds of comma-separated engine parameters.

---

## American Airlines ACMS Format

### American A321 — A38 Climb/Cruise Profile
Confirmed on N917UY.

```
A38/A32138,1,1/C1TRP,114757,KTPA,KCLT,07,8,78404
/C2279358,-0826427,082,01418,279,0739,1
/C3281979,-0826036,159,01410,362,0542,1
/C4285186,-0825845,227,01408,403,0405,1
/C5288705,-0826163,277,01399,433,0323,1
```

Header: `A38/A321` = report type / aircraft type, then `38,1,1` = report number, page.

`/C1` = flight identification:
- `TRP` = report type (climb profile?)
- `114757` = time UTC (hhmmss)
- `KTPA,KCLT` = origin, destination
- `07,8` = parameters
- `78404` = weight or fuel

`/C2` through `/C5` = sequential climb profile samples:
| Field | Example | Description |
|-------|---------|-------------|
| Latitude | `279358` | 27.9358°N (degrees × 10000) |
| Longitude | `-0826427` | 82.6427°W (degrees × 10000, negative = west) |
| Heading/track | `082` | Degrees |
| Altitude | `01418` | Hundreds of feet? (14,180 ft) or direct feet |
| Speed | `279` | IAS knots |
| Time-to-go | `0739` | Seconds or encoded time |
| Phase flag | `1` | Status indicator |

---

## United Airlines ACMS Formats

### United 737 — ABS Climb Profile
Confirmed on N37532.

```
ABS026AA_N37532,B737N37-3260401,UA    ,KTPA,KEWR,0887,BCG2E-S200-0009
N2845.6,W08236.1,011207,24613,-26.0,250,010,CL,00000,0,
N2907.4,W08239.1,011210,28707,-36.5,247,014,CL,00000,0,
```

Header: `ABS026AA` = report type, then:
- `_N37532` = tail
- `B737N37-3` = aircraft type and config
- `260401` = date (YYMMDD)
- `UA    ` = airline
- `KTPA,KEWR` = origin, destination
- `0887` = flight number? or fuel/weight
- `BCG2E-S200-0009` = ACMS software load

Each sample line:
| Field | Example | Description |
|-------|---------|-------------|
| Latitude | `N2845.6` | 28°45.6'N = 28.760°N |
| Longitude | `W08236.1` | 082°36.1'W = 82.602°W |
| Time | `011207` | Day 01, 12:07 UTC |
| Altitude | `24613` | 24,613 feet |
| Temperature | `-26.0` | OAT in °C |
| Speed | `250` | IAS knots |
| Wind | `010` | Wind speed (knots) or component |
| Phase | `CL` | CL=climb, ER=enroute, DE=descent |
| Params | `00000,0` | Additional status/flags |

### United 737 — B43A Landing/Approach Report
Confirmed on N77544.

```
B43A
N77544  9561383KRDUKIAHER01041100
3679243260A85060A860BCG3A-L200-0002
 34009-1113270444160780 -248 -518161840
PRI11193310006 337412 -843596  -13    23166000
 33998270344660780 -235 -505
OFFOFFAUT
```

- `B43A` = report type identifier
- Second line: tail, serial(?), flight number, origin KRDU, destination KIAH, date/time
- `BCG3A-L200-0002` = ACMS software load
- Data lines contain approach/landing parameters
- `OFFOFFAUT` = auto-off event flags (autoland/autopilot disconnect status)

---

## Air Canada / Rouge — AGFSR Format

### Label 4T — Position/ETA Report
Confirmed on C-GSJB.

```
AGFSR RV1657/01/01/TPAYYZ/1219Z/290/3110.9N 8232.6W/370/      /0089/0024/M59/265029/0256/   /463/1136/1149/----/----
```

Fields (slash-delimited):
| Field | Example | Description |
|-------|---------|-------------|
| Prefix | `AGFSR` | Report type |
| Flight | `RV1657` | IATA callsign |
| Departure day | `01` | |
| Arrival day | `01` | |
| Route | `TPAYYZ` | Origin TPA, destination YYZ |
| Time | `1219Z` | UTC |
| Heading | `290` | Degrees |
| Position | `3110.9N 8232.6W` | 31°10.9'N, 82°32.6'W |
| Altitude | `370` | FL370 = 37,000 ft |
| (reserved) | | |
| Fuel | `0089` | Hundreds of pounds? |
| Fuel used | `0024` | |
| Temperature | `M59` | -59°C |
| Wind | `265029` | 265° at 029 knots |
| Ground speed | `0256` | Knots |
| (reserved) | | |
| TAS | `463` | Knots |
| ETA times | `1136/1149` | Encoded ETAs |

---

## WestJet — Label 12 Position Report

Confirmed on C-GFVE.

```
N 30.628,W 82.470,36000,123824, 134,.C-GFVE,1418
```

Simple comma-delimited:
| Field | Example | Description |
|-------|---------|-------------|
| Latitude | `N 30.628` | 30.628°N |
| Longitude | `W 82.470` | 82.470°W |
| Altitude | `36000` | Feet |
| Time | `123824` | UTC hhmmss |
| Speed/heading | `134` | Ground speed or heading |
| Tail | `.C-GFVE` | Registration (dot prefix) |
| Fuel/flight | `1418` | Fuel remaining or flight number |

---

## Southwest — Label 37 (Encrypted/Proprietary)

Confirmed on N8808Q, N8827Q. Label 37 messages from Southwest contain compressed or encrypted proprietary data. The payload appears garbled:

```
036S95B
V:E)KVT,OKOXLBJ00 KFX74J0LLKL550BK4MCK...
```

**Do NOT attempt to decode these payloads.** Note the label, aircraft, and that it's proprietary Southwest content.

---

## Southwest — POS Reports (H1)

Confirmed on N8972S. Extended position with full route decode:

```
POSN38445W076394,OHS01,124105,149,CON01,2,124132,AMEEE,M6,270033,356K,281K,1462,169,KBWI,KFLL,,91,143628,786,73
/PR1462,238,380,169,,46,6,270035,M63,180,P0,P0
/RI:DA:KBWI:AA:KFLL..CON01..AMEEE:D:CONLE5.SCOOB:F:SCOOB..EARZZ..CHIEZ.Y291.MAJIK:A:CUUDA3.MAJIK:F:LOKKR066B
```

**POS line:**
- `N38445W076394` = position 38.445°N, 76.394°W
- `OHS01` = current waypoint
- `124105` = time UTC (hhmmss)
- `149` = heading/track
- `CON01` = next waypoint
- `M6` = temperature (-6°C? or Mach?)
- `270033` = wind (270° at 33 kt)
- `356K,281K` = TAS, IAS (knots)
- `1462` = fuel
- `KBWI,KFLL` = origin, destination

**`/PR` line:** Performance data

**`/RI` line:** Full route decode:
- `DA:KBWI` = departure airport
- `AA:KFLL` = arrival airport
- `..CON01..AMEEE` = waypoints
- `:D:CONLE5.SCOOB` = departure procedure (SID) CONLE5, transition SCOOB
- `:F:SCOOB..EARZZ..CHIEZ.Y291.MAJIK` = enroute (F=filed route) via airway Y291
- `:A:CUUDA3.MAJIK` = arrival procedure (STAR) CUUDA3, transition MAJIK
- `:F:LOKKR066B` = final approach fix or approach procedure

---

## Common H1 sub-patterns

### RESREQ (Resource Request/Response)
Confirmed on N565JB (JetBlue), N951XV (American). Format:
```
RESREQ/AK,1158AF6
```
- `RESREQ` = resource request message type
- `/AK` = acknowledgment
- Hex checksum follows

Often paired with a POS report in the same multi-part transmission.

### RESPWI (Weather Information Response)
Confirmed on N47291 (United). Format:
```
RESPWI/AC,5B/TS124107,010426/DI124105,124107,124107FD3E
```
- `RESPWI` = weather info response
- `/AC` = acknowledgment type
- `/TS` = timestamp
- `/DI` = data interval/parameters
- Hex checksum at end

### FDE (Flight Data Event)
Confirmed on N122SY (SkyWest). Format:
```
FDE1240570401ABZ
12385615310   0   0
12391905310   0   0
```
- `FDE` = flight data event
- Timestamp and parameters follow
- Data lines contain time, altitude(?), and status values
