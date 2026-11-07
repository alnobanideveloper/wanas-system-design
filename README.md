# Wanas (ونس)

Wanas connects older adults with volunteers for regular companionship, and it gives families a consented way to help. Seniors, volunteers, and family members use one React Native app. Administrators use a Next.js dashboard. One C# API and PostgreSQL run on a single server at `wanas-jo.com`.

This repository is the system design for that product. It is the engineering blueprint for implementation. It does not contain the application source.

## What is in this design

- Accounts for seniors, volunteers, family members, and administrators
- Profiles, private home pins, and area-based matching
- Match suggestions that stay inactive until an administrator confirms them and both people accept
- Scheduled phone calls and push reminders
- Family links with consent and separate permissions
- Everyday help requests in four non-professional categories
- Equipment listings to donate, lend, or request, with a recorded handover
- Safety reports and an administration record

In-app chat and in-app voice or video are outside this design. The phone call itself stays on the normal telephone network.

## How to read the documentation

| Path | Contents |
| --- | --- |
| [docs/SYSTEM_DESIGN.md](docs/SYSTEM_DESIGN.md) | The architecture document, arc42 sections 1–12 |
| [docs/diagrams/](docs/diagrams/) | Context, container, component, sequence, data, and deployment diagrams, with editable sources |
| [docs/adr/](docs/adr/) | Architecture decision records |
| [docs/api/api.md](docs/api/api.md) | API inventory and request/response contracts |
| [docs/references.md](docs/references.md) | Sources cited in the document |
| [Wanas-System-Design.pdf](Wanas-System-Design.pdf) | The same architecture document, exported for review |

Start with section 1 for the product, section 3 for the system boundary, and section 4 for the technical approach. The deployment picture is in section 9. The decisions and risks are in section 11.

## Review questions

The design is written so a reviewer can ask:

- Why this architecture?
- How do the components communicate?
- What happens when something fails?
- How is data structured and protected?
- Which alternatives were rejected, and why?
- What would need to change if the project grows?
- What is the greatest technical risk?
