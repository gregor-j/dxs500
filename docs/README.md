# DXS500 device specification

This documentation specifies device behavior for a future local serial library.
It does not describe a completed or hardware-validated implementation. The scope
is DXS500 AC dielectric withstand testing and insulation-resistance measurement.

## Reading order

1. [Device scope, sources, and verification register](device/README.md)
2. [Safety and recovery](device/safety-and-recovery.md)
3. [Serial communication](device/serial-communication.md)
4. [Commands](device/commands.md) and [status and errors](device/status-and-errors.md)
5. [AC Hipot](device/hipot.md) and [insulation resistance](device/insulation-resistance.md)
6. [Deployment compatibility](deployment/README.md)

Use the [glossary](../GLOSSARY.md) for domain terminology. The documents specify
device contracts, not a public API, software architecture, CLI, or packaging
implementation. Other instruments and network transport are outside the target.

## Evidence conventions

The primary source is SEFELEC, *Operating manual XS SERIES - English*,
reference PENT0438, revision v0.55. References must use printed page numbers;
PDF page indexes must be identified separately.

Each device document uses **[M p. N]** or **[M pp. N-M]** for that same source.
In the supplied 156-page PDF, one-based PDF page index equals printed page
number plus one; the cover is PDF page 1. References identify evidence rather
than reproducing the manual. The manufacturer PDF is not distributed here.

**Manufacturer:** a statement supported by M. **Requirement:** a project
constraint for the future implementation, not a manufacturer guarantee.
**Observation:** limited evidence from an identified inspection.
**Unverified:** missing or contradictory evidence, tracked as V1-V10 in the
[verification register](device/README.md#verification-register).

Distinguish manufacturer-specified behavior, observed runtime constraints, and
unverified behavior. Supplementary summaries and previous implementations are
discovery aids, not authoritative protocol specifications.

Document supported behavior without inventing missing contracts. Record
manufacturer contradictions and missing details as verification blockers for
the affected implementation capability. Do not select an undocumented default,
numeric limit, response delimiter, or event ordering to make the specification
appear complete.

In particular, ordinary RS232 query-response termination and ordering relative
to XON and asynchronous events require clarification (pp. 119–120, 126–127).
Conflicting AC voltage and current limits also require clarification
(pp. 15–16, 62, 123). Until these blockers are resolved, the specification must
not be represented as fully implementation-ready.

Active device communication and measurements are outside this documentation
work. Separately approved passive observation and controlled direct serial
reproduction are tracked in [issue #4](https://github.com/gregor-j/dxs500/issues/4).
That planned work is not completed verification and does not block documenting
known behavior and remaining gaps.

Examples must be labelled as illustrative unless they have an explicitly
identified, sanitized observation source. Observations must state their scope
and limitations; they do not establish behavior for every instrument, firmware,
or installed option. Public documents must contain no internal deployment
identifiers, source paths, or access details.

## Configuration policy

The specified measurement workflow must explicitly select a parameter set and
set every applicable setting that can affect the measurement or its verdict.
This includes detection and filter modes and the intended enabled or disabled
state of threshold comparisons. Do not rely on undocumented defaults or values
left by an earlier operator.

This is a requirement for the future implementation, not a claim that the
instrument resets these settings automatically. The manual describes ten
parameter sets per function and automatic storage of changes (pp. 53–54,
65–66). Configuration therefore changes the selected stored parameter set.
Automatic restoration of its previous contents is not promised.

Where command availability, parameter units, or a disabling value remain
unverified, the affected configuration step remains blocked under the evidence
conventions above.

## Completion and limitations

This specification captures the available source evidence, including gaps; it
does not certify the instrument, test installation, or future software. No
serial sessions or measurements were performed for this documentation. Earlier
passive runtime observations are isolated in the deployment document. Existing
application logs are not treated as raw serial evidence.

Source traceability, DXS500 applicability, consistency, local links, and
publication sanitization are the documentation checks. Hardware validation,
option discovery, exact response framing, and resolution of manufacturer
contradictions remain separate work.
