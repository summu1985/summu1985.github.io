---
title: "The 8-Hour Identity"
subtitle: "Just-in-time access for temporary workers without permanent enterprise accounts"
author: "Sumit Mukherjee"
date: "2026-09-14"
version: "1.0"
---

# The 8-Hour Identity

## Just-in-time access for temporary workers without permanent enterprise accounts

**Architecture Point of View · Identity Security · Zero Trust · Just-in-Time Access**

> How can an organisation give a temporary worker access at 9:00 AM and make that access disappear automatically at 5:00 PM - without creating a permanent enterprise account or raising an IAM ticket?

## Architecture position

Temporary work should not create permanent entitlement.

A worker who needs access for one desk, one purpose and one shift should receive an identity and authorization context that reflects exactly that reality: verified when needed, approved for a bounded function, restricted by context, and expired automatically.

The durable principle is:

> **Establish identity when needed. Approve access just in time. Enforce least privilege. Make authorization disappear automatically.**

This pattern deliberately separates four decisions:

| Concept | Question |
|---|---|
| Identity | Who is this person? |
| Authentication | Can the person prove control of an accepted factor? |
| Eligibility | Is this person part of the approved worker pool? |
| Entitlement | May this person perform this function now? |

A valid phone number and a successful OTP are not sufficient authorization. The access decision is:

**Verified phone + approved worker + active entitlement + permitted context = temporary application access**

## The scenario

A regular receptionist is unavailable. An office manager selects an available person from an approved pool of temporary workers. The worker has no corporate directory account and the organisation has no shift-management platform.

For the day, the worker needs to create, retrieve and update customer queries using the reception application. The worker must not receive administrative rights, bulk export, unrelated system access, or access beyond the assignment.

The operational requirement is therefore not "create a user." It is "activate a bounded entitlement."

## Logical architecture

**Office Manager**  
↓ activates worker and validity window  
**Temporary Access Service** ← **Eligibility Registry**  
↓ supplies approved entitlement  
**OTP / Identity Service**  
↓ trusted OIDC assertion  
**Identity Broker / Keycloak**  
↓ short-lived application token  
**Reception Applications / APIs**

### Stable architecture roles

| Role | Responsibility |
|---|---|
| Eligibility Registry | Minimal approved-worker record and status |
| Temporary Access Service | Approval, access profile, validity, approver, reason and revocation |
| OTP / Identity Service | Authentication factor verification and trusted identity assertion |
| Identity Broker | Claim mapping, bounded session and token issuance |
| Application / Policy Layer | Server-side enforcement of permitted operations and context |

The identity platform should not become the worker-scheduling system. Eligibility and time-bound entitlement remain business decisions.

## Access journey

1. Manager selects an eligible worker.
2. Manager activates the Reception profile until a defined time.
3. Worker enters the registered phone number.
4. Identity service verifies eligibility and an active entitlement.
5. OTP is issued and verified.
6. Trusted identity and entitlement claims are passed to the identity broker.
7. A short-lived token is issued.
8. The application enforces role, scope, context and expiry.
9. Access disappears automatically when the entitlement expires or is revoked.

The important artifact is an **expiring entitlement**, not a permanent employee account.

## Claims and authorization

A trusted assertion may include:

```json
{
  "sub": "tmp-1042",
  "worker_type": "temporary",
  "access_profile": "reception",
  "location": "reception-01",
  "entitlement_id": "ent-7f93",
  "approved_by": "emp-4582",
  "valid_until": "2026-09-14T17:00:00+05:30"
}
```

Only trusted server-side components may supply claims used for authorization. Browser-supplied role, location, approval or expiry values must never become authoritative.

A practical model is **attribute-based context feeding a constrained RBAC profile**.

## Expiry and revocation

Automatic expiry must be enforced, not displayed.

Controls should include:

- token expiry that never exceeds the approved entitlement window;
- bounded session idle and maximum lifetimes;
- denial of refresh or token exchange after entitlement expiry;
- entitlement re-checks for sensitive operations where appropriate;
- immediate manager revocation;
- a worker-blocking path for lost phones or suspected misuse.

The worker may remain eligible for future assignments while having no current authorization.

## Identity-broker implementation

Keycloak is one possible implementation of the identity-broker role. It can broker OIDC identity, map trusted claims, issue application tokens and maintain bounded sessions.

Where compatible with the selected broker flow and required features, transient users can reduce unnecessary persistence. Where a durable representation is technically required, keep the identity minimal and make the entitlement strictly time-bound.

The architecture objective is **eliminating standing access**, not forcing a transient-user implementation where it reduces supportability.

## Security baseline

Minimum controls:

- pre-register and verify workers before eligibility;
- mask phone numbers in operational interfaces;
- throttle OTP attempts and protect against replay and abuse;
- record approver, reason, profile, activation and expiry;
- enforce least privilege on the server side;
- prohibit offline access for temporary-worker clients;
- correlate worker and entitlement identifiers in audit logs;
- provide immediate revocation;
- define retention and deletion rules for worker and audit data.

For higher-risk applications, add stronger authentication, managed-device or network context, step-up controls, sensitive-field masking, anomaly detection and supervisor approval.

SMS OTP is a convenience mechanism, not a universal assurance level. Authentication strength must match the sensitivity of the data and actions exposed.

## Build the smallest business service necessary

An organisation without shift management does not need to build a workforce-management platform just to solve this access problem.

A lightweight eligibility registry can answer: "Is this person approved to be in the temporary worker pool?"

A lightweight access service can answer: "Has this person been approved for this function, at this location, until this time?"

IAM then consumes those trusted answers.

This keeps workforce workflow outside the identity platform while avoiding daily manual provisioning.

## Alternatives

### Entire eligible pool can authenticate at any time
Lowest operational overhead, but weakens the connection between today's assignment and today's access. Suitable only for lower-risk scenarios with strong contextual controls.

### Manager activates access on arrival - recommended baseline
One deliberate approval action produces a clear audit record and is usually the best balance when no scheduling platform exists.

### Physical-presence-assisted activation
A rotating desk QR code, managed kiosk or approved network can add context, but should not replace eligibility and authentication.

### Provision-and-delete lifecycle
May be necessary for legacy applications, but reintroduces synchronization, cleanup and orphaned-account risk.

## What to test

A useful proof should demonstrate:

1. an eligible worker activated for a bounded reception profile;
2. OTP authentication using a development provider;
3. permitted create/retrieve/update functions;
4. denied administrative and bulk-export functions;
5. trusted context visible in the token without unnecessary personal data;
6. automatic denial after expiry;
7. immediate manager revocation;
8. audit correlation across approver, entitlement, worker and application action;
9. no unnecessary durable identity when transient brokering is used.

## Architecture takeaway

The key question is not:

**"How quickly can we create and delete a temporary user?"**

It is:

**"Can we make access exist only for the task, context and time for which it was approved?"**

That reframes temporary workforce access from an account-lifecycle problem into a just-in-time authorization problem.

---

**Sumit Mukherjee**  
*Enterprise Architecture · Cloud Native · Application Modernization · Security*  
Architecture Point of View · 14 September 2026 · v1.0

*This is an independent architecture Point of View. Product names illustrate possible implementations and do not imply endorsement or representation by any employer or vendor.*
