# AC Hipot

Source **M**: SEFELEC, *Operating manual XS SERIES - English*, PENT0438,
v0.55; printed pages. This is the standard DXS500 500 VA AC function, not
DC Hipot of an XS506 or a presumed installed option. [M pp. 1, 15, 35]

## Capabilities and settings

| Quantity | Manufacturer contract | Serial control / limitation |
| --- | --- | --- |
| Output | 100-5,000 VAC, 50 or 60 Hz | `ACV <NR1>` volts; command description instead starts at 10 V (V4). No documented remote frequency setter (V6). |
| Output accuracy | +/- (3% + 50 V), at 100-5,000 V, current below 1 mA, Delta-I/IMAX/combined detection | Do not apply this accuracy outside those conditions |
| Voltage readout | +/- (1.5% + 20 V) of reading | Result `VOLT` is in volts |
| IMAX | 0.1-110 mA, 0.1 mA steps | `HLIM <NR3>` amperes; setup chapter says maximum 99.9 mA (V4) |
| IMIN | 0-109 mA; zero disables lower comparison | `LLIM <NR3>` amperes; setup maximum conflicts (V4); independent IMIN resolution not explicitly stated |
| Delta-I | 10-100 mA per 10 microseconds, 10 mA steps; amplitude +/-10%, pulse width +/-20% | `IDE`; serial scaling not explicit (V7) |
| Current readout | +/- (2.5% + 2 digits), one digit = 0.1 mA; 999-point display | `AMP` result is in amperes; display precision is not a full serial grammar |
| Rise/fall in AUTO | 0-1 s in 0.1 s steps, then 1-999 s in 1 s steps | `RTIM`/`FTIM` use NR1, not established fractional syntax |
| Hold in AUTO | 1-999 s in 1 s steps | `HTIM <NR1>` seconds |

Sources: [M pp. 15-17, 59, 62-64, 123-126]. AC voltage setting resolution
beyond the serial integer argument is not separately established. Conflicting
limits are not silently reduced to an invented supported range.

The maximum-setting short-circuit current exceeds 200 mA AC. IMAX detection
can take 200-300 ms; fast IMAX is specified at 20 ms, with AC detection accuracy
+/- (10% + 2 units) in M's notation. These modes are not instantaneous
current-limiting safety guarantees. Above 110 mA AC, M describes a strong-current
display indication, not a complete serial overflow format. [M pp. 15-17]

## Detection, timing, and filtering

`DET I` selects IMAX, `FI` fast IMAX, `DELTA` current variation, and the
corresponding `+DELTA` tokens combine mechanisms. `OFF` removes current
monitoring and load-based voltage adjustment; it must not be selected
accidentally as a fallback. With IMIN active, PASS requires current strictly
above IMIN and below IMAX. IMIN must be below IMAX; zero disables IMIN.
With IMIN disabled, current at or above IMAX fails. [M pp. 16, 62, 126]

`TIM AUT` uses rise/hold/fall. `TIM FAIL` selects DEFAULT, an untimed mode
ending on breakdown or operator stop rather than an automatic hold deadline.
`TIM UDIV2` uses AUTO starting at half the selected voltage. Local MANUAL mode
permits operator voltage adjustment, but no serial `TIM MAN` is documented.
[M pp. 17, 64, 125]

Expert ramp detection (`RDET`) is common to rise and fall. `IMRA` and `IDERA`
control ramp thresholds; `UDISP`/`IDISP` control ramp displays. Software IMAX
and IMAX+Delta-I ramp modes require current display and rise/fall times at
least 2 s. Turning displays off can improve ramp linearity, so display choices
are not purely cosmetic. Ramp threshold scaling and IDERA/IDRA spelling remain
V7. [M pp. 66-67, 126, 128]

The Setup filter has NOR/CAP/RHT settings. M recommends CAP for capacitive
Hipot specimens on p. 68, but describes its voltage-regulation/discharge and
RHT residual-voltage behavior specifically for **DC** on p. 42. Do not assign
those DC guarantees to AC operation; AC effects require verification.
`M335` changes power mode associated with EN 60335/60204, not a test-installation
compliance certificate. [M pp. 42, 67-68, 125]

## Required configuration and persistence

**Requirement:** deliberately choose the stored parameter set, AC voltage,
thresholds and their enabled states, timing mode and applicable times,
detection mode and applicable Delta-I threshold, filter mode, and applicable
expert ramp/display/power settings. Confirm frequency and any setting without
a documented setter by an approved process; absence of a setter is not a
default. Settings whose serial contract remains unverified block the affected
configuration, rather than receiving guessed values.

Ten parameter sets, 0-9, are documented; modifications are automatically stored.
No automatic restoration is promised. Parameter access locks, an installed
sequence mode, accessories, and options may prevent the intended configuration.
Do not assume reset clears them. [M pp. 43, 47, 65-68]

## State-aware workflow

This is a required logical workflow, not a validated byte transcript. Each
message follows [serial synchronization](serial-communication.md).

1. Establish physical safety prerequisites and exclusive serial access.
   Establish startup/remote context; send initial `REM`, identify with
   `*IDN?` before function selection, and enable `SRQ`. Stop on uncertain
   identity/context instead of assuming repeated REM is valid.
   [M pp. 37-38, 121-123, 126, 131]
2. While idle, enter `CONF` for applicable global settings, then `QUIT`.
   Select `HIP`, deliberately select `PAR`, and set the required test
   parameters. Preserve high > low ordering; `CBHLIM` can reset the old
   low threshold before a new high is set, followed by the intended `LLIM`.
   Check for dialog/configuration errors before starting.
   [M pp. 62, 65-67, 123-126]
3. Confirm intended single-function mode, interlock and error state; a closed
   interlock alone is not full safety clearance. Send `MEAS` once.
   [M pp. 38, 47, 122, 126]
4. Handle XON, Z, status, and ESR as distinct evidence. Continue while
   running; handle errors/open interlock explicitly. Establish current-test
   completion and verdict using [status rules](status-and-errors.md), not
   merely the presence or absence of PASS. [M pp. 120-122]
5. Acquire `MEAS?` in the function context before cleanup that might destroy
   result state. M returns current values and does not specify serial
   post-STOP retention; completed-result availability is V9/V10.
   [M pp. 126-127]
6. Stop/exit/release through `STOP`, `QUIT`, and `GTL` as applicable to the
   known state. Failure at a step must be reported, not converted into
   successful release. Do not infer safe discharge from acknowledgements.
   [M pp. 121, 126; recovery requirements below]

For fault or abort paths, prioritize [safety and recovery](safety-and-recovery.md)
instead of continuing ordinary result acquisition or automatically restarting.

## Result meaning

Illustrative payload shape, not a captured or validated session:

```text
VOLT <scientific-voltage> AMP <scientific-current>
```

The quantities are volts and amperes; conversion to kV/mA is presentation, not
wire syntax. Preserve both values and the separately established verdict.
M's examples do not fix leading whitespace, precision, exponent width,
overflow strings, or general response termination. [M pp. 124, 126-127]

For breakdown, M describes HV cutoff at the next control-wave zero crossing
and storage of breakdown voltage/current. For normal AUTO completion, voltage
is cut off after fall time and comparison determines the verdict. The exact
serial post-completion snapshot/retention is not fully defined. A low-current
failure is possible without breakdown, and a numeric result is not proof of
PASS. [M pp. 69-71]
