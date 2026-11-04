# DXS500 command reference

Source **M**: SEFELEC, *Operating manual XS SERIES - English*, PENT0438,
v0.55; all references are printed pages. This reference covers standard DXS500
AC Hipot and insulation, not every XS-series command.

## Reference conventions

Contexts: **Init** = initialization/start display, **Setup** = configuration,
**Hip** = Hipot, **Meg** = insulation. **All** means these four contexts, not
permission to change configuration during an energized test. M does not give a
complete running-versus-idle command-validity matrix. Configure while idle;
run-state ambiguities remain unverified. Contexts come from M pp. 127-129,
qualified by dedicated descriptions on pp. 121-126.

Every outgoing message ends in LF and uses the XON processing acknowledgement.
Query rows below describe payloads **in addition to** that synchronization,
not replacements for it. Ordinary reply delimiters/order remain V1.
XON alone does not establish that a parameter was accepted without a dialog
error. See [serial communication](serial-communication.md) and
[status and errors](status-and-errors.md). [M pp. 120-122]

NR1/NR3 denote the numeric formats in the serial document, not arbitrary unit
suffixes. Use volts, amperes, ohms, and documented time units, not application
kV/mA/Mohm conventions. Parameter changes affect the selected stored parameter
set; persistence scope of global Setup settings is not fully established.
[M pp. 53-54, 65-66, 123-126]

## Control, identity, and registers

| Syntax | Context | Purpose, argument constraints, payload, and side effects | Source |
| --- | --- | --- | --- |
| `REM` | Init | Enter remote; first command, startup-screen restriction; no prior XON wait. Re-entry unresolved (V3). | M pp. 121, 131 |
| `GTL` | All | Return local; does not establish stop/discharge completion. | M p. 121 |
| `LLO` | All | Lock return to local; restoration semantics not fully specified. Not part of default workflow. | M p. 121 |
| `SRQ` | All | Enable Z event notification; send after REM. Initial masks and exact triggers unresolved (V8). | M pp. 120, 126, 129 |
| `*SRE <NR1>` | All | Set status/service-request enable mask; selects request reasons. Integer bit mask; independent numeric min/max not stated. Do not invent defaults. | M p. 121 |
| `*ESE <NR1>` | All | Set event enable mask controlling event summary requests; integer bit mask; independent min/max not stated. | M p. 121 |
| `*SRE?` | All | Return SRE mask in hexadecimal notation. | M p. 122 |
| `*ESE?` | All | Return ESE mask in hexadecimal notation. | M p. 122 |
| `*STB?` | All | Return status byte in hexadecimal `#H...` notation; no read-to-clear rule stated. | M p. 122 |
| `*ESR?` | All | Return event byte in hexadecimal `#H...` notation; reading clears its bits. | M p. 122 |
| `*CLS` | All | Reset standard registers to power-on state; not a documented HV shutdown or full instrument reset. | M p. 121 |
| `*RST` | All | Near power-on reset; returns local, requires REM again. Complete setting/result effects not specified. | M p. 121 |
| `*IDN?` | Init before function selection | Four comma-separated identity fields; no literal DXS500 response, serial value, or firmware revision assumed. Summary's broader contexts conflict with dedicated instruction. | M pp. 122, 127 |
| `*TST?` | Init | Hex bit mask of missing functions: bit 0 Hipot, 1 megohm, 2 continuity, 3 leakage. Not an installed-option inventory or safety certification. | M pp. 123, 128 |
| `*LRN?` | Hip, Meg | Return current function/state/parameters in a form intended for later reuse. Full grammar and complete coverage unspecified (V7); do not promise snapshot/restore. | M pp. 122, 127 |

## Function selection and shared parameters

| Syntax | Context | Contract and constraints | Source |
| --- | --- | --- | --- |
| `CONF` | Init, Setup | Select configuration; dedicated text requires initialization display. | M pp. 123, 128 |
| `HIP` | Init, Hip | Select Hipot; dedicated text requires initialization display. | M pp. 123, 129 |
| `MEG` | Init, Meg | Select insulation; dedicated text requires initialization display. | M pp. 123, 129 |
| `PAR <NR1>` | Hip, Meg | Select active function's parameter set. Ten memories 0-9 in operating chapters; command definition gives no separate bounds. | M pp. 53-54, 65-66, 123 |
| `ACV <NR1>` | Hip | Set AC volts in active set. Command range 10-5,000 conflicts with 500 VA specification 100-5,000 (V4). AC setting resolution not independently specified here. | M pp. 15, 123 |
| `DCV <NR1>` | Meg | Set DC volts: standard 10-500, 1 V steps; option XS-26 20-1,000. Hipot DC context in family table requires separate capability, not standard DXS500. | M pp. 22, 35, 50, 123, 128 |
| `HTIM <NR1>` | Hip, Meg | Set hold seconds. Hip AUTO 1-999; Meg 0 for continuous or 1-999. | M pp. 17, 23, 124 |
| `RTIM <NR1>` | Hip, Meg | Set rise seconds. Local fractional/V/s capabilities do not define additional serial syntax (V6). See function timing tables. | M pp. 17, 23, 53, 124 |
| `FTIM <NR1>` | Hip, Meg | Set fall time. Definition omits unit; seconds are supported by adjacent timing descriptions, but fractional/V/s encoding is unestablished (V6). | M pp. 17, 23, 124 |
| `HLIM <NR3>` | Hip, Meg | High threshold in A for Hip, ohm for Meg. Must exceed low threshold. See function tables for ranges/conflicts. | M pp. 16, 23, 50-51, 62, 124 |
| `CBHLIM <NR3>` | Hip, Meg | Reset low threshold before setting high threshold, same units as HLIM. Final low threshold must then be explicitly configured; reset is not proof that zero is a valid final Meg setting. | M p. 124 |
| `LLIM <NR3>` | Hip, Meg | Low threshold in A for Hip, ohm for Meg; must be below high. Hip zero disables minimum-current comparison. Meg minimum conflicts (V5). | M pp. 16, 23, 50-51, 62, 124 |

These setters have no documented value-echo payload; use message acknowledgement
and error/status interpretation rather than invented readbacks.
[M pp. 120-126]

## Setup and advanced settings

| Syntax | Context | Contract, applicability, and limits | Source |
| --- | --- | --- | --- |
| `BEEP ON` / `BEEP OFF` | Setup | Control failed-result audible signal; cannot suppress all beeps. M's example incorrectly uses DISP; do not infer equivalence. | M p. 125 |
| `DISP ON` / `DISP OFF` | Setup | Control display mode; exact effects on serial timing/results not defined. | M p. 125 |
| `EXP ON` / `EXP OFF` | Setup | Enable/disable expert mode; needed to deliberately control applicable expert settings. | M pp. 125, 129 |
| `M335 ON` / `M335 OFF` | Setup, expert | Power mode associated with EN 60335/60204; operating chapter describes at least 100 mA AC capability at 500-4,000 VAC. No claim of installation compliance. | M pp. 67, 125 |
| `FILT NOR` / `FILT CAP` / `FILT RHT` | Setup | Normal/capacitor/real-hold-time modes. CAP supports stable capacitive insulation readings; DC-specific Hipot regulation/discharge behavior must not be assigned to AC. p. 68 also recommends CAP for capacitive Hipot specimens without defining AC details. | M pp. 42, 68, 125 |
| `TIM AUT` / `TIM FAIL` / `TIM UDIV2` | Hip | Automatic rise/hold/fall; untimed DEFAULT (FAIL token); AUTO starting at half test voltage. FAIL is applicable to 500 VA Hipot. No serial MAN token documented. | M pp. 17, 64, 125, 129 |
| `DET OFF` / `DET I` / `DET FI` / `DET I+DELTA` / `DET FI+DELTA` / `DET DELTA` | Hip | Select no detection, IMAX, fast IMAX, or Delta-I combinations. OFF also removes load-based voltage adjustment. | M pp. 15-16, 126 |
| `IDE <NR3>` | Hip | Delta-I setting; physical range 10-100 mA per 10 microseconds, 10 mA steps. Serial definition does not explicitly establish argument scaling (V7). | M pp. 15, 63, 126 |
| `RDET OFF` / `RDET I` / `RDET FI` / `RDET I+DELTA` / `RDET FI+DELTA` / `RDET DELTA` | Hip, expert | Select common rise/fall detection mode. Software IMAX modes require current display and ramp times at least 2 s. | M pp. 66, 126 |
| `IMRA <NR3>` | Hip, expert | Ramp IMAX/fast-IMAX threshold; physical limits refer back to main current adjustment. Exact serial scaling not explicit (V7). | M pp. 62, 67, 126 |
| `IDERA <NR3>` | Hip, expert | Ramp Delta-I threshold; physical adjustment refers to Delta-I settings. Summary spells IDRA; accepted spelling/scaling requires confirmation (V7). | M pp. 63, 67, 126, 128 |
| `UDISP ON` / `UDISP OFF` | Hip, expert | Enable/disable voltage display during ramps; display selection can affect ramp linearity. | M pp. 67, 126 |
| `IDISP ON` / `IDISP OFF` | Hip, expert | Enable/disable current display during ramps; required for software IMAX ramp detection. | M pp. 66-67, 126 |

No undocumented ranges, factory settings, query variants, or aliases are implied.
Advanced settings are not permission to disable required protection.

## Execution and results

| Syntax | Context | Payload/effect and limitations | Source |
| --- | --- | --- | --- |
| `MEAS` | Hip, Meg | Run selected function with active parameters; not proof of eventual PASS. Do not repeat after an uncertain exchange. | M pp. 54, 68, 126, 129 |
| `MEAS?` | Hip, Meg | Current measurement values: Hip `VOLT <number> AMP <number>` in V/A; Meg `OHM <number>` in ohm. Scientific examples do not define every numeric/nonnumeric case. Not a verdict query. | M pp. 126-127 |
| `STOP` | Hip, Meg | Stop current function; no documented serial discharge-complete or safe-touch guarantee. | M p. 126 |
| `QUIT` | All | Exit current function; no documented replacement for STOP or physical safety procedure. | M pp. 126, 129 |

## Exclusions

`GND`, `ACC`, threshold-unit `OHM`/`VOLT`, `LEAK`, and SXS `SEQ`/eight-step
definitions are not standard DXS500 test controls. `CAD`/`CTH` describe SXS16
capacitive-sample detection, not the general CAP filter. `MOD`/`SBS` describe
other sequence families. XS-04 does not establish compatibility with that
grammar. Hipot `DCV` requires a confirmed DC option outside this AC contract.
[M pp. 35, 47, 65, 68, 123-126]
