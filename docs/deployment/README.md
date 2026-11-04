# Local deployment compatibility

## Observed environment

The following anonymized observations are supplied by
[issue #3](https://github.com/gregor-j/dxs500/issues/3) from one previously
inspected target environment. They were not newly measured while writing this
specification and are not universal properties of every deployment.

| Area | Observed constraint |
| --- | --- |
| Platform | Linux x86_64 kiosk environment; glibc 2.36 observed |
| Interpreter | Python 2.7.16; no Python 3 executable found in inspected standard locations |
| Serial dependency | pyserial 3.4 installed |
| Packaging | setuptools 22.0.5 present; pip and virtual-environment tooling not found in inspected runtime locations |
| Bootstrap | ensurepip present, but its version probe emits missing-hash errors; usable dependency installation not established |
| TLS | Python SSL import fails because a required shared library is unavailable |
| System installation | No conventional system package manager found in inspected locations |
| Serial access | Appropriate device permissions and exclusive access required; production execution need not be root |

The underlying runtime inspection was passive. It did not establish physical
device identity, actual serial communication, or measurement behavior. Later
application-log observations do not constitute direct serial verification.

## Requirements for the future library

Support Python **2.7.16** and **pyserial 3.4**, including byte-level protocol
handling compatible with that runtime. Do not require Python-3-only syntax,
dependencies, or serial APIs introduced after pyserial 3.4. This is a project
compatibility constraint, not a manufacturer protocol requirement.

Use a configurable local serial-device selector, verified line settings, and
an execution identity with appropriate permissions. Enforce exclusive device
ownership operationally; a second reader can consume another client's data.
Do not publish deployment paths or prescribe root as the normal execution user.

Do not assume on-target downloads, functioning TLS, pip, virtual environments,
a system package manager, or interpreter upgrades. The eventual library is
intended for inclusion in the kiosk image. Image creation, integration,
distribution, persistence, boot infrastructure, packaging implementation, and
service changes are outside this documentation.

See [serial communication](../device/serial-communication.md) for the device
contract and [safety and recovery](../device/safety-and-recovery.md) for ownership
and uncertain-state requirements. Runtime availability is not proof that a
measurement installation is safe or a serial protocol has been validated.
