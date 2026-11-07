# 12. Cross-Cutting Concepts and Glossary

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 12: Cross-cutting Concepts and Glossary |
| **Status** | Draft for the final system |

These rules apply to every module. The mobile app, the admin dashboard, and the API use the same words for the same records.

## 12.1 Error handling

Every failed API call returns one JSON object:

```json
{ "error": "invalid_code" }
```

The `error` value is a stable code the client can show in Arabic or English. The HTTP status says which kind of failure it is.

| Status | Meaning | What happens to stored data |
| --- | --- | --- |
| 400 | The JSON failed validation. | Nothing is written. |
| 401 | The token is missing or not valid. | Nothing is written. |
| 403 | The account is locked, or the role or family permission does not allow the call. | The row is left as it is. |
| 404 | The id does not match a row the caller may see. | Nothing is written. |
| 409 | The call conflicts with the current status, such as a phone that already has an account, or a match that is not active. | Nothing new is written. |
| 502 | WhatsApp did not accept the registration message. | The pending code is removed. |

A locked account uses `account_locked`. A role or permission refusal uses `forbidden`. Clients show the failure. They do not decide the rule.

## 12.2 Validation

The API checks the body before it changes PostgreSQL. Screens in the mobile app and the admin dashboard can guide the person, and the stored rule stays on the server.

The checks that apply across modules:

| Check | Where it is enforced |
| --- | --- |
| Role is Senior, Volunteer, Family, or Admin. | `Account.Role`. |
| Preferred language is `ar` or `en`. | Account and profile. |
| A match is active only after the senior and the volunteer have both accepted, and an administrator has confirmed it. | `Match.Status` and the acceptance timestamps. |
| A call is stored only for an active match. | The scheduling use case reads `Match.Status` first. |
| A help category is grocery shopping, transportation or accompaniment, technology help, or a small errand. | `HelpRequest.Category`. |
| A listing type is donate, lend, or request. | `EquipmentListing.ListingType`. |
| A family action requires an approved link and the matching permission flag. | `FamilyLink.ConsentStatus` and the flag for that action. |

## 12.3 Logging

The API and the admin dashboard write to container logs on the VPS. A log line names the operation and the failure code. It omits passwords, bearer tokens, phone numbers, and home coordinates. The team reads those logs on the single host. There is no separate log product.

## 12.4 Security conventions

Sign-in for a senior, a volunteer, or a family member is a WhatsApp code, then a bearer token. An administrator uses the staff sign-in and receives a token whose role is Admin. The token carries one role. Database passwords, the WhatsApp credential, and the FCM and APNs credentials are supplied when the container starts. They are not stored in git or in the image.

A response about the caller may include that caller's own phone number and home pin. A response about another person includes `areaName` and a display name. Section 10 states the quality goals that depend on this rule.

## 12.5 Modules and shared names

The API is one C# process. Inside it, accounts, profiles, matching, scheduling, help requests, equipment, notifications, safety, and administration are separate modules. A change to matching does not require a change to registration or to notification delivery. The mobile app and the dashboard render state and call the API. They do not keep a second copy of these rules.

Code and documents use the product words below. A table name, a JSON field, and a sentence in this document should point at the same concept.

## 12.6 Glossary

| Term | Meaning in Wanas |
| --- | --- |
| Account | The login for one person. It has one role, one phone number, a display name, and a preferred language. |
| Role | Senior, Volunteer, Family, or Admin. The bearer token carries one role. |
| Profile | The senior or volunteer record attached to an account: area, interests, language, and, for a volunteer, availability and verification. |
| Home pin | The latitude and longitude a senior or volunteer places on the map. It is stored for area matching and is returned only to that person. |
| Area name | The place name other people see. It replaces the home pin and the phone number in those responses. |
| Family link | The record that connects one family account to one senior after consent. It holds the permission flags. |
| Consent | The senior's decision on a family link: pending, approved, or revoked. Revoking it clears the permission flags. |
| Match | The one volunteer linked to a senior. The row moves from proposed, to offered, to active, or to declined or ended. |
| Suggestion | A proposed match chosen from shared interests, language, and area. It is not an active match. |
| Acceptance | The senior's or the volunteer's decision on an offered match. Both are required before the match is active. |
| Interaction | A scheduled phone call on an active match: start time, status, and reminder status. The audio call stays on the telephone network. |
| Help request | An everyday request in one of the four allowed categories. It belongs to a senior and can name the volunteer who accepted it. |
| Equipment listing | A donate, lend, or request item. It has a status, an owner, and optional photos. |
| Handover | The recorded collection arrangement for a listing. The meeting itself happens outside the software. |
| Safety report | A concern filed by a signed-in person. An administrator records the outcome. |
| Admin action | The kept record of a lock, an unlock, an ended match, a hidden listing, or a report outcome, naming the administrator and the time. |
| Registration code | A one-time code sent on WhatsApp. It is stored until it is used or it expires, then deleted. |
| Bearer token | The session secret returned after a code is verified. Each later call sends it as `Authorization: Bearer`. |
| Locked account | An account with `IsLocked` set by an administrator. Signed-in calls return `403` and leave other rows unchanged. |

## 12.7 References

- Section 2, for the conventions on validation, errors, logs, and secrets.
- Section 7, for the columns these terms name.
- Section 8, for the error body and the role checks.
- Section 10, for authentication and private data.
- arc42 Section 12, Cross-cutting Concepts: https://docs.arc42.org/section-12/
