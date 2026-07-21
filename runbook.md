# Identity Lifecycle Runbook: Meridian Trust Bank

## 1. Purpose
This runbook describes how Meridian Trust Bank manages the identity lifecycle of employees in Okta.
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

 Role                                   Responsibility 

 HR or authorised manager | Confirms the employee’s start date, department change, or departure 
 IAM administrator        | Creates, updates, deactivates, and reviews user identities 
 Help Desk administrator  | Assists users with password reset and authentication-factor issues 
 Super Administrator      | Maintains Okta configuration, policies, group rules, applications, and administrative roles 
 Line manager             | Confirms that the user’s department and required access are correct 
 Security or audit team   | Reviews Okta System Log evidence and lifecycle-control effectiveness 

In this lab, the IAM administrator performs the HR-triggered profile changes manually. In a production environment, the changes would normally originate from an authoritative HR system.
---

## 4. Design Principles

### 4.1 Group-based application assignment
Applications must be assigned to groups rather than directly to individual users.

The approved mappings are:

  Application       Assigned group 
 
 Meridian Intranet | `All-Staff` 
 OriginateCloud    | `Dept-Lending` 
 PaySuite          | `Dept-Payments` 

Direct user-to-application assignments are not permitted because they are difficult to review, scale, and remove consistently.
Group-based assignment ensures that access changes automatically when a user enters or leaves a group.

### 4.2 Attribute-driven group membership
Department group membership is calculated from the user’s Okta profile.

The following rules are used:

| Rule            | Condition                    | Result                     |
|-----------------|------------------------------|----------------------------| 
| `Rule-Lending`  | Department equals `Lending`  | Add user to `Dept-Lending` |
| `Rule-Payments` | Department equals `Payments` | Add user to `Dept-Payments`|
| `Rule-AllStaff` | Department is not empty      | Add user to `All-Staff`    |

### 4.3 Least privilege
Users receive only the access required for their current department.
When a user changes department, the previous department access must be removed rather than accumulated.
Administrative access also follows least privilege. Help Desk personnel receive only the Help Desk Administrator role and must not receive Super Administrator permissions unless formally authorised.

### 4.4 Auditability
Every lifecycle action must be verifiable through the Okta System Log.

Evidence should identify:
- The action performed
- The affected user
- The administrator or system process that performed the action
- The date and time
- Group-membership changes
- Application-assignment changes
- Account-status changes

---

## 5. Joiner Procedure

### 5.1 Trigger

The joiner process begins when HR or an authorised manager confirms that a new employee has been approved to start work.

The request should include:

- First name
- Last name
- Corporate username or email address
- Department
- Job title, where applicable
- Start date
- Line manager
- Employment status

### 5.2 Preconditions

Before creating the user, the IAM administrator must verify that:

- The employee has been approved by HR
- The username is unique
- The department value is valid
- The required group rules are active
- Applications are assigned to groups rather than directly to users

### 5.3 Procedure

1. Sign in to the Okta Admin Console.

2. Navigate to:

   `Directory → People → Add person`

3. Enter the employee’s identity information.

4. Configure the username using the approved corporate naming convention.

5. Set the employee’s Department field to the correct approved value, such as:

   - `Lending`
   - `Payments`

6. Save the user.

7. Allow the active group rules to evaluate the profile.

8. Confirm that the user has been added automatically to:

   - `All-Staff`
   - The appropriate department group

9. Confirm that the correct applications have been assigned automatically.

10. Ask the user to complete account activation and MFA enrolment.

### 5.4 Expected automated outcome

For a Lending employee:

- The user is added to `All-Staff`
- The user is added to `Dept-Lending`
- Meridian Intranet is assigned
- OriginateCloud is assigned
- PaySuite is not assigned

For a Payments employee:

- The user is added to `All-Staff`
- The user is added to `Dept-Payments`
- Meridian Intranet is assigned
- PaySuite is assigned
- OriginateCloud is not assigned

### 5.5 Validation

The IAM administrator must verify:

- The user account is active
- The Department field is correct
- Group memberships are rule-managed
- No applications were assigned directly to the user
- The user sees only the expected application tiles
- MFA enrolment is required

### 5.6 Evidence

Capture and retain:

- User-profile screenshot
- Group-membership screenshot
- End-user dashboard showing assigned applications
- System Log events showing:
  - User creation
  - Group membership additions
  - Application assignments
  - MFA enrolment activity, where available

Store evidence in:

`evidence/01-joiner/`

---

## 6. Mover Procedure

### 6.1 Trigger

The mover process begins when HR or an authorised manager confirms that an employee has changed department or role.

The request should include:

- Employee name
- Current department
- New department
- Effective date
- Manager approval
- Any exceptional access requirements

### 6.2 Risk addressed

The mover process is designed to prevent privilege creep.

Privilege creep occurs when users receive access for a new role but retain access from previous roles that is no longer required.

The expected outcome is a replacement of role access, not an accumulation of access.

### 6.3 Procedure

1. Sign in to the Okta Admin Console.

2. Navigate to:

   `Directory → People`

3. Search for and open the employee’s profile.

4. Select **Edit**.

5. Change the Department field from the old department to the new approved department.

   Example:

   `Lending → Payments`

6. Save the profile.

7. Allow Okta group rules to re-evaluate the user.

8. Confirm that the user is removed automatically from the previous department group.

9. Confirm that the user is added automatically to the new department group.

10. Confirm that the old department application is revoked.

11. Confirm that the new department application is assigned.

12. Confirm that birthright access, such as Meridian Intranet, remains available.

### 6.4 Expected automated outcome

For a move from Lending to Payments:

- Removal from `Dept-Lending`
- Addition to `Dept-Payments`
- Removal of OriginateCloud
- Assignment of PaySuite
- Continued membership in `All-Staff`
- Continued access to Meridian Intranet

For a move from Payments to Lending:

- Removal from `Dept-Payments`
- Addition to `Dept-Lending`
- Removal of PaySuite
- Assignment of OriginateCloud
- Continued membership in `All-Staff`
- Continued access to Meridian Intranet

### 6.5 Validation

The IAM administrator must verify:

- The Department field contains the new approved value
- The user is no longer in the previous department group
- The user is in the new department group
- The previous application is no longer available
- The new application is available
- Birthright access remains unchanged
- All group memberships remain rule-managed
- No manual application assignment was introduced

### 6.6 Evidence

Capture and retain:

- Updated user profile
- Updated group memberships
- User dashboard after the move
- System Log events showing:
  - Profile update
  - Previous group membership removal
  - New group membership addition
  - Previous application revocation
  - New application assignment

Store evidence in:

`evidence/02-mover/`

---

## 7. Leaver Procedure

### 7.1 Trigger

The leaver process begins when HR confirms that an employee’s employment has ended.

The request should include:

- Employee name
- Username
- Termination date and time
- Whether the departure is immediate or scheduled
- Manager or HR approval
- Any legal-hold or investigation requirements

### 7.2 Required action

The user account must be deactivated.

The account must not be deleted because deletion may remove information required for audit and investigation.

Suspension should not normally be used for a permanent leaver because suspension is intended as a reversible temporary restriction.

Appropriate uses for suspension may include:

- Temporary leave
- Investigation
- Extended absence
- Short-term access freeze

### 7.3 Procedure

1. Sign in to the Okta Admin Console.

2. Navigate to:

   `Directory → People`

3. Search for and open the user.

4. Confirm the user’s identity and termination instruction.

5. Select:

   `More Actions → Deactivate`

6. Confirm the deactivation.

7. Verify that the account status changes to **Deactivated**.

8. Attempt to sign in using the deactivated account or otherwise confirm that authentication is blocked.

9. Review the user’s application and group access.

10. Review the System Log for the deactivation event.

### 7.4 Expected automated outcome

Following deactivation:

- The user can no longer sign in
- Active sessions are terminated
- Application access is removed
- The account remains visible for audit purposes
- The System Log records the administrator, user, action, and timestamp

### 7.5 Validation

The IAM administrator must verify:

- The account status is Deactivated
- Login attempts fail
- Active application access has been removed
- The user’s audit history remains available
- The System Log contains the deactivation event
- The action occurred within the organisation’s required offboarding timeframe

### 7.6 Evidence

Capture and retain:

- User account showing Deactivated status
- Failed login attempt
- System Log deactivation event
- Application-access removal evidence, where available

Store evidence in:

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
