# okta-jml-lifecycle
----
I built an automated joiner-mover-leaver identity lifecycle in a free Okta
org, modelled on a fictional bank (Meridian Trust Bank). Access is never
assigned to people: apps bind to groups, group rules place people in groups
from their HR attributes, and the lifecycle runs itself. The moment that
sold me on identity engineering: I changed one department field and watched
payment-system access revoke itself, with an audit log entry to prove it.

----
## What this demonstrates
- Role-based access via groups and group-bound application assignment
- Attribute-driven group rules (HR-driven provisioning pattern)
- Full JML lifecycle: automated provisioning, mover access swap, leaver
deactivation with session kill
- MFA enrollment policy and hardened password policy
- Least-privilege delegated administration
- Audit evidence for every lifecycle event from the Okta System Log

----
## The lifecycle, evidenced
- [Joiner](evidence/01-joiner): user created with role attributes, correct
access appeared with zero manual assignment
- [Mover](evidence/02-mover): one attribute change, old access revoked and
new access granted automatically
- [Leaver](evidence/03-leaver): one deactivation, sessions killed, all
access removed, timestamped log entry
- [Hardening](evidence/04-hardening): MFA enforcement, password policy,
scoped admin role

----
## Operations
See [runbook.md](runbook.md) for the full operational guide: how each lifecycle event runs, what is automated versus manual, and what evidence each step produces.

## Honest framing
Training build in a free Okta Integrator org on a fictional scenario.
Bookmark apps stand in for federated apps: assignment logic is identical,
SAML/SCIM federation requires control of the target application. In
production, the attribute changes I made by hand arrive from the HR system.
The automation, policies, and audit trail are real Okta

----

Built by: Shaibu Fuseini   [shaibu-fuseini-5b5244247](linkedin.com)


