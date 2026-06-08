# CPDLC (Controller-Pilot Data Link Communications) Decode Reference

## Overview

CPDLC enables text-based communication between ATC and flight crews, replacing or supplementing voice. CPDLC messages travel over ACARS using Label AA (uplink/downlink) or wrapped in H1 messages for delivery to avionics peripherals (FMC, etc.).

Two main CPDLC implementations exist:
- **FANS-1/A** (Future Air Navigation System): Boeing FANS-1 + Airbus FANS-A. Used in oceanic/remote airspace (NAT, Pacific, etc.). Based on RTCA DO-219/DO-258.
- **ATN CPDLC** (Aeronautical Telecommunication Network): ICAO standard for continental use. Used in European (Link 2000+) and other domestic airspace.

## Message structure

When decoded by a VDL2 tool, CPDLC messages appear with these fields:

```
CPDLC Message:
  Msg Type: fans1a_cpdlc_msg    ← or atc_cpdlc_msg
  Crc Ok: Yes                   ← CRC integrity check
  Gs Addr: USADCXA              ← Ground station address (ATC facility)
  Air Addr: .N901WN             ← Aircraft address (dot + tail number)
  Cpdlc:
    Err: No                     ← Error flag
    Atc Downlink Msg:           ← or Atc Uplink Msg
      Header:
        Msg Id: 3               ← Sequence number for this message
        Msg Ref: 3              ← References the uplink being responded to
        Timestamp: 11:55:52 UTC
      Atc Downlink Msg Element Id:
        Choice Label: WILCO     ← The actual response/instruction
        Choice: dM0NULL         ← Machine-readable element code
        Data:                   ← Additional data if present
```

## Ground station addresses

The `Gs Addr` field identifies the ATC facility. Common US addresses:
| Address prefix | Facility |
|----------------|----------|
| `USADCXA` | US domestic CPDLC (various centers) |
| `KZNY` | New York Oceanic |
| `KZMA` | Miami Oceanic |
| `KZAK` | Oakland Oceanic |
| `CZQX` | Gander Oceanic (Canada) |
| `EGGX` | Shanwick Oceanic (UK/Ireland) |
| `BIRD` | Reykjavik (Iceland) |

## Downlink elements (aircraft → ground)

These are responses or requests from the crew.

### Response elements
| Code | Label | Meaning |
|------|-------|---------|
| `dM0NULL` | WILCO | Will comply with the instruction |
| `dM1NULL` | UNABLE | Unable to comply |
| `dM2NULL` | ROGER | Message received, understood |
| `dM3NULL` | STANDBY | Need time before responding |
| `dM4NULL` | AFFIRM | Affirmative / Yes |
| `dM5NULL` | NEGATIVE | Negative / No |

### Request elements
| Code | Label | Meaning |
|------|-------|---------|
| `dM6level` | REQUEST [level] | Requesting a specific flight level |
| `dM7level` | REQUEST BLOCK [level] TO [level] | Requesting a block altitude |
| `dM9speed` | REQUEST [speed] | Requesting a specific speed |
| `dM10position` | REQUEST DIRECT TO [position] | Requesting direct routing |
| `dM13position` | AT [position] REQUEST [level] | At position, request level change |
| `dM18speed` | DUE TO PERFORMANCE | Unable due to performance |
| `dM19` | DUE TO WEATHER | Unable due to weather |
| `dM20NULL` | NEGATIVE, UNABLE DUE TO WEATHER | |
| `dM21time` | REQUEST [level] AT TIME | |
| `dM22position` | REQUEST [level] AT POSITION | |
| `dM25clearance` | REQUEST CLEARANCE | |
| `dM27distance` | REQUEST DIST [offset] [direction] OF ROUTE | Requesting lateral offset |
| `dM67NULL` | REQUEST CLIMB | |
| `dM68NULL` | REQUEST DESCENT | |
| `dM70position` | REQUEST HEADING [heading] | |

### Report elements
| Code | Label | Meaning |
|------|-------|---------|
| `dM48positionreport` | POSITION REPORT | Automated position report via CPDLC |
| `dM65NULL` | DUE TO WEATHER | Weather-related deviation |
| `dM66NULL` | DUE TO AIRCRAFT PERFORMANCE | Performance-related |
| `dM80level` | LEAVING [level] | Reporting departure from altitude |
| `dM81level` | REACHING [level] | Reporting arrival at altitude |

## Uplink elements (ground → aircraft)

Instructions from ATC to the crew.

### Clearance elements
| Code | Label | Meaning |
|------|-------|---------|
| `uM19level` | CLIMB TO AND MAINTAIN [level] | Altitude clearance up |
| `uM20level` | DESCEND TO AND MAINTAIN [level] | Altitude clearance down |
| `uM23speed` | MAINTAIN [speed] | Speed assignment |
| `uM24position` | FLY HEADING [heading] | Heading assignment |
| `uM36level` | EXPEDITE CLIMB TO [level] | Urgent climb |
| `uM37level` | EXPEDITE DESCEND TO [level] | Urgent descent |
| `uM46position` | CROSS [position] AT [level] | Crossing restriction |
| `uM47position` | CROSS [position] AT OR ABOVE [level] | |
| `uM48position` | CROSS [position] AT OR BELOW [level] | |
| `uM49position` | CROSS [position] AT AND MAINTAIN [level] | |
| `uM73position` | PROCEED DIRECT TO [position] | Direct routing clearance |
| `uM74position` | PROCEED DIRECT TO [position] AT [level] | Direct + altitude |
| `uM79position` | CLEARED TO [position] VIA [route] | Route clearance |
| `uM80level` | CLEARED [level] | Simple level clearance |

### Information elements
| Code | Label | Meaning |
|------|-------|---------|
| `uM99NULL` | CURRENT ALTIMETER [value] | Altimeter setting |
| `uM106position` | MAINTAIN [speed] OR GREATER | Speed restriction |
| `uM107position` | MAINTAIN [speed] OR LESS | Speed restriction |
| `uM116NULL` | RESUME NORMAL SPEED | Cancel speed restriction |
| `uM117unit` | CONTACT [unit] [frequency] | Handoff to new frequency |
| `uM118unit` | MONITOR [unit] [frequency] | Monitor a frequency |
| `uM120unit` | WHEN READY CONTACT [unit] [frequency] | |
| `uM169NULL` | RADAR CONTACT | ATC has radar contact |
| `uM170NULL` | RADAR CONTACT LOST | |
| `uM171NULL` | RADAR SERVICE TERMINATED | |

### Response/acknowledgment elements
| Code | Label | Meaning |
|------|-------|---------|
| `uM0NULL` | UNABLE | ATC unable to comply with request |
| `uM1NULL` | STANDBY | ATC needs time |
| `uM2NULL` | REQUEST DEFERRED | |
| `uM3NULL` | ROGER | |
| `uM4NULL` | AFFIRM | |
| `uM5NULL` | NEGATIVE | |

## Message flow

Typical CPDLC exchange:
1. **Uplink** (ATC → aircraft): e.g., `uM19 FL350` = "Climb to and maintain FL350"
2. **Downlink** (aircraft → ATC): e.g., `dM0NULL` (WILCO) referencing the uplink's Msg Id

The `Msg Ref` field in the downlink header links the response to the specific uplink it's replying to. `Msg Id` is the sequence number of the current message.

## Decode tips

- **FANS-1/A vs ATN**: Check `Msg Type` field. `fans1a_cpdlc_msg` = FANS oceanic, `atc_cpdlc_msg` = ATN continental.
- **Multi-element messages**: A single CPDLC message can contain multiple elements (e.g., a clearance with both altitude and speed).
- **FREE TEXT elements**: Both uplink and downlink support free text for non-standard communications. These appear as text strings rather than coded elements.
- **Emergency**: `dM55NULL` through `dM58NULL` are emergency-related downlinks (PAN PAN, MAYDAY, fuel emergency, etc.).
- **CRC**: The `Crc Ok` field confirms message integrity. If `No`, the message may be corrupted.
- **Latency**: CPDLC messages can have noticeable latency (seconds to minutes) compared to voice. The timestamp shows when the message was generated, not necessarily when it was delivered.
