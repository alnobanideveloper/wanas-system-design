# 6. Runtime View

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 6: Runtime View |
| **Status** | Draft for the final system |
| **Diagrams** | Four sequence diagrams |

Each diagram uses the mobile app or the admin dashboard, the Wanas API, PostgreSQL, and one outside service when a message leaves the server. The API box stands for the HTTP entry point, the domain rules, and the in-process worker from Section 5.

The pictures and their editable sources are in [diagrams](diagrams/).

## 6.1 Registration and authorization

A senior or volunteer creates an account in the mobile app. WhatsApp carries the one-time code. A later administrator request with that same token is refused.

![Registration and authorization](diagrams/sequence-registration.png)

| What happens | Result |
| --- | --- |
| The app asks for a code. | The API stores a pending code and WhatsApp delivers it. The response is `202`. |
| The code is wrong or expired. | The API answers `400`. No account is stored. |
| The code matches. | The API stores the account with its role, removes the code, and returns a session token. |
| The app calls an administrator route with that token. | The API answers `403`. No report data is read. |

## 6.2 Match suggestion and acceptance

A suggestion starts inactive. It becomes an active match only after the senior and the volunteer have both accepted.

![Match suggestion and acceptance](diagrams/sequence-match-acceptance.png)

| What happens | Result |
| --- | --- |
| The app asks for a suggestion. | The API reads the profiles and stores a suggestion as `proposed`. |
| An administrator confirms it. | The status becomes `offered`. The people still have to accept. |
| The senior accepts. | The acceptance is stored. The match stays inactive. |
| The volunteer declines. | The status becomes `declined`. There is no active match. |
| The volunteer accepts. | The status becomes `active`. |

A family member without permission receives `403`, and the match row is left as it is.

## 6.3 Schedule a call and send the reminder

A call time can be saved only for an active match. The phone call itself stays on the normal phone. The API sends the reminder through FCM or APNs.

![Schedule a call and send the reminder](diagrams/sequence-call-reminder.png)

| What happens | Result |
| --- | --- |
| The match is not active. | The API answers `409`. Nothing is saved. |
| The match is active. | The API stores the call as `scheduled` and answers `201`. |
| The push service rejects the reminder. | The reminder is marked failed. The call stays `scheduled`. |
| The push service accepts it. | The phone shows the notification, and the reminder is marked sent. |

## 6.4 Create and accept a help request

A senior, or a family member with `CanCreateHelpRequest`, creates an everyday help request. An approved volunteer accepts it. A family member without that permission is refused.

![Create and accept a help request](diagrams/sequence-help-request.png)

| What happens | Result |
| --- | --- |
| A family member without `CanCreateHelpRequest` submits a help request. | The API answers `403`. Nothing is saved. |
| A senior creates a request, or a family member has approved consent and `CanCreateHelpRequest`. | The API stores the request as `open` and answers `201`. |
| An approved volunteer accepts the open request. | The status becomes `accepted`. The row stores that volunteer. |

The category is grocery shopping, transportation or accompaniment, technology help, or a small errand.

## 6.5 References

- Section 5, `05-building-block-view.md`, for the mobile app, the admin dashboard, the API, and PostgreSQL.
- Section 4, `04-solution-strategy.md`, for WhatsApp registration, match acceptance, and push reminders.
- Diagram sources: [diagrams/sequence-registration.puml](diagrams/sequence-registration.puml), [diagrams/sequence-match-acceptance.puml](diagrams/sequence-match-acceptance.puml), [diagrams/sequence-call-reminder.puml](diagrams/sequence-call-reminder.puml), and [diagrams/sequence-help-request.puml](diagrams/sequence-help-request.puml).
- arc42 Section 6, Runtime View: https://docs.arc42.org/section-6/
