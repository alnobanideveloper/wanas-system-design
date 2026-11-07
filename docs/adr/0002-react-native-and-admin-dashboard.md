# ADR 0002. React Native app and admin dashboard

| | |
| --- | --- |
| **Status** | Accepted |
| **Date** | 2026-11-07 |

## Context

Seniors, volunteers, and family members register, keep profiles, review suggestions, and schedule calls. Administrators review volunteers, confirm suggestions, and handle safety reports. Arabic and English are required, including a right-to-left layout for Arabic (TC-08). One API serves every client (CV-06).

## Alternatives

| Alternative | Why it was set aside |
| --- | --- |
| A public website for seniors, volunteers, and family members | Those three roles use the mobile app. A browser site for them would be a second participant client. |
| Razor or Blazor inside the monolith | Administrator screens would then live in the process that owns the domain rules. The dashboard stays a client of the JSON API. |
| A separate Swift app and a separate Kotlin app | Two native codebases would each call the same API. TC-12 already provides the store accounts used to publish one React Native app to both stores. |

## Decision

Seniors, volunteers, and family members use one React Native app for Android and iOS. Administrators use a separate Next.js dashboard. Both clients call one JSON API. User-facing text is Arabic and English, and Arabic screens use a right-to-left layout.

## Reasons

The mobile app holds screens and local state. The dashboard holds volunteer review, suggestion confirmation, and safety reports. Authorization and business rules stay in the monolith (CV-03, CV-06). Seniors and volunteers do not use the dashboard. Family members are a role inside the mobile app, not a separate client.

## Consequences

**Benefits**

- One mobile codebase produces the Android and iOS apps (TC-07).
- The admin dashboard and the mobile app share one JSON contract. A business rule is implemented once, on the server.
- Administrator screens stay off the participant app.

**Disadvantages**

- The React Native app and the Next.js dashboard both have to follow the same API, including Arabic right-to-left layout.
- Participant tasks depend on the store apps. Seniors, volunteers, and family members have no public website.
- A change to the JSON contract is coordinated across both clients before a store release (CV-17).
