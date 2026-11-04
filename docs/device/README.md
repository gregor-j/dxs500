# Device scope and evidence

## Authoritative source

**M** is SEFELEC, *Operating manual XS SERIES - English*, PENT0438, v0.55.
All page references are printed pages. The supplied PDF has a one-page cover
offset: printed page 119 is one-based PDF page 120. The manual describes the
whole XS series, not one verified instrument.

Supplementary material was used to find questions, not to define command
grammar, units, ranges, or successful operation. This is an original technical
synthesis; the manufacturer manual and its full safety instructions remain
necessary for operating the equipment.

## Standard DXS500 boundary

| Function | Standard model capability | Evidence |
| --- | --- | --- |
| AC dielectric withstand (Hipot) | 500 VA, up to 5 kVAC; 50 or 60 Hz | M pp. 1, 15-17 |
| Insulation resistance | 10-500 VDC, up to 200 Gohm at 500 V | M pp. 1, 22-23 |
| DC Hipot | Not a standard DXS500 capability; do not infer it from XS506 specifications | M pp. 1, 15, 35 |
| Ground continuity, leakage, SXS eight-step sequencing | Not standard DXS500 functions in this specification | M pp. 1, 123-125 |

RS232 selection and cable requirements are in
[serial communication](serial-communication.md). Only one interface can be
selected at a time; selecting an unavailable interface produces `OPTION MISSING`.
Do not infer installed options from the family command list. [M pp. 47, 119]

## Options and installation-dependent limits

| Item | Manufacturer description | Treatment |
| --- | --- | --- |
| XS-04 | DXS automatic insulation/Hipot cycling | Recognize possible interference with single-function operation; not SXS `SEQ` grammar |
| XS-13 | 6 kVDC, 200 VA Hipot extension for DXS500 and related models | Outside standard AC workflow; no assumed installation |
| XS-20 | Insulation range extension to 2 Tohm | Document conditional range only |
| XS-26 | Insulation voltage extension, 20-1,000 VDC | Conditional voltage capability, not a standard 1,000 V setting |
| XS-07 | 3 mA AC/DC current limitation | Installation can constrain available operation |
| XS-05 | Rear measurement connections | Connection arrangement must be checked on the actual unit |
| XS-06, XS-100, XS-101 | Optional non-RS232 interfaces | Not target transports |

Sources: [M pp. 35, 47-48, 50]. Other options must not be silently enabled by
software. There is no documented complete installed-option discovery response
in the serial command reference. [M pp. 121-129]

Accessory ratings can constrain the instrument: the accessory descriptions
limit certain socket boxes to 4 kVAC and CO210 insulation measurements to
2 Gohm. The relevant accessory rating, not just the generator maximum, must be
checked by qualified personnel. [M p. 3]

Parameter access may be unlocked, read-only, or locked; denied modifications
display `ACCESS DENIED`. No serial password-unlock contract is established
here. A selected DXS option-04 sequence may automatically run multiple
functions or reject a start in the wrong first function. Single-function
operation must not assume sequence mode is off. [M pp. 43, 47-48]

## Verification register

These are explicit limits of the specification, not permission to probe
hazardous conditions. [Issue #4](https://github.com/gregor-j/dxs500/issues/4)
tracks separately approved observation and normal-path reproduction.

| ID | Missing or conflicting evidence | Consequence |
| --- | --- | --- |
| V1 | Ordinary query terminators, response/XON order, Z interleaving, multi-query boundaries [M pp. 120, 126-127] | Exact receive framing and synchronization cannot yet be completed |
| V2 | Lowest rate printed as 9.2 kbaud; wiring figure is blank in supplied PDF; DTR/DSR behavior unspecified [M p. 119] | Confirm selected rate and cable/handshake; do not invent pinout or substitute 9,600 baud |
| V3 | `REM` restricted to startup, but reset requires another `REM`; post-`GTL` re-entry unspecified [M p. 121] | No automatic reconnect/re-entry guarantee |
| V4 | AC minimum 100 V versus command minimum 10 V; current maxima 110/109 mA versus setup 99.9 mA [M pp. 15-16, 62, 123] | Affected supported ranges need clarification; do not silently select a limit |
| V5 | Insulation threshold minimum 50 kohm versus setup 0 kohm [M pp. 23, 50] | Lower bound and disabling behavior need clarification |
| V6 | Local fractional timing and V/s versus serial integer seconds; no documented frequency-selection command [M pp. 17, 23, 53, 124-125] | Do not claim all front-panel settings are remotely expressible |
| V7 | `IDE`/ramp-current scaling, `IDERA` versus summary `IDRA`, incomplete `*LRN?` grammar [M pp. 122, 126, 128] | Do not invent units, aliases, or readback parsing |
| V8 | Initial masks, exact Z triggers/coalescing, pass-bit validity outside completion, STB bit-6 summary semantics [M pp. 120-122] | No event-only completion or stale-verdict assumptions |
| V9 | Serial `STOP`/`QUIT` discharge effects, result retention, safe-state indication, communication-loss behavior [M pp. 54-55, 70, 126] | No software discharge or safe-touch guarantee |
| V10 | Actual identity/options/accessories/locks, nonnumeric results and fault encodings [M pp. 35, 43, 55, 71, 122, 126-127] | Do not infer standard configuration, all response forms, or error mappings |

Manufacturer clarification or scoped device evidence must state which item it
resolves and which firmware/configuration it covers. A normal completed test
does not validate fault recovery, numeric endpoints, or every option.
