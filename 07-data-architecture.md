# 7. Data Architecture

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 7: Data Architecture |
| **Status** | Draft for the final system |
| **Database** | PostgreSQL, mapped with Entity Framework Core |

Wanas stores companionship, schedules, help requests, and equipment listings in one PostgreSQL database on the OVHcloud VPS. Entity Framework Core maps these tables and runs the migrations before a new API container starts. A relational database fits the rules below: one senior has one volunteer, one volunteer can support many seniors, and a call, a help request, or an equipment handover always points at a known person.

The picture and its source are [diagrams/erd.png](diagrams/erd.png) and [diagrams/erd.puml](diagrams/erd.puml).

![Wanas data model](diagrams/erd.png)

## 7.1 Why this database

PostgreSQL keeps accounts, matches, schedules, requests, and listings as separate tables with foreign keys. That matches the requirement to keep those concepts separate, and it can express "one current volunteer per senior" as a unique key. The database runs in a container beside the API, which is the one-machine limit from Section 2. Entity Framework Core stays in the infrastructure layer, so the domain rules do not depend on the table layout.

## 7.2 Main entities

Primary keys are `uuid`. `Account.Phone` is unique. `Match.SeniorProfileId` is unique, so a senior row points at one match and therefore one volunteer. `Match.VolunteerProfileId` is not unique, so that volunteer can appear on many seniors' matches.

| Entity | Primary key | Foreign keys | What it stores |
| --- | --- | --- | --- |
| Account | AccountId | LockedByAdminId → Admin | Every login: senior, volunteer, family member, or administrator. `IsLocked` stops the account. |
| Admin | AdminId | AccountId → Account, unique | Staff data for an administrator: full name, title, and `IsLocked` on the staff record. |
| SeniorProfile | SeniorProfileId | AccountId → Account, unique | Area, private home pin, interests, language, and communication preference. |
| VolunteerProfile | VolunteerProfileId | AccountId → Account, unique | Area, private home pin, interests, languages, availability, and verification status. |
| FamilyLink | FamilyLinkId | FamilyAccountId → Account, SeniorProfileId → SeniorProfile | Consent and the permissions granted to one family member for one senior. |
| Match | MatchId | SeniorProfileId → SeniorProfile, unique; VolunteerProfileId → VolunteerProfile; ConfirmedByAdminId → Admin | The one volunteer linked to a senior, plus both acceptance times. |
| Interaction | InteractionId | MatchId → Match | A scheduled call: start time, status, and reminder status. |
| HelpRequest | HelpRequestId | SeniorProfileId → SeniorProfile; CreatedByAccountId → Account; AcceptedByVolunteerId → VolunteerProfile | An everyday help request and the volunteer who accepted it. |
| EquipmentListing | ListingId | OwnerAccountId → Account; ReviewedByAdminId → Admin | A donate, lend, or request listing. |
| EquipmentPhoto | PhotoId | ListingId → EquipmentListing | A stored photo for a listing. |
| EquipmentHandover | HandoverId | ListingId → EquipmentListing; RequesterAccountId → Account | The collection arrangement for an item. |
| SafetyReport | ReportId | ReporterAccountId → Account; MatchId → Match; ReviewedByAdminId → Admin | A concern and the outcome an administrator recorded. |
| AdminAction | ActionId | AdminId → Admin; TargetAccountId → Account; MatchId → Match | A lock, an unlock, a rematch, or another important staff action. |

Home latitude and longitude stay on the profile. A response about another person returns `areaName` and a display name. The exact home pin and the phone number stay on the stored row.

### Columns

These are the columns for each entity. Primary keys are `uuid`.

#### Account

| Column | Rule |
| --- | --- |
| AccountId | Primary key. |
| Role | Senior, Volunteer, Family, or Admin. |
| Phone | Unique. |
| DisplayName | The name returned with another person's `areaName`. |
| PreferredLanguage | `ar` or `en`. |
| IsLocked | When true, sign-in stops. |
| LockedAt | Required when `IsLocked` is true. Empty when `IsLocked` is false. |
| LockedByAdminId | Foreign key to Admin. Required when locked. Empty when unlocked. |
| Status | Stored on the account row. |

#### Admin

| Column | Rule |
| --- | --- |
| AdminId | Primary key. |
| AccountId | Foreign key to Account. Unique. The account role is Admin. |
| FullName | The administrator's full name. |
| StaffTitle | The staff title. |
| IsLocked | Lock on the staff record. Set to the same value as the account lock. |

#### SeniorProfile

| Column | Rule |
| --- | --- |
| SeniorProfileId | Primary key. |
| AccountId | Foreign key to Account. Unique. |
| AreaName | Returned to other people as `areaName`. |
| HomeLatitude | Private home pin. Returned on `GET /me` for this senior. |
| HomeLongitude | Private home pin. Returned on `GET /me` for this senior. |
| Interests | Used when choosing a suggestion. |
| PreferredLanguage | `ar` or `en`. |
| CommunicationPreference | `phone` or `visit`. |

#### VolunteerProfile

| Column | Rule |
| --- | --- |
| VolunteerProfileId | Primary key. |
| AccountId | Foreign key to Account. Unique. |
| AreaName | Returned to other people as `areaName`. |
| HomeLatitude | Private home pin. Returned on `GET /me` for this volunteer. |
| HomeLongitude | Private home pin. Returned on `GET /me` for this volunteer. |
| Interests | Used when choosing a suggestion. |
| Languages | `ar`, `en`, or both. |
| Availability | `weekday_mornings`, `weekday_evenings`, or `weekends`. |
| VerificationStatus | Pending, Approved, or Rejected. |

#### FamilyLink

| Column | Rule |
| --- | --- |
| FamilyLinkId | Primary key. |
| FamilyAccountId | Foreign key to Account. |
| SeniorProfileId | Foreign key to SeniorProfile. |
| ConsentStatus | Pending, Approved, or Revoked. |
| CanCreateHelpRequest | True when this family member may create a help request. |
| CanViewSchedule | True when this family member may view and schedule calls. |

#### Match

| Column | Rule |
| --- | --- |
| MatchId | Primary key. |
| SeniorProfileId | Foreign key to SeniorProfile. Unique, so the senior has one current match. |
| VolunteerProfileId | Foreign key to VolunteerProfile. The same volunteer can appear on many seniors' matches. |
| Status | Proposed, Offered, Active, Declined, or Ended. |
| SeniorAcceptedAt | Set when the senior accepts. |
| VolunteerAcceptedAt | Set when the volunteer accepts. |
| ConfirmedByAdminId | Foreign key to Admin. Required when the status is Offered or Active. |

#### Interaction

| Column | Rule |
| --- | --- |
| InteractionId | Primary key. |
| MatchId | Foreign key to Match. |
| StartsAt | The agreed call time. |
| Status | Scheduled, Completed, Cancelled, or Missed. |
| ReminderStatus | `pending`, `sent`, or `failed`. |

#### HelpRequest

| Column | Rule |
| --- | --- |
| HelpRequestId | Primary key. |
| SeniorProfileId | Foreign key to SeniorProfile. |
| CreatedByAccountId | Foreign key to Account. The senior, or the family member who created the request. |
| Category | `grocery`, `transportation`, `technology`, or `errand`. |
| Description | What the requester needs. |
| Audience | `matched_volunteer` or `approved_volunteers`. |
| Status | Open, Accepted, Completed, or Cancelled. |
| AcceptedByVolunteerId | Foreign key to VolunteerProfile. Required when the status is Accepted or Completed. |

#### EquipmentListing

| Column | Rule |
| --- | --- |
| ListingId | Primary key. |
| OwnerAccountId | Foreign key to Account. |
| ListingType | `donate`, `lend`, or `request`. |
| Title | The item name. |
| Description | The item description. |
| Condition | `new`, `good`, or `fair`. |
| AreaName | The area shown for the item. |
| Status | Available, Reserved, Collected, Transferred, or Unavailable. |
| ReviewedByAdminId | Foreign key to Admin. Set when an administrator hides the listing. |

#### EquipmentPhoto

| Column | Rule |
| --- | --- |
| PhotoId | Primary key. |
| ListingId | Foreign key to EquipmentListing. |
| StorageKey | The file key for the photo stored on the VPS. |

#### EquipmentHandover

| Column | Rule |
| --- | --- |
| HandoverId | Primary key. |
| ListingId | Foreign key to EquipmentListing. |
| RequesterAccountId | Foreign key to Account. The person arranging collection. |
| CollectionNote | The agreed collection note. |
| Status | `arranged` when the handover is recorded. |

#### SafetyReport

| Column | Rule |
| --- | --- |
| ReportId | Primary key. |
| ReporterAccountId | Foreign key to Account. |
| MatchId | Foreign key to Match. |
| Description | The concern. |
| Status | Open until an outcome is stored, then closed. |
| Outcome | `no_action`, `warning_sent`, `match_ended`, or `account_locked`. |
| ReviewedByAdminId | Foreign key to Admin. |

#### AdminAction

| Column | Rule |
| --- | --- |
| ActionId | Primary key. |
| AdminId | Foreign key to Admin. |
| ActionType | The staff action: a lock, an unlock, a rematch, a hidden listing, or a report outcome. |
| TargetAccountId | Foreign key to Account. |
| MatchId | Foreign key to Match. |
| Notes | The note stored with the action. |
| CreatedAt | When the action was stored. |

## 7.3 Relationships

| From | To | Cardinality | Rule |
| --- | --- | --- | --- |
| Account | SeniorProfile | One to zero or one | Only a senior account has a senior profile. |
| Account | VolunteerProfile | One to zero or one | Only a volunteer account has a volunteer profile. |
| Account | Admin | One to zero or one | Only an administrator account has admin data. |
| SeniorProfile | FamilyLink | One to many | A senior may grant access to more than one family member. |
| SeniorProfile | Match | One to zero or one | A senior has one volunteer. |
| VolunteerProfile | Match | One to many | A volunteer can be the match for many seniors. |
| Match | Interaction | One to many | One match has many scheduled calls. |
| SeniorProfile | HelpRequest | One to many | A senior can have many help requests. |
| VolunteerProfile | HelpRequest | One to many | A volunteer can accept many requests, including requests from several seniors. |
| Account | EquipmentListing | One to many | A person can publish many listings. |
| EquipmentListing | EquipmentPhoto | One to many | A listing can have several photos. |
| EquipmentListing | EquipmentHandover | One to many | Collection is recorded separately from the listing text. |
| Account | SafetyReport | One to many | A person can file more than one report. |
| Admin | Account | One to many | An administrator locks and unlocks accounts. |
| Admin | AdminAction | One to many | Staff actions stay with the administrator who took them. |

## 7.4 Constraints and validation

| Rule | How it is enforced |
| --- | --- |
| Role is Senior, Volunteer, Family, or Admin. | Check on `Account.Role`. |
| A locked account records when and which administrator locked it. | `IsLocked = true` requires `LockedAt` and `LockedByAdminId`. |
| An unlocked account has no lock stamp. | `IsLocked = false` requires `LockedAt` and `LockedByAdminId` to be empty. |
| Admin staff data belongs to an administrator account. | `Admin.AccountId` is unique, and that account's role is Admin. |
| A senior has one match row. | Unique `Match.SeniorProfileId`. |
| An active match has both acceptances. | `Status = Active` requires `SeniorAcceptedAt` and `VolunteerAcceptedAt`. |
| A confirmed match names the administrator. | `Status` of Offered or Active requires `ConfirmedByAdminId`. |
| Call status is Scheduled, Completed, Cancelled, or Missed. | Check on `Interaction.Status`. |
| A call is scheduled only for an active match. | The API reads `Match.Status` before inserting an interaction. |
| Help category is grocery shopping, transportation or accompaniment, technology help, or a small errand. | Check on `HelpRequest.Category`. Medical care, medication, money, and emergencies are not values in that list. |
| Help status is Open, Accepted, Completed, or Cancelled. | Check on `HelpRequest.Status`. |
| An accepted help request names a volunteer. | `Status = Accepted` or `Completed` requires `AcceptedByVolunteerId`. |
| A family member creates a help request only with consent and that permission. | `FamilyLink.ConsentStatus = Approved` and `CanCreateHelpRequest = true`. |
| Listing type is Donate, Lend, or Request. | Check on `EquipmentListing.ListingType`. |
| Listing status is Available, Reserved, Collected, Transferred, or Unavailable. | Check on `EquipmentListing.Status`. |
| The exact home pin and phone number are not public fields. | They live on `Account` and the profile. Profile responses used by other people return `AreaName` only. |

`Match.Status` moves from Proposed, to Offered, to Active, or to Declined or Ended. Volunteer verification is Pending, Approved, or Rejected. A volunteer enters matching only after Approved.

## 7.5 Ownership and lifecycle

| Data | Owner | Lifecycle |
| --- | --- | --- |
| Account and `IsLocked` | The person owns the profile facts. An administrator owns the lock. | Created at registration. A lock keeps the rows and blocks sign-in. Unlock clears `LockedAt` and `LockedByAdminId`. |
| Admin | The staff record. | Created when an administrator account is added. `Admin.IsLocked` blocks that staff user without deleting the record. |
| Senior and volunteer profiles | The account holder. | Updated by that person. The home pin is replaced only by that person. |
| FamilyLink | The senior, through consent. | Stays until the senior withdraws it. Withdrawal sets consent to Revoked and clears the permission flags. |
| Match | The senior, the volunteer, and the confirming administrator. | One row per senior. Ending it sets Status to Ended and writes an `AdminAction`. A later volunteer replaces `VolunteerProfileId` only after both new acceptances. |
| Interaction | The matched pair. | Scheduled, then Completed, Cancelled, or Missed. The row remains. |
| HelpRequest | The senior. A family member with permission may create it. | Open, Accepted, Completed, or Cancelled. |
| Equipment listing, photos, and handover | The listing owner and the requester. An administrator may hide it. | Status follows the item from Available through collection. Hiding sets Unavailable and writes an `AdminAction`. |
| SafetyReport | The reporter. The reviewing administrator owns the outcome. | Open until an outcome is stored. |
| AdminAction | The administrator who acted. | Inserted once and kept. |

A registration code is temporary. It is stored only until it is used or expires, then deleted. It is not a lasting entity on the diagram.

## 7.6 Integrity and consistency

Foreign keys stop a match, a call, a help request, or a listing from pointing at a missing person. Deleting an account is not a normal action. Locking it leaves the historical rows in place.

The unique senior key and the acceptance check keep the companionship rule in the database, not only in a screen. One volunteer still has many match rows because the unique key is on the senior, not on the volunteer.

Help requests and equipment are in this model so those records have owners and statuses. A help request cannot name a category outside the four everyday tasks. An equipment item cannot skip from Available to Transferred without a handover row.

`AdminAction` is the consistency record for locks, ended matches, hidden listings, and report outcomes. The report row holds the outcome. The action row holds who changed it and when.

## 7.7 How this supports the system

The model gives a senior one volunteer and lets that volunteer be matched with other seniors. Calls hang off that match, so a schedule exists only for a real pair. Help requests hang off the senior and can name the matched volunteer or stay open for an approved volunteer. Equipment listings hang off the owner, with photos and a recorded collection. `IsLocked` on `Account`, `IsLocked` on `Admin`, and `LockedByAdminId` give administrators a lock without removing the person's data. `AdminAction` keeps the staff record the assignment asks for.

Private coordinates and phone numbers have columns, and the public read path does not return them. That is the same privacy rule as Section 2 and Section 4.

## 7.8 References

- Section 1, `01-introduction-goals-requirements.md`, especially profiles, matching, scheduling, Helping Hands, Community Exchange, and administration.
- Section 2, `02-architecture-constraints.md`, for PostgreSQL, separate concepts, and private contact details.
- Section 4, `04-solution-strategy.md`, for Entity Framework Core and match acceptance.
- Section 6, `06-runtime-view.md`, for the registration, match, reminder, and help-request writes.
- Diagram source: [diagrams/erd.puml](diagrams/erd.puml).
