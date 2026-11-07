# ADR 0001. One C# monolith on one VPS

| | |
| --- | --- |
| **Status** | Accepted |
| **Date** | 2026-11-07 |

## Context

Production runs on one OVHcloud VPS-1. The published plan is 2 vCores, 4 GB RAM, and 40 GB NVMe SSD (TC-01). Docker Compose is how that machine is run (TC-02, CV-13). The JSON API, the companionship rules, persistence, and reminders all have to fit on that host.

## Alternatives

| Alternative | Why it was set aside |
| --- | --- |
| Microservices, one process per bounded context | Extra processes and network hops would spend the 2 vCores and 4 GB RAM. The seven companionship modules already share one database. |
| A second server, or a managed database outside this host | TC-01 keeps the API, the database, and uploaded files on the one VPS. A second machine sits outside the known running costs (OC-01). |

## Decision

The backend is one C# monolith on the OVHcloud VPS-1. The JSON API, the domain rules, Entity Framework, and the in-process worker are one process. Docker Compose on that same machine also runs the Next.js admin dashboard, PostgreSQL, and a Caddy TLS proxy.

## Reasons

One language holds the domain, the API, Entity Framework, and the worker, and that set runs as one process on 4 GB (Section 4.2 and 4.3). The bounded contexts stay modules inside that process. Caddy terminates TLS for `wanas-jo.com` (CV-14). PostgreSQL is reachable on the Docker network and is not published on the host (CV-11).

## Consequences

**Benefits**

- The team deploys one Compose stack and can name the running version from that file (CV-13, CV-16).
- Reminders and push reuse the same use cases and the same database as the API.
- A change to a matching rule stays in the domain module and ships with the same process.

**Disadvantages**

- The API, the dashboard, PostgreSQL, the worker, and the proxy share 2 vCores and 4 GB RAM. A reminder run competes with request handling.
- A fault in the monolith process stops the API and the worker together.
- The host is one machine. A host outage stops the API, the dashboard, and the database together, as Section 11 records.
