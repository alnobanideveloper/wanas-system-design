# ADR 0003. PostgreSQL with Entity Framework Core

| | |
| --- | --- |
| **Status** | Accepted |
| **Date** | 2026-11-07 |

## Context

Wanas stores accounts, profiles, matches, acceptances, consent, schedules, help requests, and equipment. A senior has one volunteer. A volunteer can support many seniors. A call, a help request, or an equipment handover points at a known person. TC-03 fixes PostgreSQL on the same VPS as the API. CV-15 applies migrations before the new API container starts.

## Alternatives

| Alternative | Why it was set aside |
| --- | --- |
| A second database server | TC-01 and TC-03 put PostgreSQL in a container beside the API. A separate database host is outside this limit. |
| Dapper with hand-written SQL | The aggregates would be the same. Hand-written SQL would not produce the migrations that CV-15 applies, and it would sit beside the domain. |

## Decision

PostgreSQL holds the relational records for accounts, matches, acceptances, consent, help requests, and equipment. Entity Framework Core maps those records, stays in the infrastructure layer, and produces the migrations. `Match.SeniorProfileId` is unique, so a senior has one match row. `Match.VolunteerProfileId` is not unique, so a volunteer can appear on many match rows.

## Reasons

Foreign keys keep a match, an acceptance, a family-consent link, a help request, and an equipment handover pointed at a real account or profile. The domain does not reference Entity Framework, so a table change stays in infrastructure (Section 4.2). A release applies the EF migrations, then starts the new application container (CV-15).

## Consequences

**Benefits**

- One current volunteer per senior is a unique key, not only a check in a screen.
- Consent, help requests, and equipment listings stay separate tables with owners and statuses.
- The schema that starts serving requests is the schema the running API was migrated to.

**Disadvantages**

- PostgreSQL shares the VPS disk and the daily backup of the previous 24 hours with the API and uploaded files.
- Application queries go through Entity Framework in the infrastructure layer.
- The schema allows many match rows for one volunteer. It does not cap how many seniors that volunteer holds.
