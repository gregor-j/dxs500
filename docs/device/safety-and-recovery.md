# Safety and recovery

Source **M**: SEFELEC, *Operating manual XS SERIES - English*, PENT0438,
v0.55; printed pages. This document does not replace the manufacturer's
operating/safety instructions or the installation's approved safety procedures.

## Preconditions

The instrument can generate lethal voltage and current. M requires qualified
users, protective earth, suitable unmodified manufacturer-approved accessories,
sound leads, accessible power control, and precautions against contact with
energized probes/specimens. Check that the HV indicator is not on before
connecting or disconnecting. Guard connections must not be assumed touch-safe.
[M pp. 37-38, 54, 69]

The double physical safety loop must be in its approved closed condition for
normal voltage generation. An open loop prevents normal testing and may produce
`SAFETY LOOP`. This document provides no bypass or wiring instructions.
Closed-loop status is only one prerequisite, not certification of the safety
installation. [M pp. 38, 55, 71]

Physical remote-trigger mode is different from serial remote control. In that
mode a measurement can stop without displaying a safety-loop-open message;
absence of the display message is not proof of a closed loop.
[M pp. 46-47]

## Documented control effects and limits

| Control or observation | Established meaning | Not established |
| --- | --- | --- |
| `STOP` | Stops current function | Safe voltage reached, discharge completed, result retained, acknowledgement delayed until safe |
| `QUIT` | Exits current function | Equivalent to physical discharge, substitutes for stop |
| `GTL` | Returns local | Stops generation or makes the specimen safe to disconnect |
| `*RST` | Near power-on reset, returns local | Complete HV/discharge/result behavior or universal recovery |
| XON | Message processing completed | Physical safety clearance or valid specimen verdict |
| STB not-running / PASS | Test status indications with validity limits | Discharged output or touch-safe specimen |

Sources: [M pp. 120-122, 126]. No documented serial command is explicitly
equated with the complete front-panel discharge procedure.

M describes insulation discharge current limited by 2.2 kohm. For capacitive
specimens it instructs entering discharge mode and waiting before disconnecting,
with guidance of about one second per 100 microfarads. This is not a universal
upper bound or proof of a safe residual voltage. [M pp. 22, 134]

The 1.5 Mohm discharge path and residual-voltage thresholds described elsewhere
refer to DC Hipot/filter/sequence situations. They must not be asserted as
standard AC Hipot or RS232 discharge-complete guarantees. The physical discharge
key also clears retained display readings; do not infer the same result-lifetime
behavior for every serial cleanup command. [M pp. 15, 42, 48, 55, 70]

## Required recovery behavior

The following are requirements for future software, not undocumented instrument
guarantees:

1. Keep abort, failed specimen, instrument fault, and communication uncertainty
   distinct. Do not convert recovery failures into a successful test.
2. On abort with known synchronized communication, request STOP in its valid
   context. Exit/release only as supported by known state. Report each failed
   cleanup step and any uncertain output state; do not say cleanup succeeded
   merely because it was attempted.
3. If framing or acknowledgement is lost, do not blindly send further messages:
   M warns that overlapping processing can cause erratic operation. A missing
   reply does not show whether START or STOP took effect. Escalate to responsible
   personnel and the approved physical safety procedure. [M p. 131]
4. Do not automatically restart, replay `MEAS`, power-cycle, reset, reopen the
   port, or clear receive data as a safety-recovery shortcut. V1/V3/V9 remain
   blockers to assumptions about recovery and remote re-entry.
5. Treat interlock opening and device errors as invalidating ordinary successful
   completion unless valid current-test evidence establishes otherwise. A stale
   PASS bit must not authorize a new action.
6. Restore production ownership only under an approved handover with device
   state understood. Closing a host file descriptor is not an instrument stop
   or discharge operation.

Do not attempt to validate abort/discharge behavior by deliberately creating
hazardous conditions. The separately approved verification work is scoped in
[issue #4](https://github.com/gregor-j/dxs500/issues/4); its normal-path results
will not prove fault-path safety.
