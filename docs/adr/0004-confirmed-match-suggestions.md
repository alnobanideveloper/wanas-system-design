# ADR 0004. Confirmed match suggestions

| | |
| --- | --- |
| **Status** | Accepted |
| **Date** | 2026-11-07 |

## Context

A volunteer takes part in matching only after review. A connection becomes active only after the required people accept it (OC-06). Profiles already store interests, language, and area. The home pin is kept so area matching can use it, and it stays off responses about another person. The final system still needs a way to propose a pair.

## Alternatives

| Alternative | Why it was set aside |
| --- | --- |
| A fully manual match, with no suggestions | Administrators would pair people with no computed shortlist. Interests, language, and area would not be used to propose a pair. |
| A suggestion that becomes active on its own | OC-06 requires review and acceptance. An automatic active match would skip the administrator and both people. |
| A separate matching service | Matching is a module inside the monolith. A second service would add a process on the same 4 GB host (ADR 0001). |

## Decision

Match suggestions are computed inside the monolith from shared interests, language, and area. A suggestion is stored as `proposed` and stays inactive. An administrator confirmation moves it to `offered`. It becomes `active` only after the senior and the volunteer have both accepted.

## Reasons

The inputs are already on the profiles, so the monolith can propose a pair without a second service (Section 4.3). `ConfirmedByAdminId` is required once the status is `offered` or `active`. An active row also requires `SeniorAcceptedAt` and `VolunteerAcceptedAt`. A decline sets the status to `declined`. Area matching uses the stored home pin. A response about another person returns `areaName` only.

## Consequences

**Benefits**

- A pair can be proposed from profile data before anyone treats it as a match.
- No match is active until an administrator has confirmed it and both people have accepted.
- The matching module can change without a change to registration or to notification delivery.

**Disadvantages**

- Every suggestion waits on an administrator and on both people. A computed pair stays inactive until those steps are stored.
- Area matching depends on the home pin. A response that returned the pin would expose a private location. Section 11 records that risk.
- The comparison of profiles runs in the API process, on the same 2 vCores as requests and reminders.
