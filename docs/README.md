# DXS500 device specification

This documentation specifies device behavior for a future local serial library.
It does not describe a completed or hardware-validated implementation. The scope
is DXS500 AC dielectric withstand testing and insulation-resistance measurement.

## Evidence conventions

The primary source is SEFELEC, *Operating manual XS SERIES - English*,
reference PENT0438, revision v0.55. References must use printed page numbers;
PDF page indexes must be identified separately.

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
