# Status, events, and errors

Source **M**: SEFELEC, *Operating manual XS SERIES - English*, PENT0438,
v0.55; printed pages. Transport framing remains subject to
[V1 and V8](README.md#verification-register).

## Register payloads

The register queries return hexadecimal values with a `#H` prefix. M uses a
two-digit example; it does not define every permissible lexical variation.
Do not strip arbitrary characters until malformed traffic looks valid.
[M p. 122; parsing restriction is a project requirement]

| STB bit | Zero | One |
| --- | --- | --- |
| 0 | Interlock open | Interlock closed |
| 1 | No instrument error indicated | Instrument error, e.g. incorrect voltage |
| 2 | End of test/not running indication | Test running |
| 3 | FAIL indication | PASS indication |
| 4 | Unused | Unused |
| 5 | No ESR summary | Logical OR of ESR bits |
| 6 | No STB summary | Logical OR of STB bits, as printed |
| 7 | Unused | Unused |

Source: [M p. 122]. Bit 6's apparent self-reference and its interaction with
masks are not fully explained. M does not establish pass-bit validity before
a test, during running, after STOP, or during instrument errors. It does not
say that reading STB clears it.

| ESR bit | Meaning when set |
| --- | --- |
| 0-3 | Unused |
| 4 | Dialog error type 2: out-of-range numeric value or command in wrong context |
| 5 | Dialog error type 1: incorrect command |
| 6 | Unused |
| 7 | Power-on event |

**Reading `*ESR?` clears its bits.** Retain that observation before another
read; repeated diagnostics must not erase the only evidence. `*ESE?` returns
the ESR mask; `*SRE?` returns the STB/service-request mask. Corresponding setters
select enabled causes; do not assume power-on mask values. `*CLS` resets
standard registers, not necessarily every device state. [M pp. 121-122]

## Event interpretation

After `SRQ`, Z can announce test end, dialog error, open interlock, or other
events. It contains no reason code or successful-test guarantee. XON marks
message-processing completion instead. A query payload is a third kind of
traffic. None can substitute for the others. [M pp. 120, 126]

**Requirement:** maintain the relationship to the current start request and
previous status observations. A useful decision sequence is:

1. Resolve the start exchange without assuming XON means the specimen passed.
2. Interpret Z as a reason to inspect state, not as the end condition itself.
   Status polling can also observe state; no polling period is prescribed.
3. Handle instrument error, interlock opening, dialog error, or lost
   communication explicitly. Do not wait only for a PASS bit.
4. While bit 2 is set, treat the test as running; bit 3 alone cannot complete it.
5. A not-running indication must relate to the current test, not an idle or
   stale register. If start acceptance/completion cannot be established, retain
   an uncertain outcome rather than claiming success.
6. For an established normally completed test without invalidating errors,
   interpret bit 3 as the verdict and acquire the measurement. Numeric
   acquisition failure does not retroactively turn it into a specimen failure.

These are conservative project requirements based on M pp. 120-122, 126.
Exact transitions and verdict validity still require V8 verification, especially
when a fast test completes before running is observed. Requiring observation
of a running bit on every test is not a manufacturer rule.

## Distinct outcomes

| Outcome | Meaning and required treatment |
| --- | --- |
| Running | Current test still active; no final verdict yet |
| Normally completed, PASS | Completion and valid current-test verdict established |
| Normally completed, FAIL | Specimen failed configured criteria; not inherently a transport exception |
| Instrument error | Instrument cannot establish the intended test conditions; do not report ordinary completed PASS/FAIL alone |
| Open interlock | Physical prerequisite not satisfied; do not restart automatically |
| Aborted | Requested or external stop; do not reuse an old PASS bit as a completed result |
| Invalid/incomplete response | Protocol evidence insufficient; preserve the distinction from a device-reported fault |
| Communication failure/uncertain state | Command may have executed; output and test outcome unknown |

This classification is a project contract, not an invented additional device
register. See [recovery](safety-and-recovery.md).

## Display errors versus serial evidence

M describes `DIALOG ERROR : 1` for an unrecognized command and
`DIALOG ERROR : 2` for context/range errors as **display messages**. ESR gives
serial diagnostic bits; do not require the display strings as serial replies.
[M pp. 120, 122]

Insulation display states include `OVER-RANGE`, `UNDER-RANGE`, `CHARGING`,
`SAFETY LOOP`, and `BOARD NOT READY`. Hipot includes `VOLTAGE ERROR`, `I<IMIN`,
`SYNCHRO ERROR`, and `BOARD NOT READY`; overload indication is also documented.
Their full single-function serial encodings are not specified.
[M pp. 17, 55, 71; V10]

An out-of-range resistance must not be fabricated as a finite maximum, zero,
or a pass. A low-current Hipot failure can indicate a bad specimen connection
without breakdown. A numeric result does not by itself encode either fault.
[M pp. 55, 71, 126; handling is a project requirement]
