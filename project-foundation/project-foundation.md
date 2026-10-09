# Northstar Cloud: project foundation

Version: 0.3 · Date: 29 September 2026 · Product baseline: 1.0

Status: working portfolio baseline; expanded documentation drafts awaiting editorial review. All product behaviour below is fictional and deliberately specified for this project. It is not a description of an implemented service or a security implementation specification.

## 1. Product concept and boundaries

Northstar Cloud is a web application for administering an organization's Northstar user accounts. Administrators invite users, organize them into teams, assign predefined roles, configure sign-in requirements, and review administrative activity.

The concept draws on the ordinary administration functions found in B2B SaaS products. Its interface, sample data, and documentation will be created for this project. No existing product is the reference implementation.

The sample organization is Alder Field Services, a fictional business with a small central operations team. Its administrator needs to onboard colleagues, keep access current, and investigate routine sign-in problems. The example is intentionally limited to administration; Northstar does not include a separate field-service application.

### Audience and purpose

Primary readers are organization administrators who understand basic account administration but may need help with authentication settings. Secondary readers are members looking for sign-in assistance. SSO configuration assumes access to an identity provider and familiarity with SAML terminology.

The portfolio should demonstrate task selection, information architecture, clear procedures, permission-aware writing, consistent UI references, and review of AI-generated drafts. It should not imply that a production application has been built or that the documentation has been tested against one.

### Scope

| Included in baseline 1.0 | Excluded |
| --- | --- |
| One organization per account | Organization switching, parent organizations, tenants, resellers |
| Invitations and account status management | Bulk import, SCIM, automatic provisioning, external guest roles |
| Flat teams for grouping users | Nested teams, team managers, team-level permissions |
| Three predefined organization roles | Custom roles, individual permission overrides |
| Password sign-in and one SAML SSO connection | Social login, multiple identity providers, passwordless sign-in |
| Authenticator-app MFA for password sign-in | SMS, push approval, hardware-key administration |
| Personal email notification preferences | Notification rules, digests, SMS, chat integrations |
| Activity filtering and CSV export | Analytics dashboards, scheduled reports, custom report builders |
| SSO as the single integration example | Public API, webhooks, app marketplace, external account management |

“Basic integrations” is deliberately narrowed to one SSO connection. An Integrations page would duplicate Authentication without adding a useful task, so it is not included.

## 2. Authority and change control

This document is the source of truth for product behaviour, permissions, terminology, and UI labels. Guides and mockups must follow it. They cannot introduce features to make a procedure easier to write or a screen look fuller.

Rule identifiers below provide traceability for working drafts. They are internal references, not content that must appear in published guides.

When a contradiction appears:

1. Record the conflicting statements and affected rule, guide, or screen.
2. Decide the intended behaviour in this specification.
3. Record the decision and increment the specification version.
4. Update affected documentation, mockups, and sample data together.

Do not silently resolve a conflict in only one artifact. An undefined behaviour is an open question, not permission for AI to invent it.

## 3. Product specification

### Organization and account model

| ID | Rule |
| --- | --- |
| ORG-01 | An organization is the boundary for users, teams, settings, and activity records. Each account belongs to exactly one organization. |
| ORG-02 | The organization is provisioned before the documented setup workflow begins. Its first user is an active Owner with password sign-in and MFA already enrolled. Self-service organization creation is outside scope. |
| ORG-03 | Organization settings contain Organization name and Organization ID. The name is editable, required, and 2–80 characters after trimming surrounding spaces. The ID is read-only and does not change when the name changes. |
| ORG-04 | All displayed activity timestamps and report date boundaries use UTC. There is no organization timezone setting. |
| USR-01 | A user record contains name, email address, role, status, team memberships, and last sign-in. An invited user has no name until accepting the invitation. Email addresses are compared case-insensitively and cannot be edited in this version. |
| USR-02 | User status is Invited, Active, or Deactivated. An expired invitation remains Invited and displays a separate Expired invitation indicator. Expired is not a fourth user status. |
| USR-03 | Users are managed individually. There are no bulk actions or permanent deletion of accepted accounts. Deactivated accounts remain available in user lists and historical activity records. |

### Roles and Permissions

Every user has exactly one organization role: Owner, Administrator, or Member. Roles apply throughout the organization. Teams never grant permissions.

| Capability | Owner | Administrator | Member |
| --- | --- | --- | --- |
| View own overview, profile, and team memberships | Yes | Yes | Yes |
| Change own name and optional email notifications | Yes | Yes | Yes |
| Manage own password-sign-in MFA | Yes; cannot disable | Yes; cannot disable | Yes; subject to policy |
| View the organization user directory and all teams | Yes | Yes | No |
| Invite users as Administrator or Member | Yes | Yes | No |
| Resend or revoke pending invitations | Yes | Yes | No |
| Change another Member to Administrator or reverse that change | Yes | Yes | No |
| Assign or remove the Owner role | Yes | No | No |
| Deactivate/reactivate another Administrator or Member | Yes | Yes | No |
| Deactivate/reactivate another Owner | Yes; safeguards apply | No | No |
| Reset another user's local MFA | Yes; other users only | Other Members only | No |
| Create, edit, or delete teams; manage membership | Yes | Yes | No |
| Edit organization settings | Yes | No | No |
| Configure SSO or the organization MFA policy | Yes | No | No |
| View roles reference and activity reports; export CSV | Yes | Yes | No |

| ID | Safeguard |
| --- | --- |
| ROLE-01 | An organization must always have at least one active Owner. An action that would remove the last active Owner is blocked. |
| ROLE-02 | Users cannot change their own role, deactivate themselves, or reset their own MFA through user administration. Another authorized user must perform those actions. |
| ROLE-03 | Invitations can assign Administrator or Member, never Owner. An Owner may promote an active user after invitation acceptance. Roles cannot be changed while a user is Invited or Deactivated. |
| ROLE-04 | Administrators cannot change an Owner's role, status, or MFA. The application hides unauthorized controls and also rejects unauthorized direct requests. |
| ROLE-05 | Role and status changes invalidate the affected user's existing sessions. A new sign-in evaluates current permissions and authentication requirements. |

### Invitations and account lifecycle

| ID | Rule |
| --- | --- |
| INV-01 | Invite user accepts one Email address and one Role, defaulting to Member. Successful submission creates an Invited record and sends an invitation. Existing Invited, Active, or Deactivated email addresses cannot be invited again. |
| INV-02 | Invitations expire seven days after issue. Resend invitation invalidates the previous link and starts a new seven-day period. Revoke invitation invalidates the link and removes the pending user record; the activity event remains. |
| INV-03 | Acceptance requires the invited email identity, a name, and successful authentication. Password acceptance creates a password; SSO acceptance uses an exact case-insensitive match with the invited email. Authentication and any required local MFA enrollment complete before status becomes Active. |
| INV-04 | Teams can contain Active or Deactivated users, but new membership can be added only for Active users. Invitations do not offer team selection. |
| USR-04 | Deactivation immediately blocks all sign-in methods and invalidates sessions. Role and team memberships are retained. A confirmation explains these effects. |
| USR-05 | Reactivation restores sign-in eligibility with the retained role and memberships. It does not bypass current SSO or MFA requirements or issue a new invitation. |

### Teams

| ID | Rule |
| --- | --- |
| TEAM-01 | A team has a required Name of 2–60 characters and an optional Description of up to 200 characters. Names are unique within the organization, case-insensitively, after trimming spaces. |
| TEAM-02 | A user can belong to zero or more teams. Teams are flat and have no owner, manager, or role assignment. |
| TEAM-03 | Removing membership does not deactivate the account. Deleting a team removes its memberships and leaves user accounts and roles unchanged. Team deletion requires confirmation. |
| TEAM-04 | Team member counts include retained Deactivated members. Their status is visible in the membership list. |

### Authentication and recovery

SSO and local MFA are separate controls. An identity provider's MFA is configured outside Northstar. Northstar does not add a local MFA challenge after successful SSO.

| ID | Rule |
| --- | --- |
| AUTH-01 | Default sign-in is email and password. Passwords are 12–128 characters; there is no composition rule. Password reset links expire after 30 minutes, are single-use, and a new request invalidates the previous link. Resetting a password does not reset MFA. |
| AUTH-02 | Five consecutive failed password attempts temporarily block password sign-in for that account for 15 minutes. SSO is unaffected. A successful password sign-in clears the failure count. Administrators cannot manually unlock an account. |
| SSO-01 | An organization can configure one SAML 2.0 connection. The UI exposes Service provider entity ID and Assertion consumer service URL as read-only values, and accepts Identity provider entity ID, Sign-in URL, and X.509 certificate. Email is supplied as the NameID identifier. |
| SSO-02 | Connection states are Not configured, Draft, and Enabled. Saving valid connection fields creates a Draft. Test connection starts a separate identity-provider sign-in and must return the current Owner's email successfully. A successful test is required before Enable SSO becomes available. Editing draft fields invalidates the test result. |
| SSO-03 | Enabled SSO is initially optional. Members and Administrators may use SSO or password sign-in. Turning on Require SSO blocks password sign-in for those roles and invalidates their sessions. The Owner must first acknowledge the effect in a confirmation dialog. |
| SSO-04 | Owners retain password sign-in with mandatory local MFA, even when Require SSO is on. This provides a recovery path. Owners may also use SSO. The exception is explained beside Require SSO and in both authentication guides. |
| SSO-05 | SSO does not create accounts, assign roles, sync teams, or reactivate users. The user must have an active account or a valid pending invitation. |
| SSO-06 | Enabled connection fields are read-only. To replace them, an Owner uses Disable SSO, edits the retained draft, tests it, and enables it again. Disabling SSO also clears Require SSO and invalidates all sessions; users without a password must use Forgot password before password sign-in. |
| MFA-01 | Northstar supports one authenticator-app enrollment per user and ten single-use recovery codes. Owners and Administrators must enroll before their first password-authenticated session. Members enroll optionally unless Require MFA for password sign-in is on. |
| MFA-02 | Only an Owner can change Require MFA for password sign-in. The default is off. Enabling it invalidates password-authenticated sessions; affected users enroll at their next password sign-in. SSO sessions and identity-provider MFA are unaffected. |
| MFA-03 | Enrollment requires scanning a QR code or entering a setup key, verifying a six-digit code, and acknowledging that recovery codes have been saved. Secrets and usable recovery codes must never appear in portfolio screenshots. |
| MFA-04 | Disabling optional MFA requires a password plus an authenticator code or unused recovery code. It is unavailable when role or policy requires MFA. An authorized administrative reset removes the enrollment and recovery codes, invalidates sessions, and requires new enrollment if MFA is mandatory. |
| MFA-05 | A user who loses their authenticator uses a recovery code or asks an authorized administrator for a reset. A sole Owner without a recovery code must contact Northstar support. Support identity verification is outside this sample; no automatic bypass is promised. |

These are documentation-level behaviours. Protocol validation, cryptography, abuse prevention, and support identity-verification procedures are outside the portfolio specification.

### Notifications and activity reports

| ID | Rule |
| --- | --- |
| NOT-01 | Notification preferences belong to the signed-in user. There are two optional email categories, both on by default: Team membership changes and Product updates. There are no organization-wide notification preferences. |
| NOT-02 | Invitations, password resets, account deactivation/reactivation notices, role changes, and local MFA changes are required account emails and cannot be disabled. Northstar does not claim to send alerts about MFA changes at the identity provider. |
| NOT-03 | Team membership changes sends one email to each affected Active user when added, removed, or removed by team deletion. Deactivated users receive no optional emails. Events that occurred during deactivation are not sent later. |
| RPT-01 | Activity reports record invitation actions, account activation, role/status changes, team changes, organization-name changes, SSO changes, MFA policy/enrollment/reset changes, sign-in successes/failures, and report exports. Personal name and notification-preference edits are excluded from this sample's event set. |
| RPT-02 | Each event contains timestamp, actor, action, target, and result. Results are Success or Failure. Unknown identities in failed sign-ins use Unknown user; attempted email addresses are not exposed. Passwords, tokens, certificates, and MFA secrets are never report fields. |
| RPT-03 | Events are retained for 90 days. The default date range is the last seven UTC calendar dates, including today. Custom ranges are inclusive, cannot include future dates, and must fall within the retained period. |
| RPT-04 | Filters are Date range, Actor, and Category. Categories are All categories, Users, Teams, Authentication, Organization, and Reports. Apply filters refreshes the table. Export CSV downloads the complete filtered result, not just the visible page. No scheduled delivery is offered. |
| RPT-05 | A result set with no records displays No activity found. Export CSV is disabled for an empty result. Export timestamps use ISO 8601 UTC; the UI uses an explicit UTC label. The export activity event is recorded after the exported dataset is captured. |

## 4. Core workflows

These are workflow contracts for later guides, not complete procedures.

| Workflow | Actor and prerequisite | Main flow | Result and important exception |
| --- | --- | --- | --- |
| Set up the organization | Active Owner; initial access already provided | Open Organization; review ID; edit name; save | Name changes throughout the UI. No signup or billing steps. |
| Invite the first user | Owner or Administrator | Open Users; select Invite user; enter email and role; send | Invited record appears. Duplicate email is rejected; use the existing record. |
| Accept an invitation | Invitee; unexpired link | Open link; provide name; authenticate; enroll in required local MFA | User becomes Active. Expired/revoked links require help from an administrator. |
| Manage a user | Authorized role; existing record | Open user details; change role, deactivate, or reactivate | Permissions and sessions follow the rules above. Last-Owner and self-action safeguards apply. |
| Create a team | Owner or Administrator | Create named team; add Active users | Grouping changes; permissions do not. Duplicate names are rejected. |
| Configure SSO | Owner; identity-provider administration access | Exchange metadata values; save draft; test; enable; optionally require SSO | Existing accounts can authenticate with the provider. Failed tests leave SSO disabled. |
| Configure MFA | Owner for policy; each user for enrollment | Set policy if needed; affected users enroll at password sign-in | Local password sign-in is protected. SSO uses provider requirements. |
| Change notifications | Any active user | Open My profile > Notifications; set preferences; save | Only that user's optional email preferences change. |
| Generate an activity report | Owner or Administrator | Open Activity reports; set filters; apply; export CSV | CSV reflects applied filters and full results. No results means no export. |
| Recover sign-in access | User; administrator or Owner where necessary | Identify invitation, status, password, SSO, or MFA issue; use the matching recovery action | Troubleshooting follows actual account state. Password reset cannot bypass required SSO, deactivation, or MFA. |

## 5. UI and navigation contract

The following are exact interface labels. Documentation must preserve their capitalization. Routes identify mockup screens; they are not claims that a live application exists.

### Application shell

Use a persistent header containing Northstar Cloud, the organization name, Help, and a profile menu. The left sidebar uses this fixed order: Overview, Users, Teams, Roles and permissions, Activity reports, Settings. Settings expands to Organization and Authentication. The Authentication screen has Single sign-on and Multi-factor authentication tabs.

Owners see all items. Administrators do not see Settings. Members see Overview and My teams only. The profile menu contains My profile and Sign out for all roles. My profile has Profile, Security, and Notifications tabs. Help links to the documentation home.

| Screen / route | Exact labels and primary actions | Defined content |
| --- | --- | --- |
| Overview `/overview` | View users; View teams | Owners/Administrators: user-status counts, team count, links to administration. Members: own role and memberships; no organization-wide counts. |
| Users `/users` | Invite user; Search users; Status; Role | Columns: Name, Email address, Role, Status, Last sign-in. Search matches name/email; filters combine. Default sort: Email address ascending. |
| User details `/users/{id}` | Change role; Deactivate user; Reactivate user; Reset MFA; Resend invitation; Revoke invitation | Show only actions permitted for the viewer and applicable to the target's status. Teams shown separately. No Edit email action. |
| Invite user dialog | Email address; Role; Send invitation; Cancel | Default role Member. No team field or Owner option. |
| Teams `/teams` | Create team; Search teams | Columns: Name, Members. Default sort: Name ascending. |
| Team details `/teams/{id}` | Edit team; Add members; Remove from team; Delete team | Name, Description, membership list with user status. Add members selects Active users only. |
| My teams `/my-teams` | No editing actions | Member's own team names and descriptions; no directory of other members. |
| Roles and permissions `/roles` | No editing actions | Read-only comparison of the three roles. Role changes happen in user details. |
| Activity reports `/activity` | Date range; Actor; Category; Apply filters; Export CSV | Columns: Timestamp (UTC), Actor, Action, Target, Result. Newest first; 25 rows per page. |
| Organization `/settings/organization` | Organization name; Organization ID; Save changes | ID is read-only. |
| Authentication: Single sign-on `/settings/authentication/sso` | Service provider entity ID; Assertion consumer service URL; Identity provider entity ID; Sign-in URL; X.509 certificate; Save configuration; Test connection; Enable SSO; Require SSO; Disable SSO | Status, test outcome, and Owner recovery exception. Enabled fields are read-only. |
| Authentication: Multi-factor authentication `/settings/authentication/mfa` | Require MFA for password sign-in; Save changes | Mandatory MFA for privileged password sign-in is explained even when the switch is off. |
| My profile: Profile `/profile` | Name; Email address; Save changes | Email is read-only. |
| My profile: Security `/profile/security` | Set up MFA; Disable MFA | Enrollment status and role/policy restriction. Password recovery starts from the sign-in screen. |
| My profile: Notifications `/profile/notifications` | Team membership changes; Product updates; Save changes | Optional switches and explanation of required account emails. |

### Sign-in and interaction conventions

The public sign-in screen collects Email address, then shows Continue with SSO when the organization has enabled SSO. Password sign-in is available according to the user's role and organization policy. Its labels are Password, Sign in, and Forgot password. There is no organization selector.

Forms use visible labels, inline validation, and a primary action at the bottom right. Cancel appears immediately before the primary action. Save changes is disabled until values change. A failed save preserves entered values and explains what needs correction.

Destructive confirmations name the affected user or team, explain consequences, and use the exact action as the confirmation button. Do not use a generic Yes button. Unauthorized actions are hidden; temporarily unavailable authorized actions are disabled with an explanation.

List screens use 25 rows per page with Previous and Next controls. Empty lists explain the state and provide the relevant create action when permitted. Success messages describe the completed action, for example, “Invitation sent to jordan.lee@example.com.”

## 6. Terminology guide

| Use | Meaning / usage | Avoid as a substitute |
| --- | --- | --- |
| organization | Northstar's administrative boundary; use Canadian spelling throughout prose | tenant, workspace, company account |
| user | A person with an Invited, Active, or Deactivated account | seat, resource |
| Member | The specific predefined role; capitalize when naming the role | ordinary user, standard role |
| member | A person belonging to a team; lowercase in prose | Using Member when no role is intended |
| Owner / Administrator | Exact role names | super admin, admin role |
| team | A flat grouping of users; confers no permissions | group, department |
| role / permission | A named access level / an action that role permits | Using the terms interchangeably |
| deactivate / reactivate | Block / restore sign-in eligibility while retaining the account | delete, suspend, archive, enable user |
| revoke invitation | Invalidate an invitation and remove its pending record | deactivate invitation |
| single sign-on (SSO) | Authentication through the configured identity provider | integration login, federated account |
| identity provider | External service that authenticates SSO users | Northstar directory |
| multi-factor authentication (MFA) | Additional verification; identify local or provider context | two-step login, 2FA in UI labels |
| recovery code | A single-use alternative to a local authenticator code | backup password, recovery key |
| activity report | Filtered administrative and sign-in events | analytics dashboard, compliance certification |
| sign in / sign-in | Verb / adjective, respectively | login as a verb |

Introduce SSO and MFA in full on the first substantive use in each standalone guide. Do not shorten UI labels to make steps read more smoothly.

## 7. Proposed Confluence information architecture

Use About Northstar Cloud as the space home. The twelve pages below include that home page; there are no extra empty category pages. The hierarchy groups related tasks while keeping the page count small.

| Page | Parent | Purpose and boundaries | Rule references |
| --- | --- | --- | --- |
| About Northstar Cloud | Space home | Product scope, audience, task links, conspicuous fictional-sample disclosure; brief link to separate AI triage project when available | ORG-01; scope |
| Set Up Your Organization | About Northstar Cloud | Review pre-provisioned organization, change name, outline recommended next tasks | ORG-02–04 |
| Invite Your First Users | Set Up Your Organization | Send an invitation and explain acceptance, expiry, resend, and revoke; link onward to user management | INV-01–04 |
| Manage Users | About Northstar Cloud | Find users, change roles, deactivate, reactivate, and reset MFA within authority | USR-01–05; ROLE-01–05; MFA-04–05 |
| Create and Manage Teams | Manage Users | Create/edit/delete a team and add/remove membership; explain that teams do not grant access | TEAM-01–04; INV-04 |
| Roles and Permissions | Manage Users | Permission matrix and ownership safeguards; refer to Manage Users for procedures | ROLE-01–05 |
| Configure SSO | About Northstar Cloud | Prerequisites, generic SAML field mapping, test, enable, require, disable; no vendor-specific provider walkthrough | SSO-01–06 |
| Configure MFA | About Northstar Cloud | Separate Owner policy task from user enrollment and recovery; explain the SSO boundary | MFA-01–05 |
| Configure Notifications | About Northstar Cloud | Personal optional email settings and required account messages | NOT-01–03 |
| Generate Activity Reports | About Northstar Cloud | Retention, UTC, filtering, empty results, CSV export | RPT-01–05 |
| Troubleshoot Sign-In Problems | About Northstar Cloud | Symptom/cause/action table covering invitations, deactivation, temporary password block, required SSO, email mismatch, and lost MFA | AUTH-01–02; INV-02–03; SSO-03–06; MFA-04–05 |
| Release Notes | About Northstar Cloud | One clearly fictional 1.0 release entry; no invented customer outcomes or unfounded “fixed” defects | Baseline 1.0 |

Keep this foundation outside the twelve-page customer documentation set. It can be a separate portfolio process artifact. Readers should encounter a realistic help space, while reviewers can separately inspect the decisions behind it.

## 8. Documentation style assumptions

- Write to the reader as “you.” State the required role and prerequisites before the procedure.
- Use title case for document titles and Confluence article names, including their cross-references, contents entries, and bookmarks. Use sentence case for procedural subheadings. Preserve exact interface capitalization and bold actionable labels.
- Use numbered steps for procedures. Start with the location, use one primary action per step, and describe the expected result when it helps the reader verify success.
- Prefer “select” to mouse-specific instructions. Use “enter” for text and “turn on” or “turn off” for switches. Avoid “simply,” “easily,” and promotional adjectives.
- Use short, specific notes only where timing matters. Explain session invalidation and account access consequences before the action that causes them.
- Keep task guides focused. Link to the roles reference instead of repeating its entire matrix. Put recovery detail in troubleshooting and link to it from the relevant procedure.
- Use Canadian English in prose. Dates use “29 September 2026”; timestamps include UTC. UI status labels retain their exact case.
- Screenshots supplement written instructions; every task must remain understandable without them. Include descriptive alt text that states what the image helps identify.
- Distinguish intended mockup validation from application testing. Do not claim successful end-to-end testing, accessibility compliance, or measured outcomes without evidence.

## 9. Screenshot and UI consistency rules

### Visual baseline

| Element | Fixed decision |
| --- | --- |
| Canvas | Desktop, 1440 × 900 CSS pixels; browser zoom 100%; consistent device scale for all captures |
| Layout | 64 px header; 240 px sidebar; 32 px content padding; one primary action in the page header where applicable |
| Typography | Arial, sans-serif; 16 px body, 14 px table text, 28 px page titles |
| Colour | White content; background #F5F7FA; text #202B3A; muted text #526174; primary #2457A6; border #D7DEE8 |
| Components | 40 px controls; 48 px minimum table rows; 6 px corner radius; restrained borders; no gradients or decorative illustrations |
| States | Status always includes text; never communicate success, failure, selection, or account status by colour alone |
| Navigation | Current item visibly selected; header organization and current user remain consistent with scenario |
| Capture | Use reusable UI components and shared sample data. Static SVG/PNG illustrations are permitted for this portfolio edition and must be identified as fictional product mockups. They demonstrate intended interface states, not implemented or tested workflows. Browser captures may replace them later. |

These are design targets, not a claim of validated accessibility. Check text contrast, focus indicators, reading order, and label associations when building the mockups.

### Canonical sample data

Default screenshot scenario: baseline 1.0, 29 September 2026 at 14:00 UTC, signed in as Morgan Ellis (Owner). Organization: Alder Field Services. Organization ID: `org_alder_001`. All people and organizations in these fixtures are fictional.

| User | Email address | Role | Status | Teams |
| --- | --- | --- | --- | --- |
| Morgan Ellis | morgan.ellis@example.com | Owner | Active | Operations |
| Priya Shah | priya.shah@example.com | Administrator | Active | Operations |
| Casey Tran | casey.tran@example.com | Member | Active | Client Support |
| Rowan Bell | rowan.bell@example.com | Member | Deactivated | Client Support |
| — | jordan.lee@example.com | Member | Invited | None |

Baseline counts are three Active, one Invited, one Deactivated, and two teams. Each team has two members; Client Support includes one Deactivated member. Jordan's invitation was issued on 28 September 2026 at 14:00 UTC and expires on 5 October 2026 at 14:00 UTC.

Default authentication: SSO Not configured; Require MFA for password sign-in off; Morgan and Priya already enrolled in local MFA. Optional email preferences are on for all Active users.

### Capture controls

- Give each screenshot a scenario and sequence identifier. Record specification version, acting role, before/after state, relevant rule IDs, and page placement in a capture manifest when images are created.
- Use explicit scenario variants for enabling SSO, accepting Jordan's invitation, or changing account status. Do not quietly change the baseline dataset. A before/after pair must reflect the same operation.
- Use reserved example domains and synthetic data. Any displayed URL under `northstar.example` is illustrative and must not be presented as a working service.
- Do not show real customer data, third-party branding, copied product layouts, working tokens, scannable enrollment secrets, or certificate contents. Use clearly labelled nonfunctional placeholders where a sensitive field matters.
- Use context-preserving crops with consistent scale. Keep enough navigation to locate the task; never crop out an important warning to make the image cleaner.
- Add numbered callouts only when needed and keep their style consistent. Do not draw controls that are absent from the reusable UI.
- Identify the space as a fictional portfolio sample on its home page. Caption screenshots “Northstar Cloud 1.0 — fictional product mockup.” This distinction must survive reuse on the portfolio site.

## 10. Editorial review and AI-assisted workflow

AI can propose scaffolding, generate initial drafts, and help check terminology. Eliza's editorial review should determine task boundaries, sequencing, explanations, and whether the content is useful. Record actual decisions and revisions as they happen; do not manufacture an editorial history.

For each future guide, retain a small review record: relevant rule IDs; unresolved questions; AI draft status; editorial changes with reasons; and the UI scenario used to check the procedure. Particularly useful examples will be corrections to permission assumptions, SSO/MFA interactions, and misleading account-state instructions.

Before treating a guide as ready, verify its UI path and labels, permitted role, prerequisites, resulting state, exceptions, links, screenshot state, and disclosure. A mockup walkthrough checks consistency, not production behaviour.

The AI triage agent remains a separate project with its own inputs, evaluation, and case study. The help-space home may link to that project under a short portfolio note once it exists. Do not imply that the agent runs inside Northstar or depends on this content.

### Decisions resolved in this baseline

| Potential contradiction | Resolution |
| --- | --- |
| Broad “administration platform” implies external account provisioning | Access management is limited to Northstar itself. |
| “Basic integrations” implies another guide and feature area | One SSO connection satisfies the limited integration scope. |
| Teams appear to grant roles | Teams are organizational groupings only. |
| Required SSO could lock out the last Owner | Owner password sign-in with mandatory MFA remains available. |
| Required MFA could imply a second challenge after SSO | Local MFA policy applies only to password sign-in. |
| Expired invitations could inflate the status model | Expired is an invitation indicator within Invited. |
| Notifications could imply an organization policy | Optional notification preferences are personal. |
| Twelve guides plus category pages could exceed the requested size | The home page is one of twelve; existing guides act as parents. |

No business-rule conflicts were identified while drafting the twelve-guide set. Actual identity-provider instructions and backend security design remain outside scope. The documentation and static illustrations require Eliza’s editorial review; neither production testing nor Confluence import validation has occurred.


## 11. Documentation expansion decisions for version 0.2

Date: 29 September 2026. These decisions complete previously unspecified UI details while retaining the baseline business rules. They were selected during AI-assisted drafting under the instruction to complete the documentation set; they are not recorded as individual editorial approvals by Eliza.

| Area | Clarification | Affected materials |
| --- | --- | --- |
| User details | Select the Email address link on Users to open details. The Change role dialog contains Role and Save changes. Reactivate user requires a confirmation with the same label. Reset MFA is shown only for an eligible target with local MFA enrolled; it is unavailable for pending invitations. | Manage Users; invitation guide; user details illustration |
| Teams | Select the team Name to open details. Create team uses Name, Description, and Create team. Edit team uses Name, Description, and Save changes. Add members provides active-user selection and an Add members confirmation. Remove from team appears beside each membership and opens a confirmation naming the person and team. | Create and Manage Teams; team illustrations |
| Required SSO | The confirmation button is Require SSO. Disabling uses Disable SSO in both the page and confirmation. | Configure SSO |
| MFA enrollment | Verification uses Authentication code and Verify code. Completion uses I have saved my recovery codes and Finish setup. The MFA challenge and optional-disable flow offer Use a recovery code. Disable MFA requires Password and either Authentication code or an unused recovery code. | Configure MFA; troubleshooting |
| Password recovery | Forgot password opens a request for the entered email using Send reset link. The emailed reset form collects a new password and confirmation with Reset password. | Troubleshooting |
| Reports | Unrestricted Actor is All actors. Date range offers Custom with Start date and End date. Category mapping: invitation/account/role/status events → Users; team/membership events → Teams; SSO/local MFA/sign-in events → Authentication; organization-name events → Organization; exports → Reports. | Generate Activity Reports; report illustration |
| Illustrations | Deterministic static SVG/PNG illustrations are the documented production method for this edition. They are not browser screenshots or evidence of implemented behaviour. Sensitive authentication values are omitted. | All guides; image manifest |
| Scenario variants | Casey has an optional local MFA enrollment in user-details.png so Reset MFA is applicable. MFA-policy.png shows an unsaved change to On. Create-team.png shows an unsaved new Service Desk team. These do not change baseline users, memberships, or policy. | Image manifest |

Document format: Carlito 11 pt body text, 1.3 line spacing, explicit visible procedure numbers, original Northstar logo, and version/sample disclosure in the footer. Explicit numbers must be renumbered when steps change. Internal cross-references use page titles until real Confluence URLs exist. The home page remains one of the twelve Confluence pages even when its Word layout spans more than one printed page.


## Version 0.3 style decision

Eliza requested title case for document titles and article names on 29 September 2026. The twelve titles, title references, contents, and PDF bookmarks have been updated. Product UI labels retain their specified capitalization.
