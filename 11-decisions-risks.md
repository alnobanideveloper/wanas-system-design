# 11. Architecture Decisions, Trade-offs, and Risks

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 11: Architecture Decisions, Trade-offs, and Risks |
| **Status** | Draft for the final system |
| **Related section** | Section 4 records the solution strategy. This section records each accepted decision, what it gave up, and the risks that remain. |

This section describes the final Wanas companionship system. The decisions below are accepted. Each record states the context, the alternatives set aside, and the consequences. The records live in [adr](adr/).

## 11.1 Architecture decisions

| ADR | Decision |
| --- | --- |
| [ADR 0001](adr/0001-monolith-on-one-vps.md) | One C# monolith on the single OVHcloud VPS-1: the API, domain rules, Entity Framework, and the in-process worker are one process. |
| [ADR 0002](adr/0002-react-native-and-admin-dashboard.md) | One React Native app for seniors, volunteers, and family members, and a separate Next.js dashboard for administrators. |
| [ADR 0003](adr/0003-postgresql-entity-framework.md) | PostgreSQL with Entity Framework Core. A senior has one match row. A volunteer can appear on many. |
| [ADR 0004](adr/0004-confirmed-match-suggestions.md) | Suggestions are computed in the monolith and stay inactive until an administrator confirms them and both people accept. |

## 11.2 Trade-offs

| Decision | Gained | Given up |
| --- | --- | --- |
| [ADR 0001](adr/0001-monolith-on-one-vps.md) | One process and one Compose stack on 2 vCores and 4 GB RAM. The API, the rules, Entity Framework, and the worker deploy together, with the dashboard, PostgreSQL, and the Caddy TLS proxy on the same host. | Microservices and a second server. The parts of the stack share one machine and one memory budget. |
| [ADR 0002](adr/0002-react-native-and-admin-dashboard.md) | One Android and iOS app, one admin dashboard, and one JSON API. Arabic and English, including right-to-left layout. | A public website for the three participant roles, Razor or Blazor screens inside the monolith, and separate Swift and Kotlin apps. |
| [ADR 0003](adr/0003-postgresql-entity-framework.md) | Relational records for accounts, matches, acceptances, consent, help requests, and equipment. EF Core stays in the infrastructure layer and produces the migrations. One senior, one match row. | A second database server, and Dapper with hand-written SQL. The volunteer side of a match has no cap. |
| [ADR 0004](adr/0004-confirmed-match-suggestions.md) | Suggestions from shared interests, language, and area, kept inactive until an administrator confirms the pair and the senior and the volunteer both accept. | A fully manual match with no suggestions, a suggestion that becomes active on its own, and a separate matching service. |

## 11.3 Risks, limitations, and unresolved questions

- The single VPS is a single point of failure. A host outage stops the API, the dashboard, and the database together.
- WhatsApp must deliver the registration code. If it refuses, no account is created.
- A rejected push leaves the call scheduled and marks the reminder failed. The people can miss the reminder.
- The home pin is stored for area matching. A bug in a response could expose it. Responses about another person are supposed to return `areaName` only.
- Equipment handover is recorded in Wanas. The physical meeting still happens outside the software, in a public arrangement.
- One volunteer can be matched to many seniors, which can overload that volunteer. The product does not yet cap how many seniors one volunteer holds.
- In-app chat and in-app voice or video are not in this design. Contact stays a scheduled phone call.

These questions are still open:

- Which OVHcloud datacenter hosts the VPS-1. Section 2 leaves the location open. Hosting in Jordan is not required.
- How many seniors one volunteer may hold. ADR 0003 allows many match rows for one volunteer and sets no maximum.

## 11.4 References

- Section 4, `04-solution-strategy.md`, especially 4.2 and 4.3.
- Section 2, `02-architecture-constraints.md`, for the single VPS, PostgreSQL, React Native, and match acceptance.
- Section 7, `07-data-architecture.md`, for the one-senior match row and equipment handover.
- arc42 Section 11, Risks and Technical Debt: https://docs.arc42.org/section-11/
- ADR templates: https://adr.github.io/adr-templates/
