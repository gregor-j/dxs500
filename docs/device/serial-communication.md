# Direct local RS232 communication

Source **M**: SEFELEC, *Operating manual XS SERIES - English*, PENT0438,
v0.55; references use printed pages. See the [verification register](README.md#verification-register).

## Port and electrical setup

The instrument must have `INTERFACE : RS232` selected. It uses a rear nine-pin
connector and a CO179 cable. M lists RXD, TXD, DTR, DSR, and signal ground.
The supplied PDF does not show usable wiring in the cable figure. Do not infer
a straight-through/null-modem pinout or a DTR/DSR assertion policy. [M p. 119; V2]

| Setting | Manufacturer statement |
| --- | --- |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |
| Selectable speeds | Printed as 9.2, 19.2, 38.4, 57.6, 115.2 kbaud |

Source: [M p. 119]. The first speed is not an established 9,600 baud value.
Use a verified instrument setting; the specification does not choose a default
speed or software/hardware flow-control combination.

**Requirement:** accept a deployment-supplied local serial-device selector and
verified communication settings. Do not hard-code a production path. The
execution identity needs read/write permission and exclusive ownership.
No other process may read or command the same serial interface concurrently.
These are host-side requirements, not additional instrument commands.

## Outgoing messages

| Property | Contract | Source |
| --- | --- | --- |
| Command representation | Text command names and numeric syntax shown below; use the ASCII subset for these messages | M pp. 120-121; ASCII restriction is a project requirement |
| End of message | LF, byte `0x0A` | M p. 120 |
| Commands within one message | Separated by `;` or `:` | M p. 120 |
| Case | Uppercase and lowercase commands accepted | M p. 120 |
| Message bounds | At most 100 characters and eight commands; wording does not define whether LF counts toward 100 | M p. 120 |
| NR1 | Signed integer format | M p. 121 |
| NR2 | Signed decimal format | M p. 121 |
| NR3 | Scientific notation with decimal mantissa and signed exponent | M p. 121 |

**Requirement:** keep outgoing messages below the ambiguous character boundary.
Use each command's documented numeric format; general recognition of NR2 does
not authorize decimal arguments to an NR1 command. Expanded command spellings
appear in parentheses in M, but undocumented abbreviations are not implied.
[M p. 121]

Illustrative syntax only, not a captured session:

```text
REM<LF>
HIP:PAR 1<LF>
```

`<LF>` denotes one byte, not literal angle-bracket text. This example does not
show replies, timing, complete configuration, or permission to start a test.

## Acknowledgements, replies, and events

Completion of processing a message is marked by **XON `0x11`**. Wait for that
acknowledgement before sending the next message. The initial `REM` is sent
without first waiting for XON. Missing LF prevents processing; sending another
message while the preceding one is being handled can cause erratic operation
or a hang. [M pp. 120, 131]

This XON is application-level synchronization evidence. M does not document
XOFF or establish conventional XON/XOFF flow control. **Requirement:** the
serial layer must not consume the acknowledgement invisibly as flow control.
Do not enable such behavior merely because M calls the byte XON.

After `SRQ`, the instrument can emit **`Z`** for events including test end,
format error, or open interlock. Inspect `*STB?` and, when needed, `*ESR?` to
interpret the event. A `Z` is neither XON nor proof of a passed test. [M
pp. 120, 126]

**Unverified (V1):** the ordinary `*IDN?`, register, and single-function
`MEAS?` reply terminators are not unambiguously specified. The CR rule on
p. 127 is in the sequence-result subsection; it is not evidence that every
ordinary reply ends in CR. Exact ordering of reply data, XON, and Z, event
interleaving, and multiple-query boundaries remain unresolved. Do not promote
an application log's stripped string into a byte-framing contract.

**Requirement:** distinguish acknowledgement, query payload, and unsolicited
event without discarding bytes belonging to another response. Do not send a
new message merely because a payload or Z arrived. Do not clear the receive
buffer to hide unsynchronized traffic or assume a fixed delay restores
synchronization. Use bounded waits, but leave the exact framing algorithm
blocked until V1 is resolved.

## Remote lifecycle and contexts

1. Establish exclusive port access and known instrument context. `REM` must
   be the first command. M restricts it to the startup screen immediately
   after power-on. Do not automatically power-cycle to satisfy this condition.
   [M pp. 121, 131]
2. After remote entry and the corresponding acknowledgement, identification
   belongs before function selection. `*IDN?` has four comma-separated fields:
   manufacturer, unit reference, serial-number field, software revision.
   M's family examples are not literal DXS500 responses and do not justify
   requiring a constant serial number or revision. [M p. 122]
3. Send `SRQ` after `REM` when using event notification. `HIP`, `MEG`, or
   `CONF` selects the relevant context from initialization. [M pp. 123, 126]
4. `QUIT` exits the current function; `GTL` returns local. `LLO` locks return
   to local and is not required by the measurement workflows. [M pp. 121, 126]
5. `*RST` nearly resets the unit and returns it local; M requires another
   `REM` before following commands. It does not specify complete state
   preservation or reconcile this with the startup-only restriction.
   [M p. 121; V3]

Re-entry after `GTL`, a port close, a failed exchange, or a reconnect is not
guaranteed. Opening a host serial port is not proof of initialization context.
The context summary permits `*IDN?` broadly, but its dedicated instruction
restricts it to before function selection; follow that narrower instruction
until clarified. [M pp. 122, 127]

## Communication outcomes

**Requirement:** distinguish connection/open failure, wait expiration with no
response, incomplete response, malformed response, unexpected traffic, and loss
of synchronization. Preserve enough diagnostic evidence to identify which
message has an uncertain outcome, without publishing deployment identifiers.

Bound connection, acknowledgement, query, and measurement waits separately.
Chosen limits are client policy, not manufacturer-specified deadlines.
Measurement duration depends on timing mode; an unlimited/continuous mode
cannot be assigned a fabricated manufacturer completion deadline.
[M pp. 17, 23, 64]

A timed-out write/start/configuration exchange may have taken effect.
Do not blindly retry `MEAS` or configuration, declare a failed specimen, or
assume the output is off. Follow [safety and recovery](safety-and-recovery.md).
