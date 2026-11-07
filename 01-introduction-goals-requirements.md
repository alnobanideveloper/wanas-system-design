# 1. Introduction, Goals & Requirements

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 1: Introduction, Goals & Requirements |
| **Status** | Draft for the final system |
| **Initial context** | Jordan |
| **Primary focus** | The full Wanas system: one React Native app for seniors, volunteers, and family members, and a Next.js admin dashboard. The first operational pilot can start with companionship. |

## 1.1 Project Overview

Wanas is a community platform that connects older adults with young volunteers to build meaningful relationships through regular conversations, phone calls, and, where appropriate, in-person meetings.

The platform is based on the idea that support for older adults is not only about providing services. It is also about listening, sharing experiences, exchanging knowledge between generations, and helping people feel connected to their community.

Wanas will help older adults find compatible volunteers based on shared interests, location, language, and availability. It will also provide a simple way to arrange recurring calls and visits. It includes practical help requests from older adults and their families, and the free sharing or lending of useful equipment. The first operational pilot can start with companionship.

Seniors, volunteers, and family members use one React Native app. Administrators use a separate Next.js dashboard. Arabic and English are supported, including a right-to-left layout for Arabic. There is no public website for seniors or volunteers.

Wanas is intended to be a community-support platform, not a medical-care, nursing, emergency-response, or financial service.

## 1.2 Problem Statement

Some older adults may experience loneliness, limited social interaction, or difficulty finding regular opportunities to connect with people outside their usual circle. Their families may also want them to have more social contact and support, but may not always have enough time or access to suitable community programs.

At the same time, young people may be willing to volunteer, listen, share their skills, and learn from older generations, but they may not have a simple and trusted way to connect with older adults and maintain regular contact.

Existing informal arrangements can be difficult to organize. A connection may depend on personal contacts, and calls or visits may stop when there is no clear schedule or follow-up.

Wanas aims to address this gap by providing a simple way to:

- Connect older adults with suitable volunteers.
- Help both sides build and maintain regular contact.
- Arrange calls and visits in a clear, organized way.
- Allow seniors and authorized family members to request suitable everyday help.
- Encourage the community to share useful equipment.

The project must also address trust, privacy, consent, reliability, and personal safety. These are important parts of the service, not optional additions.

## 1.3 Project Goals

The main goals of Wanas are:

| ID | Goal | Expected result |
| --- | --- | --- |
| G-01 | Reduce social isolation through regular human contact. | Seniors can build ongoing relationships with volunteers. |
| G-02 | Create suitable connections between generations. | Matching considers interests, location, language, and availability. |
| G-03 | Make companionship easier to organize. | Users can arrange and remember calls and visits. |
| G-04 | Give seniors a simple way to request suitable everyday help. | Requests can be sent to a matched volunteer or an approved volunteer community. |
| G-05 | Involve families without removing the senior's independence. | Family members can help through separate accounts and controlled permissions. |
| G-06 | Encourage community sharing. | People can offer or request useful equipment for free when this service is introduced. |
| G-07 | Build a safe and trusted service. | Users can report concerns, request a new match, and receive support from administrators. |
| G-08 | Validate the idea with limited resources. | A small pilot can test the service before major technical investment. |

## 1.4 Target Users and Stakeholders

### 1.4.1 Seniors

Older adults in Jordan who are interested in social interaction, companionship, or suitable everyday community support.

They should be able to use Wanas with as little technical effort as possible. Some seniors may prefer help from a family member or a coordinator when registering or using the platform.

Main needs:

- Find a volunteer with shared interests.
- Choose how and when to communicate.
- Arrange regular calls or visits.
- Ask for suitable help.
- Feel respected and in control of their participation.
- Stop a connection or request a different volunteer if they feel uncomfortable.

### 1.4.2 Volunteers

People who are willing to give time to older adults. The initial project assumption focuses on young volunteers aged 18–35.

Volunteers may offer conversation, listening, shared activities, knowledge exchange, and approved non-professional help.

Main needs:

- Create a profile and state their interests and availability.
- Review and accept or decline match requests.
- Arrange calls and visits.
- View and respond to suitable help requests.
- Understand the service rules and their boundaries.
- Report concerns and manage their volunteering commitments.

### 1.4.3 Family Members

Relatives or trusted people who want to help a senior use Wanas.

A family member has a separate account linked to the senior's profile. The senior remains the main person receiving the service. Family access is controlled by the senior's consent and the permissions granted.

Main needs:

- Help a senior register and complete their profile.
- Suggest or help review volunteer matches.
- Submit help requests on the senior's behalf.
- Coordinate calls or visits where permitted.
- Follow request status and receive relevant updates.

Family members do not automatically gain access to private conversations or the right to act as the senior. These permissions must be handled separately and clearly.

### 1.4.4 Administrators

Wanas team members who manage the platform and support its users.

Main needs:

- Review and verify volunteer registrations.
- Manage senior and volunteer accounts.
- Create, review, and manage matches.
- Help resolve matching problems.
- Review reports and safety concerns.
- Monitor help requests and equipment listings.
- View basic platform activity and pilot results.

### 1.4.5 Other Stakeholders

- **Local community centers and NGOs:** Potential partners who can help reach seniors, support registration, or provide safe meeting locations.
- **Universities and student groups:** Potential sources of volunteers and project support.
- **Corporate CSR programs and grant providers:** Possible future sources of financial support.
- **Senior families and caregivers:** Important stakeholders in trust, consent, and coordination.

Partnerships and funding are possible future directions, not confirmed dependencies for the initial MVP.

## 1.5 User Roles and Account Model

Wanas has four main roles:

| Role | Account model | Main responsibility |
| --- | --- | --- |
| Senior | Individual account and senior profile | Receives companionship and support. |
| Volunteer | Individual account and volunteer profile | Provides time and approved support. |
| Family member | Individual account linked to one or more senior profiles | Supports a senior with permission. |
| Administrator | Staff account with administrative permissions | Operates and supervises the platform. |

All normal users can use one shared authentication system. Seniors, volunteers, and family members use the React Native app. Administrators use the Next.js dashboard. The role and permissions assigned to each account determine which screens and actions are available.

Family access model:

- The family member uses their own login details; they do not share the senior's password.
- A family member must be linked to a senior profile through an approval or consent process.
- A senior may have more than one linked family member.
- Different family members may receive different permissions.
- Access can be changed or withdrawn.
- Private conversations remain private unless the senior explicitly chooses otherwise or a clearly defined safeguarding process requires action.

The exact process for confirming consent, including cases where a senior cannot provide informed consent, must be defined before real users are onboarded.

## 1.6 Functional Requirements

The following requirements describe what the system should do. They are written so that they can later be checked through testing.

**Priority meanings:**

- **High:** Required for the initial MVP.
- **Mid:** Valuable, but can follow after the core pilot.
- **Low:** A possible later improvement.

### 1.6.1 Registration and Profiles

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-01 | The system shall allow seniors, volunteers, and family members to create individual accounts. | High |
| FR-02 | The system shall provide a separate profile type for seniors, volunteers, and family members. | High |
| FR-03 | The system shall allow users to update their own profile information. | High |
| FR-04 | A senior profile shall include basic information, location, interests, preferred language, and communication preferences. | High |
| FR-05 | A volunteer profile shall include basic information, location, interests, languages, availability, and preferred volunteering activities. | High |
| FR-06 | The system shall allow a family member to request a link to a senior profile. | Mid |
| FR-07 | The system shall record whether the senior has approved the family link and what permissions were granted. | Mid |
| FR-08 | The system shall allow administrators to review volunteer registrations before they can participate in matching. | High |

### 1.6.2 Matching and Connections

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-09 | The system shall allow administrators to find suitable volunteers for a senior using profile information. | High |
| FR-10 | Matching shall consider shared interests, location, language, availability, and communication preferences. | High |
| FR-11 | The system shall allow a proposed match to be reviewed before the connection becomes active. | High |
| FR-12 | The system shall record whether the senior and volunteer have accepted the connection. | High |
| FR-13 | The system shall allow a user to request a different match or report an uncomfortable connection. | High |
| FR-14 | The system shall allow administrators to review and manage match changes. | High |
| FR-15 | The initial pilot shall support manual or semi-manual matching rather than depending on a complex automated algorithm. | High |
| FR-16 | The system shall suggest volunteers who share a senior's interests and live nearby. A suggestion shall not become an active match until the required people confirm it. | Mid |

### 1.6.3 Communication and Scheduling

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-17 | The system shall allow users in an active match to see the agreed communication arrangements. | High |
| FR-18 | The system shall support arranging a date and time for a phone call. | High |
| FR-19 | The system shall support arranging an in-person visit or activity, subject to the platform's safety rules. | Mid |
| FR-20 | The system shall send reminders for scheduled calls or meetings through the notification method available in the MVP. | Mid |
| FR-21 | The system shall allow users to cancel or reschedule an agreed interaction. | Mid |
| FR-22 | The system shall record basic interaction status, such as scheduled, completed, cancelled, or missed. | Mid |
| FR-23 | The system shall provide chat between a senior and a volunteer in an active match. | Mid |
| FR-24 | The system shall provide voice or video calls between a senior and a volunteer in an active match. | Mid |
| FR-25 | The system shall keep an active match open after a completed call or visit so the pair can schedule further contact. | High |

### 1.6.4 Helping Hands

Helping Hands is a structured way for a senior to ask for everyday, non-professional assistance from the matched volunteer or the wider approved volunteer community. It remains in this system. The first operational pilot can start with companionship.

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-26 | The system shall allow a senior to create a help request in an approved category: grocery shopping, transportation or accompaniment to an appointment, technology and smartphone help, or a small everyday errand. | Mid |
| FR-27 | The system shall allow the requester to send the request to the matched volunteer or make it available to approved volunteers in the wider community. | Mid |
| FR-28 | A volunteer shall be able to accept or decline a help request. | Mid |
| FR-29 | A volunteer who accepted a request shall be able to mark it completed. | Mid |
| FR-30 | The system shall show the request status to the requester, such as open, accepted, completed, or cancelled. | Mid |
| FR-31 | A family member with permission shall be able to create a help request on behalf of a linked senior. | Mid |
| FR-32 | Help requests shall stay within approved non-professional tasks. The system shall not offer medical care, medication, money handling, or emergency response through a help request. | Mid |

### 1.6.5 Community Exchange

Community Exchange is a free equipment listing service for seniors. It remains in this system. The first operational pilot can start with companionship.

Possible listing types:

- **Donate:** Give an item permanently.
- **Lend:** Allow someone to use an item temporarily and return it.
- **Request:** Ask the community for an item that is needed.

Examples include walkers, wheelchairs, crutches, other mobility aids, medical beds, and other non-medication equipment.

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-33 | A contributor shall be able to submit an equipment listing with a description, photos, condition, location, and listing type. | Mid |
| FR-34 | A user shall be able to browse available equipment and submit a request. | Mid |
| FR-35 | The system shall show an item's condition and whether it is available, reserved, collected, transferred, or no longer available. | Mid |
| FR-36 | Administrators shall be able to review, hide, or remove equipment listings. | Mid |
| FR-37 | The people involved in a listing shall be able to arrange collection of the item, and the system shall record that arrangement. | Mid |

### 1.6.6 Administration and Safety

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-38 | The system shall provide an administrator interface for managing users and profiles. | High |
| FR-39 | Administrators shall be able to review volunteer verification information. | High |
| FR-40 | Users shall be able to report a safety or behavior concern. | High |
| FR-41 | Administrators shall be able to review reports and record the action taken. | High |
| FR-42 | The system shall allow administrators to suspend or deactivate an account when necessary. | High |
| FR-43 | The system shall support cancelling a match and arranging a new match. | High |
| FR-44 | The system shall keep a basic record of important administrative actions. | Mid |

### 1.6.7 Family Circle

Family Circle lets a family member take part with the senior's consent. The family member uses the React Native app through a consented link. Registration help can be done manually in the first operational pilot.

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-45 | A family member with the senior's consent shall be able to update the linked senior's profile within the permissions granted. | Mid |
| FR-46 | A family member shall be able to view the linked senior's scheduled calls, visits, and help-request statuses allowed by their permissions. | Mid |
| FR-47 | A family member shall receive notifications about important updates they are permitted to see. | Mid |
| FR-48 | A family member with permission shall be able to help coordinate a scheduled call or visit for the linked senior. | Mid |

A family member creates a help request for the senior through FR-31. Linking the account and recording consent are FR-06 and FR-07.

## 1.7 Main Use Cases

The following use cases describe the most important ways people will use Wanas.

### UC-01: Register as a Senior

**Primary actor:** Senior

**Supporting actor:** Family member or administrator, if needed

1. The senior or an authorized helper opens the registration process.
2. Basic information and preferences are entered.
3. The senior's location, interests, language, and preferred communication method are recorded.
4. The senior reviews the information and confirms participation through the defined consent process.
5. The profile becomes available for matching after any required review.

**Expected result:** A senior profile is created and is ready for matching.

### UC-02: Register and Verify a Volunteer

**Primary actor:** Volunteer

**Supporting actor:** Administrator

1. The volunteer creates an account.
2. The volunteer enters their interests, location, languages, and availability.
3. The volunteer agrees to Wanas rules and boundaries.
4. The administrator reviews the registration and verification information.
5. The volunteer is approved, rejected, or asked for more information.

**Expected result:** Only approved volunteers can take part in active matching.

### UC-03: Match a Senior with a Volunteer

**Primary actor:** Administrator

**Supporting actors:** Senior and volunteer

1. The system provides the information needed to compare suitable profiles.
2. The administrator reviews possible volunteers based on the matching factors.
3. A proposed connection is shared with the relevant people.
4. The senior and volunteer accept or decline.
5. The system records the result.
6. If both accept, the connection becomes active.

**Expected result:** An active match is created only after the required acceptance steps.

### UC-04: Arrange a Recurring Call

**Primary actors:** Senior and volunteer

1. The senior and volunteer agree on a suitable day and time.
2. The call is recorded as a scheduled interaction.
3. The platform sends reminders when supported.
4. After the planned time, the interaction can be marked as completed, cancelled, or missed.
5. The next call can be arranged.

**Expected result:** Both sides have a clear schedule for regular contact.

### UC-05: Request Everyday Help

**Primary actor:** Senior or authorized family member

**Supporting actors:** Volunteer and administrator

1. The requester selects a help category and describes what is needed.
2. The requester chooses a personal request or community request, where available.
3. The request is sent to the matched volunteer or made available to suitable approved volunteers.
4. A volunteer accepts or declines.
5. The request is assigned and its status is updated.
6. The requester and volunteer coordinate the agreed details.
7. The request is marked as completed or cancelled.

**Expected result:** The request has a clear owner and status, and its progress can be followed.

### UC-06: Link a Family Member to a Senior

**Primary actors:** Family member and senior

1. The family member creates their own account.
2. The family member requests access to the senior's profile.
3. The senior approves the link and selects the allowed permissions through the defined process.
4. The system records the relationship and permissions.
5. The family member can perform only the approved actions.

**Expected result:** A family member can support a senior without sharing login details or receiving automatic access to private information.

### UC-07: Report a Concern and Request a New Match

**Primary actor:** Senior, family member with permission, or volunteer

**Supporting actor:** Administrator

1. A user submits a report or requests a new match.
2. The system records the report and notifies the responsible administrator.
3. The administrator reviews the information and contacts the relevant people when appropriate.
4. The administrator takes suitable action, which may include pausing contact, ending the match, or suspending an account.
5. The outcome is recorded.

**Expected result:** Users have a clear way to raise concerns and the team can respond.

### UC-08: Offer or Request Equipment

**Primary actors:** Community contributor and senior or family member

**Supporting actor:** Administrator

1. A contributor submits an item listing or a user submits an equipment request.
2. The listing is reviewed according to the equipment rules.
3. An interested person requests the item.
4. The parties agree on a safe handover or return arrangement.
5. The listing status is updated.

**Expected result:** Equipment can be offered or requested through a controlled process when the feature is enabled.

## 1.8 Non-Functional Requirements

Non-functional requirements describe how well and how safely the system should work.

| ID | Quality area | Requirement |
| --- | --- | --- |
| NFR-01 | Usability and accessibility | Main senior tasks shall use clear language, readable text, large controls, and as few steps as practical. |
| NFR-02 | Privacy | Personal phone numbers and home addresses shall not be shown publicly or shared before there is a clear need and appropriate consent. |
| NFR-03 | Authorization | Users shall only access information and actions allowed by their role and granted permissions. |
| NFR-04 | Consent | The platform shall record the required consent for participation and family access. |
| NFR-05 | Security | Passwords and sensitive information shall be protected using appropriate security practices. |
| NFR-06 | Safety | The platform shall provide a way to report concerns, request a new match, and contact the responsible team. |
| NFR-07 | Reliability | The system shall store important profile, match, and request changes so that they are not lost during normal use. |
| NFR-08 | Maintainability | The system shall keep user accounts, profiles, matches, interactions, and requests as clear, separate concepts. |
| NFR-09 | Cost | The MVP shall use a low-cost approach and avoid infrastructure or integrations that are not needed to validate the idea. |
| NFR-10 | Scalability | The design should allow more seniors, volunteers, neighborhoods, and partner organizations to be added without redesigning the whole system. |
| NFR-11 | Compatibility | Seniors, volunteers, and family members shall use one React Native app. Administrators shall use a Next.js dashboard. Arabic and English, including a right-to-left layout, shall be supported. |
| NFR-12 | Accountability | Important actions, including match changes and safety decisions, should be recorded for later review. |

### Proposed measurable quality targets

These are initial targets for the pilot, not proven performance results. The team should review them after testing with real users.

| ID | Proposed target | How it can be checked |
| --- | --- | --- |
| QT-01 | A senior or family helper should be able to complete the basic registration flow in no more than 5 minutes with guidance available. | Observe pilot registration sessions. |
| QT-02 | A user should be able to submit a basic help or match-related request in no more than 3 minutes. | Time a usability test. |
| QT-03 | The main pages should load within 3 seconds under normal pilot conditions. | Measure page-load time during testing. |
| QT-04 | Every active match in the pilot should have recorded acceptance from both sides. | Review match records. |
| QT-05 | Every reported safety concern should have a recorded review and outcome. | Review the report log. |
| QT-06 | No user should be able to access another user's private information without permission. | Run role and access-control tests. |

## 1.9 Core Features and Priorities

The first operational pilot can start with companionship. The system documented in this architecture includes the React Native app, the Next.js admin dashboard, family links, help requests, and equipment listings.

### MVP — Initial Pilot

The first operational pilot can start with companionship and is intended to validate that model with limited resources. The assumptions document proposes starting with approximately five senior–volunteer pairs. Family links, help requests, and equipment listings are part of the documented system.

| Feature | MVP priority | Notes |
| --- | --- | --- |
| Senior and volunteer registration | High | Registration in the React Native app. Arabic and English, including a right-to-left layout. |
| Senior and volunteer profiles | High | Capture interests, location, language, and availability. |
| Volunteer review and approval | High | Manual review by the Wanas team. |
| Matching | High | Manual or semi-manual matching. |
| Match acceptance | High | Both sides should agree before contact begins. |
| Regular phone calls | High | Can use normal phone calls or an agreed external channel. |
| Call schedule and follow-up | High | A simple schedule and manual reminders may be enough at first. |
| Safety rules and reporting process | High | Include a clear way to raise concerns and request a new match. |
| Basic administrator tools | High | The team needs to manage users and matches during the pilot. |
| Personalized match suggestions | Mid | Suggestions use shared interests and nearby location. People still confirm the match. |
| In-app chat | Mid | After the core pilot. The pilot uses an ordinary phone call. |
| In-app voice or video calls | Mid | After the core pilot. |
| In-person visits and activities | Mid | Use approved public places or partner centers and clear safety procedures. |
| Helping Hands | Mid | Grocery shopping, transport or accompaniment, technology help, and small errands. Non-professional tasks only. |
| Community Exchange | Mid | Free donate, lend, or request. Show condition and availability, and arrange collection. |
| Family Circle | Mid | With consent: update the senior profile, request help, view schedules and statuses, receive updates, and help coordinate visits. Registration support can be manual in the pilot. |

The MVP does not need a complex matching algorithm, in-app calling, or a large automated workflow. The team can manage some activities manually while learning from the pilot.

### Later Versions

Family links, help requests, and equipment listings are already in the documented system. In-app chat and in-app voice or video stay later.

After the first operational pilot, Wanas may add:

- In-app messaging and, if justified, voice or video calls.
- Volunteer hours, recognition, and certificates.
- Community activities and group events.
- Partner or NGO dashboards.
- More automated matching and reporting.

## 1.10 Project Boundaries and Out-of-Scope Features of MVP

### What Wanas Is

- A platform for companionship and community connection.
- A way to organize regular contact between seniors and volunteers.
- A way to coordinate approved, non-professional everyday help.
- A community initiative that may support free equipment sharing.
- A platform that involves families while respecting the senior's independence.

### What Wanas Is Not

The following are outside the project's service boundaries, especially in the MVP:

- **Medical care:** Volunteers do not diagnose conditions, prescribe medication, provide treatment, or replace healthcare professionals.
- **Emergency response:** Wanas is not an emergency service. Users must contact the appropriate emergency services when urgent help is needed.
- **Financial assistance or transactions:** The platform will not handle loans, cash transfers, payments, or purchases using volunteer money.
- **Professional home care:** Volunteers are not a replacement for nurses, caregivers, or other trained professionals.
- **Unsupervised private meetings:** Early in-person pilot meetings should take place only in approved public places or partner centers, following the safety process.
- **Public website for seniors or volunteers:** Seniors, volunteers, and family members use the React Native app. Administrators use the Next.js dashboard.
- **Fully automated matching in the pilot:** Matching can be managed manually or semi-manually.
- **Guaranteed prevention of loneliness:** Wanas can create opportunities for social contact, but it cannot promise a specific psychological or medical outcome.

### Project Boundary Summary

Wanas is responsible for providing the platform, setting participation rules, managing its matching process, protecting user information, and responding to reports through a defined process. It should not promise services that it cannot safely provide or control.

A disclaimer or liability waiver does not replace responsible operation, reasonable safety measures, or the need to review applicable legal requirements.

## 1.11 Assumptions and Points to Validate

The project is based on assumptions, not all of which have been proven. The assumptions document highlights user behavior, trust, safety, operating cost, and long-term sustainability as areas that need attention.

### A. Assumptions to Test First

| ID | Assumption | Why it matters | How to validate it |
| --- | --- | --- | --- |
| A-01 | Older adults in Jordan who feel lonely are open to regular social interaction with people they do not already know. | The service depends on seniors being willing to participate. | Interview older adults and test a small number of introductions. |
| A-02 | Seniors will not feel embarrassed or disrespected by contact from young volunteers. | The service must feel like companionship and knowledge exchange, not pity. | Test different ways of explaining Wanas and collect honest feedback. |
| A-03 | Volunteers aged 18–35 are willing to commit to weekly calls. | Regular contact depends on volunteer reliability. | Run a short pilot and track attendance and dropouts. |
| A-04 | Families will trust a student-led platform enough to allow contact with their older relatives. | Family trust may affect participation. | Speak with families, explain verification and safety steps, and record concerns. |
| A-05 | Short, regular phone calls can help seniors feel more socially connected. | This is a central expected benefit. | Ask participants about their experience before and after the pilot; do not treat this as a medical outcome. |
| A-06 | Seniors or their families can complete a simple digital registration process. | Registration must not become a barrier. | Observe real users completing the process and offer assisted registration. |
| A-07 | Volunteers can maintain professional boundaries and avoid unauthorized medical or financial advice. | Clear boundaries are essential for safety. | Provide rules and orientation, then monitor questions and reports during the pilot. |
| A-08 | Manual or semi-manual matching is enough for the pilot. | A complex algorithm may not be needed at the beginning. | Match a small group manually and record the time and quality of each match. |
| A-09 | Starting with around five senior–volunteer pairs is enough to identify early problems. | A small pilot keeps the workload manageable. | Run the pilot, document issues, and decide whether a larger test is justified. |
| A-10 | Early in-person meetings can be managed more safely when limited to public places or partner centers. | In-person contact creates additional safety concerns. | Review the meeting process with participants and relevant local partners before allowing meetings. |

### B. Assumptions About the Proposed Solution and Safety

| ID | Assumption | Validation or action needed |
| --- | --- | --- |
| A-11 | Seniors, volunteers, and family members can use one React Native app, and administrators can use the Next.js dashboard. | Watch a senior complete registration in the app in Arabic and in English, including a right-to-left layout, and watch an administrator complete a review in the dashboard. |
| A-12 | Avoiding financial handling and medical services will reduce major legal and ethical risks. | Keep these services outside the scope and seek appropriate advice on the remaining responsibilities. |
| A-13 | Hiding personal phone numbers and home addresses will improve privacy. | Define exactly what information is visible, when it can be shared, and how users communicate. |
| A-14 | A family-support mode can reassure relatives without removing senior independence. | Test consent, permissions, and the family experience with seniors and families. |
| A-15 | Re-matching and incident reporting can help resolve unsuitable connections. | Define who reviews reports, how quickly they are handled, and what actions can follow. |
| A-16 | Free or low-cost web tools can support the first MVP. | Check that selected tools meet basic privacy, security, and reliability needs. |
| A-17 | Presenting Wanas as an exchange of experiences between generations may reduce feelings of pity. | Test the message with seniors and adjust it based on their feedback. |
| A-18 | Equipment sharing can be managed safely through a controlled donation or loan process. | Define item checks, cleaning, handover, returns, records, and responsibility before launch. |

### C. Assumptions About Future Growth

These assumptions are relevant to long-term planning, but they do not need to be proven before the first pilot.

- Corporate social responsibility partnerships and university grants may help support Wanas without charging users.
- Automated matching may eventually support more users, but it must preserve the quality of personal connections.
- Expanding to more neighborhoods in Jordan will require local supervision and trusted partners.
- Group activities may attract more seniors as the community grows.
- Academic partnerships may help study Wanas's longer-term social impact.

### Important Open Questions

The team should investigate these questions during discovery and the pilot:

- How can Wanas introduce the service without making seniors feel weak, pitied, or embarrassed?
- What verification is practical for volunteers, and how can false or misleading information be identified?
- What should happen if a volunteer stops responding or repeatedly misses scheduled calls?
- How should Wanas respond if a senior or volunteer feels uncomfortable with a match?
- What is the correct process if a senior reports a medical emergency during a call or visit?
- How should volunteers respond to requests for money, medication, medical advice, or tasks outside the service rules?
- How can Wanas communicate with seniors who have hearing, vision, or other accessibility needs?
- What should happen if a senior wants to leave the service or withdraw consent?
- How will Wanas handle a shortage of volunteers or a shortage of participating seniors?
- How can the platform protect privacy when users choose to communicate through external channels?
- What process is needed to make equipment sharing safe and practical?
- What legal, safeguarding, and insurance requirements apply to the platform and its activities in Jordan?

These questions should be treated as work to resolve, not as risks that are automatically solved by software or a disclaimer.

## 1.12 Success Criteria

The first success measure is whether Wanas can create and maintain safe, meaningful connections. The number of registered accounts alone is not enough.

### Pilot Success Criteria

The initial pilot can be considered promising if:

1. Around five senior–volunteer pairs are successfully formed.
2. Each active pair has accepted the connection before regular contact begins.
3. Most planned weekly calls take place during the pilot period.
4. Seniors report that they feel comfortable with the interaction and want to continue.
5. Volunteers can maintain the agreed schedule and understand their boundaries.
6. Families understand the service and are comfortable with the consent and safety process.
7. Users can request a new match or report a concern, and the team can demonstrate how it responds.
8. The team can manage registration, matching, and follow-up without an unsustainable workload.

The team should define the exact pilot duration and numerical targets for call attendance, satisfaction, and volunteer retention before the pilot starts. These values should be based on the available time and participant feedback rather than presented as proven results.

### Long-Term Success Indicators

If the pilot is successful, Wanas can later track:

- Number of active senior–volunteer relationships.
- Percentage of scheduled calls and visits completed.
- Volunteer retention over time.
- Senior and family satisfaction.
- stability and reliability of the mobile app
- Number of help requests completed.
- Number of equipment items donated or lent safely.
- Number and type of safety reports, and how they were handled.
- Number of active community and organizational partners.

## 1.13 MVP Scope Summary

The initial MVP should focus on proving one central idea:

Can Wanas create trusted, suitable connections between seniors and volunteers, and help them maintain regular companionship?

The first operational pilot can start with:

- Simple registration and profiles in the React Native app.
- Volunteer review and approval in the Next.js admin dashboard.
- Manual matching based on relevant profile information.
- Acceptance from both sides.
- A way to arrange regular phone calls.
- Basic reminders and follow-up.
- A clear safety and reporting process.
- Basic administration tools.
- A small pilot of approximately five pairs.

The architecture documented here is the full system. It includes those companionship functions, family links and consent, help requests, and equipment listings. Seniors, volunteers, and family members use the React Native app. Administrators use the Next.js dashboard. Arabic and English, including a right-to-left layout, are part of both clients.

This keeps the first pilot small while the architecture covers the full community-support platform.

## 1.14 References

- **Task 01 — Complete System Design & Software Architecture.** Group assignment brief. Defines the required arc42 Section 1 topics: project overview, problem statement, users and stakeholders, functional and non-functional requirements, priorities, assumptions, success criteria, and project boundaries.
- **Wanas — Assumptions & Questions.** Project assumptions document. Used as the basis for the initial pilot assumptions, Jordan context, volunteer age range, weekly calls, manual matching, low-cost approach, safety boundaries, family involvement, equipment sharing, and future sustainability questions.
- **arc42 — Architecture Documentation:** https://docs.arc42.org/

This section describes the current proposed product scope. Requirements and assumptions should be reviewed after interviews, pilot results, and any relevant legal or safeguarding advice.
