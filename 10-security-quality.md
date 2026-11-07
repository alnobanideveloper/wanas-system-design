# 10. Security and Quality

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 10: Security and Quality |
| **Status** | Draft for the final system |

This section records how the final system checks who is calling, what that person may do, and which pilot quality goals the design is built to support. The rules below are the ones already set in Sections 1, 2, 7, and 8.

## 10.1 Authentication

Seniors, volunteers, and family members sign in with a WhatsApp one-time code. They use the same two registration calls. The `role` in the body decides which account is created. The API asks WhatsApp to deliver the code. The response does not contain the code. A correct code stores the account, removes the used code, and returns the session token.

Administrators use their own staff sign-in. A staff account already exists. The administrator requests a code on the staff path. The API sends that code when the phone already belongs to an administrator. A verified code returns a session token with the role `Admin`.

The session is a bearer token. The token carries one role: `Senior`, `Volunteer`, `Family`, or `Admin`. A signed-in request sends `Authorization: Bearer <token>`.

| Situation | Result |
| --- | --- |
| The token is missing or not valid. | `401` `unauthorized` |
| The account is locked. | `403` `account_locked`. Stored rows stay unchanged. |

## 10.2 Authorization

The four roles are Senior, Volunteer, Family, and Admin. The API allows a call when the token's role is the role that call is listed for. A call for another role returns `403` `forbidden`. Stored rows stay unchanged.

A family member has a separate account. A family member never uses the senior's password. The member is linked to a senior only after that senior gives consent. Each link has its own permissions. `CanViewSchedule` and `CanCreateHelpRequest` can differ from one family member to another. The senior can withdraw a link. Withdrawal sets consent to Revoked and clears the permission flags. The family account stays.

A family call also checks that link. Approved consent and the flag for that call are both required. A missing link or a missing permission returns `403` `forbidden`.

## 10.3 Private data

The phone number is stored on the account. The home pin is stored on the senior profile or the volunteer profile.

A response about the caller may include that caller's own phone number and home pin. A response about another person includes `areaName` and a display name only.

Logs record the operation and the failure. Logs omit passwords, phone numbers, and home addresses.

Secrets are supplied at container start. Database passwords, API keys, and Firebase or Apple credentials are provided when the container starts.

## 10.4 Input validation

The API checks the JSON before it changes data. Screens can guide the user. The rule lives in the API.

These checks are the ones already defined for the stored data.

| Check | Rule |
| --- | --- |
| Role | The value is Senior, Volunteer, Family, or Admin. |
| Active match | A match is active only after both sides have a recorded acceptance. |
| Help category | The category is grocery shopping, transportation or accompaniment, technology help, or a small errand. |
| Equipment listing type | The listing type is Donate, Lend, or Request. |

A value outside the check is rejected. The API leaves the stored rows unchanged. The help categories stay on those four everyday tasks, which is the product boundary in OC-04.

## 10.5 Quality goals

The measurable targets below come from Section 1. Each one is an assumption for the pilot. The team reviews it after testing with real users.

| ID | Pilot assumption | Design choice that supports it |
| --- | --- | --- |
| QT-04 | Every active match in the pilot has recorded acceptance from both sides. | Recorded acceptance. A match becomes active only when both acceptance times are stored. |
| QT-05 | Every reported safety concern has a recorded review and outcome. | Recorded report outcomes. The report stores the outcome, and the review writes an `AdminAction`. |
| QT-06 | No user can access another user's private information without permission. | Role checks and response filtering. The API checks the role and the family-link permissions. A response about another person returns `areaName` and a display name only. |

Three further qualities come from the same requirements and from the single-host deployment.

| Quality | What this design provides |
| --- | --- |
| Reliability | PostgreSQL stores profile, match, and request changes on a Docker volume. Migrations run before the new API serves. |
| Maintainability | Accounts, profiles, matches, scheduling, notifications, safety, and administration are separate modules in one monolith. |
| Availability | The service runs on one VPS. A host failure stops the service. The previous image tag can be started again. |

## 10.6 References

- Section 1, `01-introduction-goals-requirements.md`, for NFR-01 through NFR-12 and QT-01 through QT-06. This section uses QT-04, QT-05, and QT-06.
- Section 2, `02-architecture-constraints.md`, for TC-10, TC-11, OC-04, OC-05, CV-03, CV-04, and CV-05, and for the single VPS, the Docker volume, migrations before the new API serves, and the previous image tag.
- Section 7, `07-data-architecture.md`, sections 7.4 and 7.5, for role values, match acceptance, help categories, listing types, family-link consent, and locks that keep stored rows.
- Section 8, `08-api-design.md`, for shared sign-in, the bearer token, `401` and `403` results, and family permission checks.
- arc42 Section 10, Quality Requirements: https://docs.arc42.org/section-10/
- OWASP Application Security Verification Standard: https://asvs.owasp.org/
