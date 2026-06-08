# Airline-Specific & User-Defined Label Reference

Per ARINC 620, labels `10`–`4~` are user-defined. Carriers each chose their own report types and formats for these slots. This file is **decode-actionable**: it only documents labels where you can actually parse fields from the payload. Labels seen in captures but with no documented format are listed at the bottom under "Opaque / undocumented" so you know not to chase a structure that isn't there.

Source: empirical patterns from [airframesio/acars-message-documentation](https://github.com/airframesio/acars-message-documentation), cross-referenced with ADS-B Exchange flight traces.

## How to use this file

When you see a label in the `10`–`4T` range, the **preamble** (first 3-8 characters of the payload) is the primary signal for which report type. Match the preamble against the table below, then read the corresponding label section for field parsing.

## Common preambles across labels

| Preamble | Meaning |
|----------|---------|
| `POS` / `POSN` / `POSRPT` | Position report |
| `POX` | Position report (CSN variant) |
| `PRG` | Progress report (Airbus A330) |
| `OFF` / `OFFRP` / `OFF01` | OFF event report |
| `OS ` | Origin-station-prefixed message |
| `INRANG` | In-range report |
| `FST01` | First report |
| `LDR01` | Loader/landing report (Southwest) |
| `LOG ` | Logbook entry |
| `TOC` / `TOD` | Top of climb / Top of descent |
| `MNT01` | Maintenance report |
| `APR01` | Approach report |
| `TKO01` | Takeoff report |
| `ETA01` / `ETA02` | ETA report |
| `WXR01` / `WXRQ` | Weather request |
| `FUELRP` | Fuel report |
| `REP04` / `REP019` / `REP239` | ACMS engine telemetry report |
| `FTX01` | Free text |
| `AGFSR` | Air Canada combined position/ETA |
| `DAT` | Jetstar departure data |
| `LDW` | Landing weight |
| `BRAKE/` | Brake event |
| `CHKS` | Checks status |
| `DOOR/` | Door open/closed event |
| `CHIMES` | Chime / cabin call event |
| `MET01` | Meteorological data |
| `EMR` | Emergency/abnormal report |
| `ACK/` / `ACKMSG` | Acknowledgement |
| `MSC01` | Miscellaneous |
| `OPSCTL` | Operations control |
| `HOLDING REPORT` | Holding pattern report (Republic) |
| `CS REPORT` | Continental services / engine times report |
| `DefaultRpt01` | Air France cabin maintenance |
| `R04/`, `R76/`, `A33...` | Qantas A330 ACMS records |
| `T.0` / `T12` / `T32` | A350 MIAM encoded (binary-in-ASCII, multi-part) |

---

## Label 10 — Various report types

**Carriers:** Southwest (WN), SkyWest (OO).

Multiple coexisting formats:
- **`LDR01,...`** — Southwest landing-data report: `LDR01,189,C,SWA-2600-014,0,N <lat>,W <lon>,<alt>,<unk>,<orig>,<dest>,<dest>,<rwys>,...`. Position is decimal degrees.
- **SkyWest free text** — `1,N,R,2022-06-18 21:50:16,17275575,18JUN,4959,KIAH,KPHX,14048,1,<message text>,...`. Date/time, flight, origin, destination, then free-text body across fields.
- **Position with route** — `/N50.154/E007.477/10/0.72/126/410/LGKL/1447/0085/00016/DINKU/XERUM/1222/BAVAX/1224/`. Slash-delimited with waypoints.
- **`MET01<ICAO>`** — Met data request.
- **`POS<DDhhmm>,N<lat>,E<lon>,<alt>, <unk>,...`** — Position report.
- **`REP04...`** — ACMS report.
- **`<msgnum>/ACKMSG/<time>`** — Message acknowledgement.
- **`<msgnum>/OPSCTL/DA <orig>/DS <dest>/FOB <fuel>`** — Ops control message with free text.

## Label 11 — In-Range Arrival Report

Sent shortly before descent.

**`OS ` prefix (US carriers):**
```
OS KALB /QSL1940
```
Format: `OS <4-char ICAO destination> /QSL<HHMM UTC arrival time>`.

**POS-prefixed full position block:**
```
POS
CAS 261,LAT N 58.698,LON E 23.968,ALT 37998,FOB 52200,UTC 121847,PD ----.--,WD 263,WS 29,TAT - 28
```
Fields: CAS (current air speed kt), LAT, LON, ALT (ft), FOB (lbs), UTC time, PD (unknown), WD (wind dir), WS (wind speed kt), TAT (total air temp °C).

**INRANG keyed:**
```
036327/INRANG/DS LFPG/ETA 2251/FOB 014560
/APU AVAIL/UPL NO
```
Format: `<msgnum>/INRANG/DS <dest>/ETA <hhmm>/FOB <fuel>` plus optional flags.

**Line-by-line:**
```
MKJS-CYYZ
LAT N 33.917
LON W 77.988
ALT 37000FT
CAS 252KT
FOB 167LBS
```

## Label 12 — Position Report

**Carriers:** Spirit (NK), WestJet (WS).

**Spirit `POSN` format:**
```
POSN 295754W 953809,-------,1954, 314,,  23,17788  11,FOB   90,ETA 2000,MMUN,KIAH,
```
Fields (comma-separated): position (`N DDMMSS W DDMMSS`), unknown, UTC time HHMM, unknown, blank, unknown, altitude+unknown, `FOB <fuel>`, `ETA <hhmm>`, origin, destination.

**WestJet `N ` prefix:**
```
N 42.150,W121.187,39000,161859, 109,.C-GWSO,1742
```
Fields: latitude, longitude, altitude (ft), DDHHMMSS, unknown, `.<tail>`, unknown.

**Other variants:** route+airport position lines, simple weather/METAR requests (`WXR FS ZBAA,ZBSJ,UUEE`), and Lufthansa-style line-by-line with date.

## Label 13 — Arrival Gate Request

Estimated arrival report typically followed by gate response from ground.

Formats:
- `KPDX,KLAS,2170` — origin, dest, ETA
- `ARR GATE REQ /AIR ID AS,/DEST KPDX,/ETA 2312`
- `CREW ACKNOWLEDGMENT\nDLYREP RMNDR` — ack of delay report

## Label 14 — Three Distinct Variants

**Variant 1 (BizJets) — starts with `(2`:**
```
(2AAACN40587W 72368ATLANTIC AVIA   611QS           215 492 7060     ETA 940PM QUICK-TURN PICK UP 2 PAX NEED FUEL THX(Z
```
Must end with `(Z` to be complete. Missing `(Z` means a multi-part fragment — ignore or hold for reassembly. Used heavily by ARINCDirect/AeroData for FBO and crew comms.

**Variant 2 (Frontier Airlines) — comma-separated:**
```
6,,,Y,N,Y,KMCO,013006,5291,GOOD EVENING ORLANDO,SEE YOU IN ABOUT AN,HOUR AND TWENTY,THANKS...,
```
Position 7 = destination airport. Position 8 = ETA (HHMMSS UTC). Position 9 = flight number. Positions 10+ = free text broken across fields.

**Variant 3 (United Airlines) — OFF event report:**
```
/14 OFF EVENT      / KSAN KEWR 07 020624/TIME 0206
/AU 22518324/AON 22347321/AIN 22406321/AOT 22514323
```
`OFF EVENT` = gear leaves ground. `KSAN KEWR` = origin/destination from FMS (may be stale leg data). `07 020624` = day + HHMMSS. `/TIME` = detected OFF time. `AU/AON/AIN/AOT` are timing counters of unknown function.

## Label 15 — Mixed Position and Delay Reports

- **`(2N<lat>W <lon>...`** — BizJet position with truncated `(Z` end marker.
- **`FST01<ICAO><ICAO>N<lat>...`** — First report (departure/initial position).
- **Departure Delay Report (text)** — Multi-line listing pushback timestamps (P/B SET, L1/L2 OPEN, R1/R2 OPEN, FWD/AFT CARGO OPEN, engine start, taxi began, comments).

## Label 16 — Position Report (Boeing)

Likely Boeing-specific.

**Simple format:** `N 29.763/W 95.530` or `N 41.481,W122.595,36014,6, 139`.

**Extended format:**
```
KERAX  ,N 49.880,E  12.421,33998,0478,1249,029\TS121230,040822
```
Fields: waypoint, lat, lon, alt, unknown, unknown, unknown, `\TS<HHMMSS>`, date `DDMMYY`.

**Ryanair B737 MAX:** `120925,9991,1219, 94,N 53.863 W 2.253` = HHMMSS UTC, altitude ft, unknown, unknown, position.

**Qatar 777:** `POSA1N52969E 16393,RANOK ,080055,330,KORUP ,083244,,-53, 54, 93,828` = position, waypoint, time, FL, next waypoint, ETA, blank, OAT (°C, `-` or `M` = minus), unknown, unknown, unknown.

**Alaska/AeroMexico `N ` variant:** `N 44.203,W 86.546,31965,6, 290` (decimal degrees, alt, unknown, unknown). AeroMexico uses `N 28.177/W 96.055` (slash separator, position only).

## Label 17 — Weather/METAR Request

**Carriers:** Spirit (NK), SkyWest (OO).

- **SkyWest format:** `N,E,2022-06-18 22:37:37,12930,METAR,KMAF,` — METAR request for KMAF.
- **Spirit format:** `0,KFLLKAUS2108` — origin+dest+ETA concatenated.
- **`POS<date>/N<lat>/E<lon>/<alt>/A<alt>/M<temp>`** — Position with altitude and OAT.

## Label 18 — POX / Position with State

**`POX` prefix:**
```
POX,CSN433    ,04AUG22,121839,PANC,KORD,N048.89,E0-99.14,309, 247,838, 121.7, -69.0,035,-39.1,135139, 32240,
```
Fields: `POX`, flight, date, time HHMMSS, origin, destination, lat, lon, then state vector. Note lon is signed (`E0-99.14` = -99.14).

**TEI format also seen:** `FI FI0521/DA EDDF/DS BIKF/OT 1241/FB 0317/UTC 120220804124116` — standard TEIs.

## Label 19 — IN/Post-flight Fuel and Reports

- **TEI format:** `FI <flt>/DA <orig>/DS <dest>/OF <hhmm>/FB <fuel>/BF <board fuel>/UTC <hhmm>...`.
- **United IN FOB report:**
  ```
  /19 IN FUEL FOB RPT/ KIAD EDDF 03 053858
  /M17 POSTFLIGHT RPT FOB0271
  /POSTFLIGHT DISPLAY FOB0271
  /FMC CALCULATED IN FOB----
  ```

## Label 1E — United Brake-After-Engine-Start

```
/1E BRAKE AFTER ENG/ EDDF KORD 04 065458/TIME 065458
(FIRST BRAKE RELEASE TIME AFTER ENGINE START)
/LOC +50.0456,+008.5644
```

## Label 1G — OOOI Block

**Pattern:** `<orig>,<dest>,<date>,<event1>;<time>;,<event2>;<time>;,<elapsed>,<flight>`. Event codes like `93`, `31B`, `042`, `082` indicate OOOI markers. Used by Brussels Airlines (SN).

## Label 1L — Multi-airline obfuscated counter

**Carriers:** Lufthansa (LH), Austrian (OS), Condor (DE), EgyptAir (MS), Sunwing (WG), Frontier (F9).

Most variants are obfuscated header counters + airline-specific payloads. Condor (DE) format includes plain-text fields:
```
0002022120000325114AUG22    KBWIEDDFDE2075    .D-ABUC,10582,,0800,,LT,  72600,86880,-,-
```
Sunwing (WG) carries GPS position+state vector after the counter:
```
00708213200/GS 457125/DEP CYUL/DES MUVR/ETA 0328/GW 721/ALT 33993
```
TEI keys: GS (groundspeed), DEP/DES (orig/dest), ETA, GW (gross weight), ALT.

## Label 1M — ETA Reports

```
/AY1792/ETA01/040822/LIRN/EFHK/EFHK/2JK0
1220/EFHK22L/80
```
Fields: flight, ETA01 marker, date, origin, destination (twice), unknown code, then arrival time, runway, fuel remaining.

## Label 1P — Position Report

```
121611,N 52.622,E  8.601,32000,301,506,2301,91.90T
```
Fields: HHMMSS UTC, lat, lon, altitude (ft), heading (true), groundspeed kt, fuel/time, fuel weight + `T` (tonnes).

## Label 1R — Initial Report / Gate Return

- **`/<flt>/INT01/<date>/<orig>/<dest>/<final?>/<code>`** — Initial flight report.
- **`/1R GATE RTN       / <orig> <dest> <DD> <HHMMSS> CREW REPORT`** — United gate-return event.

## Label 1X — TUI Acceptance Message

```
00072219700/TX FRAOPXH/PA ACCEPT/FN TUI84X/EN EDN01
05AUG22/UTC 030819/DA EDDF/DS LGKO/PK 013980/CLI X34604
RMK
```
TUI ticket acceptance with TX (ticket), PA (acceptance), FN (flight), EN (station), UTC, DA/DS (orig/dest), PK (ticket weight?), CLI (client?), RMK (remarks).

Also seen as short `<orig>,<dest>,<DDMMMYY>,<HHMM>` blocks with position next line.

## Label 20 — POS / Position Reports

**`POS` preamble:**
- Type 1: `POSN38160W077075,,211733,360,OTT,212041,,N42,19689,40,544` — position (DDMMSS), unknown, UTC HHMMSS, FL, current waypoint, ETA to next, blank, OAT, fuel, unknown, groundspeed.
- Type 2: `POSN32249E045047,,082806,380,DEBNI` — truncated.
- Type 3: `POS,,,,,,,,,,` — empty (keepalive).

**`RST23A` reset/state:** `RST23A08095255KLM1385 EHAMUKBB` = reset event + day + time + flight + origin+dest.

## Label 21 — Position Report (extended)

```
POSN 34.428W 77.692, 230,065030,31995,32372,  45,-52,094730,TJSJ
```
Fields: position (decimal degrees), heading, time, alt, alt2 (target?), unknown, OAT (°C), ETA, destination.

## Label 22 — Position with State (Frontier)

```
N 404630W 735400,-------,042530,9151, ,      , ,M  5,26173  15, 206,
```
Position (DDMMSS), unknown, UTC, altitude, blank, blank, blank, temp (M = minus), unknown+unknown, heading.

## Label 23 — Landing Notifications and Brake Events

- **`ONN01<flt>/<date><time><orig><dest><time><elapsed>`** — On/landing notification.
- **`BRAKE/BRAKE RLSD <HHMM>`** or **`BRAKE/BRAKE SET <HHMM>`** — brake events.

## Label 24 — OFF / Position Events

`<HHMMSS> <orig> <dest><digit>` lines, with optional `/FN <flightnum>`. Some also contain slash-delimited position blocks `/<date>/<time>/<unk>/<alt>/N<lat>/E<lon>/<unk>/<time>/`.

## Label 25 — Pushback/Out events

`SNS01<flt>/<date><time><orig><dest><event><time>` where event ∈ {A=out, F=off, B=on, ...}.

## Label 26 — ETA Report with Weather (line-by-line, Aeroflot)

```
ETA01<flight>/<date><time><orig><dest>
FUEL <kg>
TEMP <C>
WDIR <ddd><wind>
WSPD <speed>
LATN <lat>
LONE <lon>
ETA <hhmm>
TUR
ALT <ft>
```
Each parameter on its own line.

## Label 27 — Position Report with Weather (line-by-line)

Same structure as Label 26 but `POS01` prefix.

## Labels 28 & 29 — Aircraft State (Republic Airways)

```
D,H3,1912081232,08DEC,4292,KGSO,KEWR,5500,FF04,Y,Y,Y,N,N,N,-,-,-,-,-,-,-,-,-,
```
CSV row with state flags. Label 29 also carries `MSC01` (misc) messages and de-icing reports:
```
MSC01BOX516    /08081300EDDPVHHH
JLOTT/BDL
EAT/43201/L/0.8
DEICE N
```

## Label 2A — Approach / Maintenance

- **`MNT01<flt>/<date><time><orig><dest>` + ATA code + description** — Maintenance report (e.g., `331 621 00\nCPT CLOCK LIGHT INOP`).
- **`APR01,<flt>,<orig>,<dest>,<date>,<HHMM>,<rwy>`** — Approach report.

## Label 2B — Departure events

Sample: `04212B KJFK KMIA6` followed by counter sequence. Also TEI-format flight events: `/DA EGLL/DS OLBA/OT 1338/OF 1353/FB 167/ETA 1832`.

## Label 2D — Top of Climb / Top of Descent

```
TOD01,ASL89U,LYBE,EDDT,10DEC19,0721,EDDT26R
TOC10,ETD78     ,EHAM,OMAA,17DEC19,0157,OMAA13R ,47,52500,
```
TOC adds FL and weight at TOC.

## Label 2G — Swoop/SkyService Flight Log

**`LOG ` preamble:**
```
LOG C-GLRN00072/DA CYQB/DS CYBG/FN SWG9551/TAH 00266:24
OT 1118/ROT 1118/OF ----/ON 1101/IN 1111/RIT 1111/BT 0000
24FEB20/UTC 111811/LFB 159/LFR 54/LFU 105/FT 0026/TAC 00070
DL1 NO,----/DL2 NO,----/CIS Y/LVO N
```
TEI keys: tail+leg counter, DA (dep aerodrome), DS (dest), FN (flight), TAH (total airframe hours HHHHH:MM), OT (out time HHMM), ROT (revenue out time), OF (off), ON (on), IN (in), RIT (revenue in time), BT (block time HHMM), UTC date+time, LFB (block fuel, units appear to be hundreds of lbs based on values for 737 fleet), LFR (reserve fuel, same units), LFU (used fuel, same units), FT (flight time HHMM), TAC (total air cycles), DL1/DL2 (delay codes + minutes), CIS (carbon offset?), LVO (low visibility ops, Y/N).

Also carries pre-flight `/PF ...` template: `/PF <fuel ground>/PIC ------/IID D` + `BF1 <boarded fuel>/PR1 ---/DN1 .../CLI ------/FR` + `PX <pax>,<...>/OFP <Y|N>/DEN <density>/FIN <fin number>`.

## Label 2H — Engine Trim / Departure Event

`ET301,<flt>,<orig>,<dest>,<date>,<HHMM>,<runway>` — engine trim event at airport.

## Label 2L — Jetstar DAT (departure data)

**`DAT` preamble — Jetstar (JST/JQ):**
```
DAT 16MAR25
UTC 0041
REG VHVGZ
FLT JST952
GWT 0
ZFW 602
FOB   120
CAP 160650
FO  611369
LOG 479189
LDR 0
DRT 0023
```
Each field on its own line. ZFW = zero fuel weight, FOB = fuel on board, FO = block fuel, LOG = engine logbook hours, DRT = duration.

Other 2L variants use `P/<time>/<FL>/\n+<lat>+<lon>/<OAT>/<unk>/<unk>` — short position/weather samples.

## Label 2M — Air France Cabin Maintenance

**`DefaultRpt01` preamble:**
```
DefaultRpt01/F-GSQD 081812
AIRCRAFT:
SEAT4E REMOTE UNIT HS
SEAT3EVIDEO INOP
CABIN:
```
Format: `DefaultRpt01/<tail> <DDHHMM>\nAIRCRAFT:\n<aircraft faults>\nCABIN:\n<cabin faults>`. Air France F-G* tails. Free-text fault descriptions on each line.

## Labels 2N & 2O — Takeoff/Landing Reports (Air France)

```
AF0083/08082356KSFOLFPG
23182350
TKO01AF0083/30302347KSFOLFPG
23042335
```
Flight, double-day+UTC, origin+destination concatenated, then sample times.

Label 2O adds a runway+side and additional event marker:
```
AF0256/23240757LFPGWSSS
07460751  8402L /N
N
```
Second line: HHMM HHMM <unknown> <runway><side> /<flag>.

## Label 2P — FMS Position Sample

`FM3 064536,0721,N 51.228,E 14.567,28279,  379,33` — FMS3 sampler: HHMMSS, unknown, lat, lon, alt, GS, temp.

## Label 2R — Reset Event (United)

`/2R RESET / KEWR MWCR 22 152147/1521/PHASE AIR/FLT 1436` — system reset event with origin/dest, day, HHMMSS, current time, phase (AIR/OTG), flight number.

## Label 2S — Schedule Arrival

```
1546,1558,1615,1627,040,330,A0 SCHEDULED ARR,
```
Four HHMM times (scheduled/actual departure/arrival pairs), two short codes, status text. Status examples: `A0 SCHEDULED ARR`, `DO AIRCRAFT 21:5`.

## Label 2U — Loadsheet (Air France ECD)

```
ECD/F-GSPE
LOADSHEET FINAL EDN01
AF0379/08/08MAR20/2055
YVRCDG
TOW 240602 MACTOW 33.3
ACPT33999611/212025/2059
```
Tail, edition number, flight+day+date, origin+dest, takeoff weight, MAC%, acceptance code.

## Label 3] — CHIMES (Cabin Call)

```
CHIMES,1273,RDU ,PHL ,191101,015659,1,AOC      ,ACTIVE  ,12
```
Fields: `CHIMES`, flight, origin, destination, date YYMMDD, time HHMMSS, count, channel (AOC), status (ACTIVE), unknown.

## Label 30 — ETA (Alaska Airlines)

**`/EA` preamble (Alaska Airlines):**
```
/EA1830/DSKSFO/SK24
```
`EA` = estimated arrival HHMM UTC, `DS` = destination ICAO, `SK` = unknown 2-digit code.

Also carries Delta-style free-text gate messages with `KRDU KSEA7\n/FN 1253\nA C S\nCABIN DOOR CLOSED AT 1203Z...`.

## Label 31 — Various Position/Event (AERODAT)

AERODAT format: `/AERODAT.22,C,GSO,5R,5L,,0,0,040/03,,,0,0,00,-N/A-,45800,41550,46,7.2,1838`. Also `OFF01,<orig>,<dest>,<time>,<unk>`.

## Label 32 — Position with State (SkyWest)

```
H,H,08DEC19 06:49:57,14352840,07DEC,5407,KLAX,KMFR,7860,60,L,N 38.294,W120.619,N 38.068,W120.503,32000,32000,246,107,- 18,  47280,085,320,439500,275.3, 28680,760,0734,,,,0550,
```
Long CSV with two positions (current + target), altitudes, heading, OAT, fuel, weights, times.

## Label 33 — Landing Weight Report (multi-line)

```
FLT NO     LDW      TIME
5407     45592     0704Z
WIND 000/00   8C   29.83
------------------------
REMARKS
THRUST REVERSE CREDIT
```
Multi-line ASCII-art landing report with weights, conditions, and runway data.

## Label 34 — Mixed Performance / State / Runway

No single consistent format; several distinct payloads share this label:

- **Takeoff performance line:** `41 ,3.2 , 1435,DT H054, FLEX - T/O-1 - ECS ON  AD74` — weight, MAC, time, DT code, free-text config.
- **State CSV:** `39,B,08,KFLL,KIAD7752` or `207,C,1,08,KBWI,KCVG,,B8D6` — flight, phase code, day, orig, dest + checksum.
- **Aircraft-type performance:** `G200,274,,,222,386,408,,,,,,,9DEE` — type, then CSV of perf values + checksum.
- **Runway availability** (multi-line, sometimes spans blocks):
  ```
  *03L              11001>
  *03L/L             7933
  *03R              10995>
  *21L              10995>
  A9FC
  ```
  Each line: `*<runway>` + length in feet, optionally with `>` or `/L` suffix.

## Label 36 — Position State (CSV)

`28,E,08DEC19,090218,N 37.985,W 79.875,36997, 10160,KBUR,KJFK,KJFK,31L/,/,,,,,,,,0,0,0,0,0,0,0,,120.0,006.2,` — comprehensive state vector with position, alt, target alt, origin, two destinations (planned/intended), runway, fuel-related, heading, vertical speed.

## Label 37 — Obfuscated/Compressed Proprietary

**Carriers:** Southwest (WN), Republic (YX), Delta (NW/DL), Spirit (NK), Endeavor (9E).

**WN variant** — Obfuscated, payload not decodable:
```
09RD,HI
AWP0KAD0-KJLJHKO/IMY6M:KX/M:Y:N:
```

**YX variant** — Free text (`FTX01`):
```
FTX01YX4687/04042233KJFKKDCA
THANKS
```
Format: `FTX01<flight>/<DDDDhhmm><origICAO><destICAO>\n<text>`.

For WN/Delta payloads, just note the carrier and that the body is compressed/encrypted.

## Label 38 — SX-prefixed (Lufthansa/SWISS)

`SXS9F/EDDT/1151/  35/30M` — `SX` prefix, code, airport, time, unknown, `30M` (30 minutes?).

## Label 39 — Weather Request (WXR01)

```
WXR01<flight>/<date><time><orig><dest>
M        LKPREDDCEDDFLSGGEDDS
```
Lists ICAO stations after a leading `M`.

## Label 3A — Weather Request with TYP/STA

```
WXR01UAE36     /--081512EGNTOMDB
/TYP M/STA LHBP/STA LROP/STA LTFM
```
TYP M = METAR. Up to 3 STA entries per message.

## UPS Operations Family — Labels 3B, 3C, 3E, 3F, 3G, 3I, 3J, 3K, 3M, 3N, 3S, 3U, 3W

UPS uses a consistent base format across this family: `<DDHHMM> <DD><orig 3-char> <dest 3-char> <extra>`. The "extra" portion differs per label:

| Label | Purpose | Extra format |
|-------|---------|--------------|
| `3B` | Operations message | `<HHMM><HHMM><HHMM><HHMM><station> <code>` |
| `3C` | Route stop / next leg | `<HHMM><next station> <count>` (e.g., `1243MHR 1`) |
| `3E` | Maintenance free text | `<freeform text>` (e.g., `FOB 48.0 TOF 47.7`, `NEED NEW UPDATED FLT RLSTIME`) |
| `3F` | ETA downlink | `<HHMM ETA>N` (e.g., `1255N`). `N` flag always present, meaning unknown |
| `3G` | Mechanical fault | `<equip serial><fault description>` |
| `3I` | Sensor metric | `<HHMM><4-digit value>` (e.g., `01040387`) |
| `3J` | ETA (variant 2) | empty (header only) |
| `3K` | Departure metric | `<HHMM><7-digit value>` (3 fields concatenated) |
| `3M` | Short out/event | empty |
| `3N` | Short station report | `<3-char station><14-digit value>` (no DDHHMM prefix, e.g., `IAD03320000010000003`) |
| `3S` | Ground equipment SAFT | `<station> 2000000` or `<ICAO> 00000002SAFT` |
| `3U` | Crew dispatch | `<flight info>\n<dispatch text body>` |
| `3W` | Cargo container manifest | `00001UPS02UP03///04///05/06/...18` (numbered container positions) |

Hubs: **MHR** = UPS Mather (Sacramento, West Coast sort hub), **SDF** = UPS Worldport (Louisville, main hub), **ONT** = UPS Ontario (LA-area gateway), **PHL** = Philadelphia gateway.

## Label 3D — Republic Holding Report

**`HOLDING REPORT` preamble:**
```
HOLDING REPORT
YX: 3558 KEYW KEWR
12/09/19 21:01
CURRENT FOB -  7910
HOLDING FIX - FAK
HOLDING ALT/FL - 37000
EFC TIME - 2135
COMMENTS -
```
Fields: flight + origin + destination, date/time, current FOB, holding fix, holding alt/FL, EFC (expect further clearance) time, free-text comments.

Also carries dispatch acknowledgements: `QUANPOCK4~1DIS01100611\nARR RPT RECVD`.

## Label 3H — Republic Continental Services Report

**`CS REPORT` preamble (multi-block, almost always spans 3+ messages):**
```
CS REPORT
FLT DATE - 20191219
ORG - KDEN
DEST - KSMF
ENGL START - 20191219 22:48:55
ENGL STOP - 20191220 01:15:41
ENGR START - 20191219 22:54:46
ENGR STOP - 20191220 01:15:42
MCD CLOSE - ...
CD CLOSE - ...
AC PUSH - ...
PB RELEASE - ...
TAXI - ...
PB SET - ...
MCD OPEN - ...
CD OPEN - ...
HIGHEST FL - 36018
CRUISE LEVEL OFF +5M  - 34003/454/ .79/11599
```
End-of-flight summary with all OOOI sub-events, engine L/R start/stop times, door events, highest FL, cruise level snapshots.

## Label 3L — Position (Jetstar) / US Carrier Short

Jetstar: `S 33.889/E151.177 /UTC 0438` (decimal degrees + UTC time).
US: `4853GSOIAD11307` — flight, origin (IATA 3-char), dest (IATA 3-char), time/code.

## Label 3P — Position with Performance

`POSN39191W125342,OLOFF,223442,340,REDWD,224245,N41W130,M55,00667,1551FAKE` — Position + current waypoint + UTC + FL + next waypoint + ETA + next waypoint position + OAT + fuel + CRC.

## Label 3R — TOC/TOD Reports

`TOD14DEC19,LOT9004,.9H-SUN,LPBJ,EPWA,1624,00163,0148,384,,0,` — Top of descent. Reverse for TOC.

## Label 3T — METAR/TAF Text

Multi-line METAR/TAF weather reports in standard format. Usually multi-block; needs reassembly.

## Label 3Z — Aircraft State Vector

CSV `12,0,,12,0,,11,0,,11,1,,12,0,,10,0,,0,1,1,0,0,` — Six paired values + status flags.

## Label 40 — User-defined uplink

Reserved per ICAO Doc 4444 / GOLD 2nd ed. Free text uplink from ATC, weather responses, dispatch:
```
"SHOWING HIWC 15NM RADIUS OF LOZ. TOPS ARE AT FL300"
```

## Label 4A — Door Events / Cessna

- **`/01-X`** — `/01-C` or `/01-D` from Cessnas shortly after takeoff.
- **Door events:** `DOOR/FWDENTRY CLSD 1440`, `DOOR/CARGO OPEN 1445`. Format: `DOOR/<door name> <OPEN|CLSD> <HHMM UTC>`.

## Label 4J — Position + Weather Report

```
4J01 POSWX 0318/20 ETAD/ETAD .00318S
/POS N5043.5E01121.8/OVR 0817
/ALT 270/TFW 1342/TAS 490/SAT -032
/POS GOVEN /OVR 0835
/POS DILVI
/WND 334060/TRB /SKY DCC3
```
TEIs: POS (lat/lon DDMM.M), OVR (overhead waypoint time HHMM), ALT (FL/100), TFW (total fuel weight, hundreds of lbs), TAS (true airspeed kt), SAT (static air temp °C), WND (winddir+speed concatenated, 6 digits), TRB (turbulence), SKY (sky/cloud code).

Tanker aircraft common; TFW 1342 ≈ 134,200 lbs.

## Label 4L — Engine Run Events

```
14004L KJFK KPIT7
/FN 4835
/ENG 1 RUN 001244/ENG 2 RUN 003220
```
Format: `<HHMM><label> <orig> <dest><digit>\n/FN <flight>\n/ENG 1 RUN <hours>/ENG 2 RUN <hours>` — Engine accumulated run time in HHMMSS format.

## Label 4N — Position with Airport Pair

```
22024N  MCI  JFK1
0013  0072 N040586 W074421   230
```
Header: `<DD><MM>4N  <orig IATA>  <dest IATA><digit>`.
Body: counter, counter, position (`N<DDMMMM>` / `W<DDDMMMM>` = degrees+decimal-minutes×10), heading.

## Label 4T — Air Canada AGFSR Position/ETA

```
AGFSR AC0841/21/21/FRAYYZ/1011Z/731/5331.2N00249.6W/320/CRUISE/0595/0124/M52/092055/0302/---/535/0853/0910/----/----
```
Slash-delimited fields:
1. `AGFSR` literal
2. Flight number `AC0841`
3. Day of month (departure)
4. Day of month (current)
5. Origin+destination IATA pair `FRAYYZ`
6. Message UTC time `1011Z`
7. Unknown counter
8. Position `<DDMM.M>N<DDDMM.M>W`
9. Flight level (320 = FL320)
10. Phase of flight (`CRUISE`, `CLIMB`, `DESCENT`)
11. Unknown
12. Unknown
13. SAT (°C, `M` prefix = minus)
14. Wind direction+speed concatenated (092055 = 092° at 55 kt)
15. Unknown
16. Unknown
17. Groundspeed (kt)
18. OUT time HHMM (off blocks)
19. Unknown time
20. Unknown
21. Unknown

## Label 80 — Aircrew-Addressed Downlink

Carries `3N01 POSRPT`, `3T06 FUELRP`, etc.

**Position report (3N01):**
```
3N01 POSRPT 0874/07 MMMX/KIAH .XA-VLZ
/POS N29595W095447/ALT +04983/MCH 357/FOB 0053/ETA 2235
```
Header: `3N01 POSRPT <flight>/<DD> <orig ICAO>/<dest ICAO> .<tail>`. TEIs: POS (DDMMMM format), ALT (signed ft), MCH (Mach × 1000), FOB (hundreds of lbs), ETA (HHMM).

**Fuel report (3T06):**
```
3T06 FUELRP  0317 /14  EGLL/ WSSS  .9V-SKU  14082022 1306
/XFQ 000.5/DWX N/PVS N/TKR N/RDP N/EWX N/OVF N/INT N/MIN N/OTH Y
/TXT INCREASE IN ZFW.
```
TEIs: XFQ (extra fuel quantity), then Y/N flags for DWX/PVS/TKR/RDP/EWX/OVF/INT/MIN/OTH reasons, then `/TXT` free-text reason.

## Label 83 — Airline-defined Position/State

**Comma-separated:**
```
LGAV,KEWR,232011, 40.64,- 74.61, 5004,240,-160.0, 20000
```
Origin ICAO, dest ICAO, DDHHMM, lat decimal, lon decimal, alt (ft), GS (kt), heading, unknown.

**ETAT2 variant (multi-block):**
```
4DH3 ETAT2  0907/22 ENGM/KEWR .LN-RKO
/ETA 1641
```

**Charter 001PR fixed-field variant:**
```
001PR19113523N2947.7W09536.51273750177
```
Positions 1-3 = `001`, 4-5 = `PR` (position report), 6-7 = DD, 8-13 = HHMMSS, 14 = lat dir, 15-20 = lat DDMM.M, 21 = lon dir, 22-28 = lon DDDMM.M, 29-33 = altitude, 34-38 = unknown.

## Label B0 — AFN ATS Facilities Notification Contact

```
/KZWY.AFN/FMHJBU803,.N949JT,AD2F71,000203/FPON40122W072597,1/FCOADS,01/FCOATC,01F63B
```
Fields (slash-delimited):
- `KZWY.AFN` — ATSU address + AFN tag
- `FMH<callsign>,.<tail>,<icao hex>,<HHMMSS>` — Flight plan correlation
- `FPO<position>,<unk>` — Contact position in DDMM.M format
- `FCOADS,<unk>` — ADS-C capability
- `FCOATC,<unk>` — ATC capability
- 4-char CRC at end

## Label B1 — Oceanic Clearance Request (RCL)

```
/EGGX.OC1/RCL 046
AFR088-BALIX/1754 M083F360
-RMK/MAX F3701C22
```
Structure: `/<FIR>.OC1/RCL <id>\n<callsign>-<entry waypoint>/<ETA HHMM> M<mach×100>F<FL>\n-RMK/<remarks><CRC>`. FIR codes: `EGGX` (Shanwick), `KZOA` (Oakland), `BDOOCYA` (Bodo).

## Label B2 — Oceanic Clearance Acknowledgement (CLA)

```
/PIKCLYA.OC1/CLA 1636 220610 EGGX CLRNCE 465
DLH404 CLRD TO KJFK VIA GOMUP
NAT B
GOMUP 59N020W 59N030W 58N040W 56N050W JANJO
FM GOMUP/1805 MNTN F360 M080
END OF MESSAGE<CRC>
```
Includes full track (NAT letter or RANDOM ROUTE) and entry/exit fixes.

## Label B6 — ADS Report

```
14ASQ0431/SKYSWSQ.ADS.9V-SMI070286A9E26ACA0320401F0E1E88DA00001033927E36F282
```
Format: `<DD><label><flight>/<station>.ADS.<tail><hex payload>`. ADS-C report from CMU to ground.

## Label B9 — Position Broadcast Reply

```
/KRDU.TI2/024KRDUAB5F0
```
Station+code, 3-digit identifier, station repeat, 5-char CRC. Short link-test or position acknowledgement.

## Label BA — CPDLC Downlink (hex-encoded)

Format: `/<station>.<MTI>.<tail><hex payload>`.
- `/USADCXA.DR1.JY-BAE5C1D` — DR1 message
- `/USADCXA.AT1.A7-ANK608324E503DC50` — AT1 with multi-segment hex
- `/USADCXA.CC1.N961JT61055D6049103F4E` — CC1

Station codes: `USADCXA` (US ATN gateway), `NYCODYA` (New York oceanic).
MTI codes seen: `DR1`, `AT1`, `CC1`. Payload is binary CPDLC data in hex; passes through to the FANS/CPDLC decoder.

## Label C1 — Fuel Request (line-by-line)

```
DPCCAMH
270822
AGM AN
9M-MTO/FI MH0714/MA 154I -
//////LIDO FUEL FIGURE//////
MH 129
9M-MTO
KUL MEL 27-AUG-2024 (UTC)
SI-BASED ON ZFW : 167800 KG
TRIP FUEL : 45184 KG
TAXI FUEL : 375 KG
BLOCK FUEL REQUESTED : 57800 KG
```
Multi-line fuel order with LIDO format. Also carries Hawaiian Airlines (HA) loadsheet finals with ZFW/TOW/MAC/seat row trim.

## Label HX — Undelivered Uplink Report

System message when an uplink failed.

- **`RA FMT LOCATION` prefix:** `RA FMT LOCATION N4009.6 W07540.8` — last-known position of receiver.
- **`RA FMT 43` prefix:** `RA FMT 43 GSP B02` — 3-char airport + 2-3 char code.
- **`RA IV3` prefix:** carries HOWGOZIT data multi-part.

## Label MA — A350 MIAM Encoded

Almost exclusively A350. Messages start with `T.0`, `T12`, or `T32` and contain binary-in-ASCII payload (printable characters with embedded control bytes). Almost always multi-part; needs full reassembly before decoding. Treat as opaque encoded data unless you have the MIAM decryption keys. Message bounds: `T.0&` ... `|`.

## Label Q0 — Link Test

Always empty payload. Pure connectivity verification.

## Label Q2 — ETA (compact)

```
   2002  99/DS KJFK
```
Three spaces, HHMM ETA, space, 2-3 digit unknown, `/DS <dest ICAO>`.

## Label QF — OFF/Destination

```
EWR2210ATL
```
Compact: 3-char origin IATA, HHMM UTC, 3-char destination IATA.

## Label QQ — OFF Report (ICAO) with optional position

```
KTEBKJYO1528001FE23152852N4052.1W07403.0014195
```
Fields: origin ICAO, dest ICAO, HHMM, `001FE`, DD, HHMMSS, position (DDMM.M format), more state. Position is optional; some QQ reports have only origin+dest+time.

## Label SA — Media Advisory

Format: `<digit><2-char airline><HHMMSS><suffix>` short link-state messages.

| Suffix | Meaning |
|--------|---------|
| `V` | VHF |
| `V/` | VHF preferred |
| `VS` | VHF + SATCOM |
| `S` | SATCOM |
| `S/` | SATCOM preferred |

Example: `0LH151351VS` = priority 0, Lufthansa, time 15:13:51, VHF+SAT.

Also seen: `<digit>E<6-digit>` short forms.

## Label `:;` — Aircraft Transceiver Frequency Change

Payload is a frequency in Hz or kHz (divisible by 1000 or 1,000,000):
```
 130025
 131550
 1317251200
```
Long form includes additional 4-digit suffix of unknown function.

## Label `_d` — Frame Acknowledgement

Empty content. Acknowledges receipt of a previous frame; `ack` field carries the `blk_id` being acknowledged. If no `_d` is received, originator retransmits after timeout. Not operationally useful except for DX/link analysis.

---

## Opaque / undocumented

These labels are observed in the wild but have no documented decode format — payloads are either obfuscated/encoded, single-value tokens with no apparent structure, or simply lack examples in the research. **Don't attempt to parse fields; just note the label and carrier (if known).**

| Label | What you'll see | Notes |
|-------|-----------------|-------|
| `1A`, `1B`, `1C`, `1D`, `1F` | `00015207278DA6*)B3H3OLM:::U4U11TT+8Q,` style | 5-digit counter + 6-digit timestamp + obfuscated/compressed payload. Some plain-text variants exist (e.g., Condor `DE1570,1217,EDDF,M` flight events; clean OOOI rows like `U003,,,,YMEK,YPPH,0241,040822,0241,EOF,`). Treat opaque body as proprietary |
| `1Q` | `LSZH,10,,,,,,,,` | Just an airport+digit, no payload |
| `1S` | `SAV`, `LEX`, `CHO`, `TYS` | Single airport/waypoint token |
| `1T` | Equipment serials `SN2258/22/,SN2257` or `.<tail>/<numbers>` | Brussels Airlines flight number relations; no structured decode |
| `1U`, `1V`, `1Y`, `1Z` | Numeric column data | ACMS-style sensor/engine telemetry, no documented field map |
| `1W` | Frontier gate agent free text (`GSO1000000000GATE AGENT TRIED TO BOARD 51PAX...`) or EIN crew snippets | Free text, no structure |
| `2C`, `2E`, `2F`, `2I`, `2J`, `2K`, `2T`, `2W` | Numeric/encoded data | Sensor or obfuscated; no decode |
| `2Q`, `2V`, `2X`, `2Y` | (empty in repo) | Observed but no examples documented |
| `2Z` | `EGLL` | Single airport code |
| `35` | `?????` | Literally documented as unknown placeholders |
| `3V` | `NOC01,<flt>,<orig>,<dest>,<date>` free text or `T/O PERF REQ ACCEPTED` | Notice messages + perf acks |
| `4P` | `285,B,,0,,MCO,,,7049` | Sparse CSV, no documented field map |
| `B5` | (empty in repo) | Observed ATS label, no documented format |
