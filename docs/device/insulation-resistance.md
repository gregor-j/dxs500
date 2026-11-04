# Insulation-resistance measurement

Source **M**: SEFELEC, *Operating manual XS SERIES - English*, PENT0438,
v0.55; printed pages. This is the DXS500 insulation function, not DC Hipot.

## Capabilities and configuration

| Quantity | Manufacturer contract | Serial control / limitation |
| --- | --- | --- |
| Test voltage | Standard 10-500 VDC, 1 V steps; accuracy +/- (1% + 1 V) | `DCV <NR1>` volts |
| Polarity | Positive generator pole grounded; output negative relative to earth | Not a documented selectable polarity |
| Measurement current | Maximum 2 mA +/-20% | Not an application-set threshold |
| Standard upper range | `(test voltage / 500 V) * 200 Gohm` | Not a fixed 200 Gohm at every voltage |
| Standard range examples | 50 V: 50 kohm-20 Gohm; 100 V: 100 kohm-40 Gohm; 250 V: 250 kohm-100 Gohm; 500 V: 500 kohm-200 Gohm | Do not extrapolate undocumented lower-range behavior |
| Standard accuracy | +/- (1.5% of reading + 1 digit); 2,000-digit display | No universal fixed ohm resolution across ranges specified |
| Thresholds | Standard specification: 50 kohm-200 Gohm; XS-20 extends the ceiling to 2 Tohm. Setup gives a 0 kohm minimum instead (V5). | `LLIM`/`HLIM` NR3, in ohms |
| Hold | 0 for continuous measurement; otherwise 1-999 s, 1 s steps | `HTIM <NR1>` seconds |
| Rise/fall | Locally 0.1-0.9 s, 1-999 s, or 1-500 V/s | `RTIM`/`FTIM` integer time arguments; fractional and V/s syntax unresolved (V6) |

Sources: [M pp. 22-23, 50-53, 123-124]. Threshold setting range and valid
measurement range are different concepts.

XS-20 extends the upper range to 2 Tohm at 500 V; M gives voltage-dependent
ranges and option-specific accuracies. At 50/100/250/500 V, the optional maxima
are 200/400/1,000/2,000 Gohm. For DXS, accuracy with this option is
+/- (2% + 1 digit) at up to 200 V, and
+/- ((1% * test voltage / 100) + 1 digit) above 200 V, using M's expression.
XS-26 instead permits 20-1,000 VDC in 1 V steps. These capabilities are
conditional; do not infer combined range/accuracy behavior for multiple
options that M does not specify. [M pp. 22, 35, 50]

## Threshold and filter semantics

With both thresholds active, PASS requires resistance strictly greater than
low and less than high. High must exceed low. In the basic version, setting
high to 200 Gohm disables the upper comparison; M describes selecting the
maximum resistance to disable that comparison, with a 2 Tohm maximum for
XS-20. These are threshold settings, not proof that the maximum is measurable
at every voltage. Do not assume zero disables the low comparison: its
documented minimum conflicts (V5).
[M pp. 23, 50-51]

`FILT NOR` selects normal operation; `CAP` stabilizes capacitive insulation
readings. M specifies CAP range 1 Mohm-200 Gohm (2 Tohm with XS-20), normal
accuracy with an additional +/-100 kohm term, and input impedance
10 Mohm +/-1%. The RHT description concerns DC dielectric withstand timing;
an equivalent insulation-specific timing guarantee is not established.
[M pp. 23, 42, 125]

For capacitive specimens, M advises increasing rather than decreasing test
voltage between tests because dielectric hysteresis/polarization can distort
readings. Measurement duration matters to the interpretation of the resistance.
This is operating guidance for qualified personnel, not permission for
software to schedule additional tests autonomously. [M p. 134]

## Required configuration and persistence

**Requirement:** deliberately select the parameter set and set voltage, both
thresholds including the intended upper-comparison disable state, rise/hold/fall
settings, and filter mode. Use only a verified timing representation.
Explicitly distinguish continuous hold from a finite test.

There are ten memories 0-9; modifications are automatically stored. Do not
rely on a previous operator's values or promise their automatic restoration.
Access locks and option-04 sequences must be accounted for before a
single-function test. [M pp. 43, 47, 53-54]

## State-aware workflow

This is a logical requirement, not a verified byte transcript. All messages
obey [serial communication](serial-communication.md).

1. Establish the manufacturer's physical prerequisites and exclusive serial
   ownership. Establish known startup context, send `REM`, identify before
   selecting a function, and enable `SRQ`. Remote re-entry remains V3.
   [M pp. 37-38, 121-123, 126, 131]
2. While idle, select `CONF` to set the applicable filter, then `QUIT`.
   Select `MEG` and `PAR`. Set voltage, thresholds, and times explicitly.
   Maintain high > low; if `CBHLIM` resets the old low value, subsequently
   set the intended final low threshold. Detect configuration errors before
   start rather than assuming XON proves acceptance.
   [M pp. 50-54, 120-125]
3. Confirm the intended single-function configuration and valid interlock/error
   state, then send `MEAS` once. The measurement uses the active memory.
   [M pp. 38, 47, 54, 122, 126]
4. Process events/status without treating Z as success. For finite hold,
   establish normal completion and valid verdict; for continuous hold, do not
   expect automatic completion. On error, interlock change, timeout, or abort,
   follow recovery instead of reusing stale status.
   [M pp. 23, 51, 120-122, 126]
5. Acquire `MEAS?` in context, retaining the separate verdict. M describes
   the last value held on the display after timed completion; it does not
   completely define post-stop serial availability. Continuous measurements
   are current values, not final completed-test results.
   [M pp. 51, 54-55, 126; V9/V10]
6. Apply `STOP`, `QUIT`, and `GTL` as appropriate to the known state. They
   do not establish discharge completion or permission to disconnect.
   Follow the manufacturer's physical discharge procedure.
   [M pp. 121, 126, 134]

## Results and limitations

Illustrative payload shape only, not a captured or validated device session:

```text
OHM <scientific-resistance>
```

The value is in ohms; Mohm conversion is a display concern, not protocol
scaling. M's single-function example does not establish a complete grammar for
under-range, over-range, charging, aborts, or unavailable values. Do not invent
sentinels or convert them into zero/infinity. [M pp. 55, 124, 126-127]

Preserve measurement value, test duration/configuration context, and verdict as
distinct information. A numeric OHM value alone does not prove a valid completed
test. See [status and errors](status-and-errors.md) and
[safety and recovery](safety-and-recovery.md).
