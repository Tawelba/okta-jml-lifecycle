# Identity Lifecycle Runbook: Meridian Trust Bank

## 1. Purpose
This document describes how Meridian Trust Bank's joiner-mover-leaver identity lifecycle operates in Okta, what is automated versus manual, and what evidence each step produces. It is written for the IT Operations team and any auditor reviewing our identity controls..
It covers the three main lifecycle events:
- Joiners: new employees entering the organisation
- Movers: existing employees changing departments or roles
- Leavers: employees leaving the organisation

The objective is to ensure that users receive the correct access when they join, lose access that is no longer required when they move, and have all access removed promptly when they leave.
This implementation was completed in an Okta Integrator Free Plan environment using a fictional banking organisation.

## 2. Scope
This runbook applies to the following Okta-managed resources:

### Departments
- Lending
- Payments

### Okta groups
- `Dept-Lending`
- `Dept-Payments`
- `All-Staff`
- `Admins-Helpdesk`

### Applications
- Meridian Intranet
- OriginateCloud
- PaySuite
- 
### Administrative roles
- Super Administrator
- Help Desk Administrator

The applications used in this lab are Okta Bookmark Apps. They represent business applications and demonstrate the same group-assignment logic that would be used with federated SAML, OIDC, or SCIM-integrated applications.

---
## 3. Roles and Responsibilities

 |Role                     |                                  Responsibility                                             |
 |-------------------------|---------------------------------------------------------------------------------------------|
 |HR or authorised manager | Confirms the employee’s start date, department change, or departure                         |
 |IAM administrator        | Creates, updates, deactivates, and reviews user identities                                  |
 |Help Desk administrator  | Assists users with password reset and authentication-factor issues                          |
 |Super Administrator      | Maintains Okta configuration, policies, group rules, applications, and administrative roles |
 |Line manager             | Confirms that the user’s department and required access are correct                         |
 |Security or audit team   | Reviews Okta System Log evidence and lifecycle-control effectiveness                        |
 |-------------------------|---------------------------------------------------------------------------------------------|

In this lab, the IAM administrator performs the HR-triggered profile changes manually. In a production environment, the changes would normally originate from an authoritative HR system.
---

## 4. Design Principles

### 4.1 Group-based application assignment
Applications are assigned to groups, never to individual users. People flow in and out of groups; the group-to-app wiring never changes. This makes the directory readable, auditable, and scalable.

### 4.2 Attribute-driven group membership
Group membership is computed from identity attributes (department) via group rules, not clicked by admins. When HR updates a user's department, access changes automatically.

### 4.3 Least privilege
Users receive only the access required for their current department. 
When a user changes department, the previous department access must be removed rather than accumulated.
Administrative access also follows least privilege. Help Desk personnel receive only the Help Desk Administrator role and must not receive Super Administrator permissions unless formally authorised.

---

## 5. Joiner Procedure
### 5.1 Trigger
HR creates the user in the HR system (or manually in Okta during onboarding).

#### 5.2 Who creates the user
Production: HR system pushes the user via SCIM/API.
Lab: Admin creates the user manually in Directory → People → Add person.

### 5.3 Required attributes
First name, last name
Username / email
Department (exact spelling: Lending or Payments)
Password (admin-set in lab; activation email in production)

### 5.4 What happens automatically
Rule-AllStaff fires: user joins All-Staff group (birthright access).
Rule-Lending or Rule-Payments fires: user joins the appropriate department group.
App assignments appear automatically:
All-Staff → Meridian Intranet
Dept-Lending → OriginateCloud
Dept-Payments → PaySuite

#### 5.5 Evidence produced
Screenshot of user's dashboard showing correct tiles
Screenshot of user's group memberships page (rule-managed)

`evidence/01-joiner/`

---
## 6. Mover Procedure

### 6.1 Trigger
The mover process begins when HR or an authorised manager confirms that an employee has changed department or role.

### 6.2 Risk addressed
The mover process is designed to prevent privilege creep.
Privilege creep occurs when users receive access for a new role but retain access from previous roles that is no longer required.
The expected outcome is a replacement of role access, not an accumulation of access.

### 6.3 The single attribute change
Change the user's Department attribute from one value to another (e.g., Lending → Payments).

### 6.4 What revokes and grants automatically
Rule-Lending revokes: user leaves Dept-Lending group → OriginateCloud access removed.
Rule-Payments grants: user joins Dept-Payments group → PaySuite access granted.
Rule-AllStaff remains: user stays in All-Staff → Meridian Intranet access preserved.
Critical: This is a reset-not-accumulate pattern. Old access is revoked automatically — this is the step that prevents privilege creep.

### 6.5 Evidence produced
Before/after screenshots of user's dashboard
Screenshot of group memberships page showing swap

`evidence/02-mover/`

---

## 7. Leaver Procedure
### 7.1 Trigger
HR marks the user as terminated in the HR system (or admin initiates offboarding in Okta).

### 7.2 Action
Deactivate the user in Directory → People → [User] → More Actions → Deactivate.
Why deactivate, not delete or suspend:
Deactivate is terminal: access dies, sessions end, but the account and audit trail remain. This is the correct action for termination.
Suspend is reversible: suitable for sabbatical, parental leave, or investigation — not for leavers.
Delete destroys the audit trail, which is a compliance finding.

### 7.3 What dies automatically
All active sessions are killed immediately.
All app assignments are revoked.
User cannot log in.
Account remains in directory with Deactivated status for audit history.

### 7.4 Evidence produced
Screenshot of failed login attempt
Screenshot of user's profile showing Deactivated status
- The System Log records the administrator, user, action, and timestamp

### 7.5 Validation

The IAM administrator must verify:

- The account status is Deactivated
- Login attempts fail
- Active application access has been removed
- The user’s audit history remains available
- The System Log contains the deactivation event
- The action occurred within the organisation’s required offboarding timeframe

`evidence/03-leaver/`

---

## 8. Authentication Policy

### 8.1 MFA enrolment

Okta Verify is enabled as an authenticator.
Users are required to enrol in MFA when accessing the environment.
MFA is enforced at the identity-provider level so that authentication controls can be applied consistently across all connected applications.

The IAM administrator must periodically verify that:
- Okta Verify remains active
- The enrolment policy remains enabled
- Required users cannot bypass enrolment
- Authentication-policy exceptions are approved and documented

### 8.2 Password policy

The configured password policy includes:
- Minimum password length of at least 12 characters
- Uppercase character requirement
- Lowercase character requirement
- Numeric character requirement
- Common-password checking
- Account lockout after 10 failed attempts

### 8.3 Password expiration

Routine password expiration is not enabled.

Passwords should instead be changed when:

- Compromise is suspected
- A password appears in a known breach
- The user requests a reset
- An administrator requires a reset following a security event
- Organisational policy or legal requirements demand it

The environment relies on password length, blocked common passwords, account lockout, and MFA rather than frequent forced password changes.

### 8.4 Authentication evidence

Capture and retain:

- MFA enrolment-policy screenshot
- Password-policy screenshot
- MFA enrolment prompt
- Relevant System Log events

Store evidence in:

`evidence/04-hardening/`

---

## 9. Administrative Access Model

### 9.1 Super Administrator

The Super Administrator role is restricted to authorised identity or security administrators.

Super Administrators can:

- Configure applications
- Create and modify group rules
- Modify authentication policies
- Assign administrative roles
- Manage users and groups
- Access security and audit settings

Because this role provides broad control over the identity environment, it must be assigned only where necessary.

Super Administrator accounts must:

- Use MFA
- Use strong, unique passwords
- Be reviewed periodically
- Not be shared
- Be removed promptly when no longer required

### 9.2 Help Desk Administrator

Help Desk staff receive the Help Desk Administrator role.

This role may be used to:
- Search for users
- Reset passwords
- Reset authentication factors
- Assist with account-recovery issues

Help Desk administrators must not be able to:
- Modify group rules
- Configure applications
- Change security policies
- Assign powerful administrative roles
- Modify lifecycle automation

### 9.3 Administrative review

Administrative-role assignments should be reviewed regularly.
The review should confirm:
- Each administrator still requires the role
- The role remains appropriate for the person’s duties
- No unnecessary Super Administrator assignments exist
- Departed administrators have been removed
- MFA is active for all administrators

---

## 10. Evidence Retention and Audit Review

The Okta System Log is the primary source of lifecycle evidence.

For each lifecycle event, evidence should include:

- Event type
- Actor
- Target user
- Timestamp
- Result
- Group changes
- Application changes
- Account-status changes

Recommended repository structure:
```text
evidence/
├── 01-joiner/
├── 02-mover/
├── 03-leaver/
└── 04-hardening/
