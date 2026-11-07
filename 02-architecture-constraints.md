# 2. Architecture Constraints

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 2: Architecture Constraints |
| **Status** | Draft for the final system |
| **Related section** | Section 1 defines the product, the pilot, and the requirements. This section records only the limits that already restrict design and implementation. |

A constraint is a limit the architecture has to respect. Some limits come from the budget, the product rules, or the assignment. Others are technology choices the team has accepted and now treats as binding. Choices that are still open are listed at the end so they are not mistaken for constraints.

The target described here is the final Wanas system. Around December 2027 the release is the React Native app and the Next.js admin dashboard on one rented server. They use the same API. Uploaded equipment photos stay on that machine.

## 2.1 Technical constraints

| ID | Constraint | Consequence for the architecture | Source |
| --- | --- | --- | --- |
| TC-01 | Production runs on one OVHcloud VPS-1, from $4.54 per month, starting around December 2027. The published plan is 2 vCores, 4 GB RAM, 40 GB NVMe SSD, unlimited traffic, and a daily backup of the previous 24 hours. | The admin dashboard, the API, PostgreSQL, and uploaded equipment photos share that machine. The React Native app runs on the phone and calls the API. A second server, a separate managed database, and a separate file host are outside this limit. The datacenter is chosen from OVHcloud’s locations. | Budget |
| TC-02 | Software on that machine is packaged and run with Docker. | The admin dashboard, the API, and the database are deployed as containers on the single host. | Team limit |
| TC-03 | PostgreSQL is the database. | Persistent data is relational and runs in a container on the same VPS as the application. | Team limit |
| TC-04 | The backend language is C# on .NET. | The API is one C# ASP.NET Core application in Docker. | Team limit |
| TC-05 | The public site name is `wanas-jo.com`. The first year is $6.48 with promo code NEWCOM648. The following price used for planning is $12.08 per year. The retail price is $15.98 per year. | The admin dashboard and the API are addressed through this domain. | Budget |
| TC-06 | The December 2027 release is the React Native app for seniors, volunteers, and family members, and the Next.js admin dashboard for administrators. | Both clients use the API on the same VPS. Senior tasks still need clear language, readable text, large controls, and few steps. | Product |
| TC-07 | Seniors, volunteers, and family members use one React Native app for Android and iOS. | One mobile codebase produces both store apps and calls the same API on the VPS. This app is part of the December 2027 release, together with the admin dashboard. | Team limit |
| TC-08 | The React Native app and the admin dashboard support Arabic and English. | Arabic screens use a right-to-left layout. User-facing text exists in both languages. | Product |
| TC-09 | Mobile notifications are delivered through Firebase Cloud Messaging and Apple Push Notification service. | The VPS sends notification requests to Google and Apple. The apps hold the corresponding platform credentials. | Team limit |
| TC-10 | Phone numbers and home addresses stay private until there is a clear need and consent. | Profiles, the API, and both clients have to keep contact details off public screens. | Product |
| TC-11 | Each person can do only what that person's role and granted permissions allow. | Senior, volunteer, family, and administrator access are separate. A family login never shares the senior's password. | Product |
| TC-12 | Publishing the mobile apps costs $25 once for Google Play and $100 per year for the Apple App Store. | Android and iOS releases require both store accounts. The yearly Apple fee continues for as long as the iOS app stays published. | Budget |

## 2.2 Organizational and political constraints

| ID | Constraint | Consequence for the architecture | Source |
| --- | --- | --- | --- |
| OC-01 | The known running costs are the VPS from $4.54 per month, the domain at the prices in TC-05, and the store fees in TC-12. | New paid infrastructure has to fit this small set of costs. | Budget |
| OC-02 | The React Native app and the Next.js admin dashboard go live around December 2027 on the same VPS and the same API. | The API, the dashboard, PostgreSQL, and uploaded equipment photos are on that machine for the release. | Product |
| OC-03 | The team can administer Linux on the VPS. | Deployment, container updates, and server administration are done by the team on that host. | Team |
| OC-04 | Wanas does not provide medical care, medication support, money handling, emergency response, or professional home care. | The system has no modules, payments, or workflows for those services. | Product |
| OC-05 | A family member has a separate account, linked to a senior only after consent, with permissions that can differ and can be withdrawn. | The data model records the link, the consent, and the granted permissions. | Product |
| OC-06 | A volunteer takes part in matching only after review. A connection becomes active only after the required people accept it. | Matching is a reviewed workflow with an acceptance record. Match suggestions stay inside the monolith and stay inactive until the required people confirm them. | Product |
| OC-07 | When in-person meetings are offered, they take place in approved public places or partner centers. | A visit record stores the agreed public arrangement and follows the safety rules. | Product |

## 2.3 Conventions

These rules apply to the Wanas system: how the code is written, how images are built, and how a release reaches the VPS. The Next.js admin dashboard, the .NET API, PostgreSQL, and the React Native app follow the same rules.

### Clean code

| ID | Convention | Consequence for the system |
| --- | --- | --- |
| CV-01 | Code uses the product language. | Modules and types are named for accounts, profiles, matches, interactions, consent, help requests, equipment listings, and reports. Those concepts stay separate, as required in Section 1. |
| CV-02 | Each module has one responsibility. | A change to matching does not require a change to registration, scheduling, or notifications. User-interface code renders state and calls the API. Business rules live in the application core. |
| CV-03 | Validation and authorization happen at the boundary. | The API checks input and permissions before it changes data. The admin dashboard and the mobile app can guide the user, and they do not become the place where those rules are defined. |
| CV-04 | Errors are explicit. | A failed request returns a clear result the client can show. Logs record the operation and the failure. Logs omit passwords, phone numbers, and home addresses. |
| CV-05 | Secrets stay out of the source tree. | Database passwords, API keys, and Firebase or Apple credentials are supplied when a container starts. |
| CV-06 | One API serves every client. | The admin dashboard and the React Native app call the same HTTP API. A business rule is implemented once, on the server. |

### Docker

| ID | Convention | Consequence for the system |
| --- | --- | --- |
| CV-07 | One container has one role. | The API, PostgreSQL, and the admin dashboard each have their own image. |
| CV-08 | Production images use a pinned version tag. | A release points at a specific image version. The running stack can be identified from that tag. |
| CV-09 | .NET images are built in stages. | The image that runs on the VPS contains the application runtime, which keeps the cheap server's disk for PostgreSQL data and uploads. |
| CV-10 | Database files and equipment photos are stored on Docker volumes. | Replacing the API or database container leaves PostgreSQL data and uploaded photos on the VPS disk. |
| CV-11 | Only the public edge is published on the host. | The VPS opens the web ports. PostgreSQL is reachable from the application containers on the Docker network. |
| CV-12 | Container settings come from the environment. | Image builds stay free of machine-specific connection strings, domain names, and keys. The VPS supplies those values at start. |

### Deployment

| ID | Convention | Consequence for the system |
| --- | --- | --- |
| CV-13 | Production is one Docker Compose stack on the VPS. | The team deploys by updating that stack on the single host from TC-01. The compose file is the description of what is running. |
| CV-14 | Public traffic uses HTTPS on `wanas-jo.com`. | The admin dashboard and the API are served on the domain from TC-05 with TLS. |
| CV-15 | A release applies database migrations, then starts the new application container. | The PostgreSQL schema matches the API version that begins serving requests. |
| CV-16 | Each deployment uses an identified image version, and the previous version is kept. | The team can start the previous image again from the same compose file. |
| CV-17 | The December 2027 release runs the API and the admin dashboard on the VPS, and publishes the React Native app against that API. | Android and iOS builds call the API already running on `wanas-jo.com`. |

## 2.4 Decisions

The following choices are closed for this design.

| Topic | Decision |
| --- | --- |
| Backend language | C# on .NET is decided. The API is one ASP.NET Core application. |
| Clients | The admin UI is Next.js. Participants use React Native. Both call the API on the VPS. There is no public website for seniors or volunteers. |
| Matching | Match suggestions stay inside the monolith and stay inactive until the required people confirm them. |
| Help and equipment | Helping Hands and Community Exchange are in this design. |

In-app chat and in-app voice or video remain later and outside this constraint list.

One choice remains open.

| Topic | Current position |
| --- | --- |
| Server location | OVHcloud VPS-1 fixes the machine size in TC-01. The datacenter is chosen from OVHcloud’s locations. Hosting in Jordan is not required. |

## 2.5 References

- Section 1, `01-introduction-goals-requirements.md`, especially the MVP boundary, NFR-02, NFR-03, NFR-09, and NFR-11.
- Task 01 brief: distinguish genuine constraints from decisions the team is free to make.
- arc42 Section 2, Architecture Constraints: https://docs.arc42.org/section-2/
- OVHcloud VPS, including VPS-1 from $4.54 per month: https://us.ovhcloud.com/vps/
- Google Play Console registration, US$25 one-time fee: https://support.google.com/googleplay/android-developer/answer/6112435?hl=en
- Apple Developer Program, 99 USD per membership year. TC-12 records this as $100 per year: https://developer.apple.com/programs/enroll/
