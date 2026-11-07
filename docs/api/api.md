# 8. API Design and Communication

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 8: API Design and Communication |
| **Status** | Draft for the final system |
| **Style** | JSON over HTTPS |

The mobile app and the admin dashboard call one API at `https://wanas-jo.com/api`. Every path below is relative to that base. A signed-in request sends `Authorization: Bearer <token>`.

The token carries one role: `Senior`, `Volunteer`, `Family`, or `Admin`. A call listed under another role returns `403` with `"error": "forbidden"`. A locked account returns `403` with `"error": "account_locked"`. Both leave stored rows unchanged.

Every error body has the same shape:

```json
{ "error": "invalid_code" }
```

These results apply to every signed-in call, so the tables below list only the extra checks for that call.

| Situation | Result |
| --- | --- |
| The token is missing or not valid. | `401` `unauthorized` |
| The account is locked. | `403` `account_locked` |
| The role, or the link permission, does not allow this call. | `403` `forbidden` |

A response about the caller may include that caller's own phone number and home pin. A response about another person includes `areaName` and a display name only.

## 8.1 Shared sign-in

Seniors, volunteers, and family members create their own account with the same two calls. The `role` in the body decides which account is created. An administrator does not use these calls. Staff sign-in is in [8.5](#85-administrator).

### 8.1.1 Request a registration code

| | |
| --- | --- |
| **Method and path** | `POST /registration/codes` |
| **Authentication** | None. |
| **Who** | A person registering as a senior, a volunteer, or a family member. |

Request body:

```json
{
  "phone": "+962790000000",
  "role": "Senior",
  "displayName": "Layla Hassan",
  "preferredLanguage": "ar"
}
```

`role` is `Senior`, `Volunteer`, or `Family`. `preferredLanguage` is `ar` or `en`.

Success, `202 Accepted`:

```json
{ "status": "code_sent" }
```

The API stores a pending code and asks WhatsApp to deliver it. The response does not contain the code.

| Check | Error |
| --- | --- |
| `phone` is missing or not a phone number. | `400` `invalid_phone` |
| `role` is missing or not one of the three mobile roles. | `400` `invalid_role` |
| `displayName` is missing, or `preferredLanguage` is not `ar` or `en`. | `400` `invalid_account` |
| The phone already has an account. | `409` `phone_taken` |
| WhatsApp does not accept the message. | `502` `whatsapp_unavailable`. The pending code is removed. |

### 8.1.2 Verify the code

| | |
| --- | --- |
| **Method and path** | `POST /registration/verify` |
| **Authentication** | None. The code is the proof. |

Request body:

```json
{
  "phone": "+962790000000",
  "code": "482193"
}
```

Success, `200 OK`:

```json
{
  "accountId": "3a1b2c4d-5e6f-4a7b-8c9d-0e1f2a3b4c5d",
  "role": "Senior",
  "token": "session-token"
}
```

The API stores the account with that role, removes the used code, and returns the session token. A senior or volunteer still needs a profile before matching. A family member still needs an approved link before acting for a senior.

| Check | Error |
| --- | --- |
| The phone or the code is missing. | `400` `invalid_code` |
| The code is wrong or expired. | `400` `invalid_code`. No account is stored. |
| The phone already completed registration. | `409` `phone_taken` |

## 8.2 Senior

A senior token updates one senior profile, decides family consent, holds one match, schedules calls on that match, asks for everyday help, lists equipment, and files a safety report.

| Method | Path | What it does |
| --- | --- | --- |
| GET | `/me` | Returns the senior account, profile, and home pin. |
| PUT | `/profiles/senior` | Creates or replaces the senior profile. |
| GET | `/family-links` | Lists family members waiting for consent or already linked. |
| POST | `/family-links/{familyLinkId}/consent` | Approves or withdraws a family link and sets the two permissions. |
| POST | `/matches/suggestions` | Asks the system for the senior's one volunteer suggestion. |
| GET | `/matches/mine` | Returns that one match, without the volunteer's phone or home pin. |
| POST | `/matches/{matchId}/acceptances` | Accepts or declines the offered match. |
| GET | `/interactions` | Lists calls on the senior's match. |
| POST | `/interactions` | Schedules a call on an active match. |
| PATCH | `/interactions/{interactionId}` | Cancels or reschedules a scheduled call. |
| POST | `/help-requests` | Creates an everyday help request. |
| GET | `/help-requests` | Lists the senior's help requests and their status. |
| POST | `/help-requests/{helpRequestId}/cancellation` | Cancels an open or accepted request. |
| POST | `/equipment-listings` | Publishes a donate, lend, or request listing. |
| POST | `/equipment-listings/{listingId}/photos` | Adds one photo to the caller's own listing. |
| GET | `/equipment-listings` | Lists available equipment. Another owner's phone is omitted. |
| POST | `/reports` | Files a safety report about the senior's match. |

### 8.2.1 Update the senior profile

| | |
| --- | --- |
| **Method and path** | `PUT /profiles/senior` |
| **Authentication** | Bearer token with role `Senior`. |

Request body:

```json
{
  "areaName": "Jabal Amman",
  "homeLatitude": 31.9539,
  "homeLongitude": 35.9106,
  "interests": ["reading", "walking"],
  "preferredLanguage": "ar",
  "communicationPreference": "phone"
}
```

`communicationPreference` is `phone` or `visit`. The home pin is stored on `SeniorProfile` and is returned only by `GET /me` for this senior.

Success, `200 OK`:

```json
{
  "seniorProfileId": "aa11bb22-cc33-4d44-8e55-ff6677889900",
  "areaName": "Jabal Amman",
  "preferredLanguage": "ar",
  "communicationPreference": "phone"
}
```

| Check | Error |
| --- | --- |
| A required field is missing, the language is not `ar` or `en`, or the pin is outside a usable latitude and longitude. | `400` `invalid_profile` |
| `communicationPreference` is not `phone` or `visit`. | `400` `invalid_profile` |

### 8.2.2 Decide a family link

| | |
| --- | --- |
| **Method and path** | `POST /family-links/{familyLinkId}/consent` |
| **Authentication** | Bearer token with role `Senior`. The link must point at this senior's profile. |

Request body for approval:

```json
{
  "decision": "approve",
  "canCreateHelpRequest": true,
  "canViewSchedule": true
}
```

`decision` is `approve` or `revoke`. Approval sets `ConsentStatus` to `Approved` and stores the two flags. Revoke sets `ConsentStatus` to `Revoked` and clears both flags. The family member keeps their own account.

Success, `200 OK`:

```json
{
  "familyLinkId": "c1d2e3f4-a5b6-4c7d-8e9f-001122334455",
  "consentStatus": "Approved",
  "canCreateHelpRequest": true,
  "canViewSchedule": true
}
```

The response names the family member by display name and omits their phone number.

| Check | Error |
| --- | --- |
| `decision` is missing, or a permission flag is missing on approval. | `400` `invalid_consent` |
| No link has this id, or the link belongs to another senior. | `404` `link_not_found` |
| The link is already revoked and the decision is `approve` after revoke. A new request from the family member is required. | `409` `link_revoked` |

### 8.2.3 Ask for a match suggestion

| | |
| --- | --- |
| **Method and path** | `POST /matches/suggestions` |
| **Authentication** | Bearer token with role `Senior`. |
| **Request body** | Empty. The API reads this senior's profile. |

The API chooses one approved volunteer using shared interests, language, and area. It stores one `Match` row with status `proposed`. A senior has one match row, so a later suggestion after `ended` or `declined` replaces `VolunteerProfileId` on that same row and clears both acceptance times.

Success, `201 Created`:

```json
{
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "status": "proposed",
  "volunteer": {
    "displayName": "Omar Nasser",
    "areaName": "Jabal Amman"
  }
}
```

| Check | Error |
| --- | --- |
| The senior has no profile yet. | `409` `profile_required` |
| The current match is `proposed`, `offered`, or `active`. | `409` `match_exists` |
| No approved volunteer fits interests, language, and area. | `404` `no_volunteer` |

### 8.2.4 Accept or decline the match

| | |
| --- | --- |
| **Method and path** | `POST /matches/{matchId}/acceptances` |
| **Authentication** | Bearer token. The caller must be the senior or the volunteer on this match. |

Request body:

```json
{ "decision": "accept" }
```

`decision` is `accept` or `decline`.

Success when this senior accepts and the volunteer has not decided yet, `200 OK`:

```json
{
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "status": "inactive"
}
```

Success when the second person also accepts:

```json
{
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "status": "active"
}
```

Success when either person declines:

```json
{
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "status": "declined"
}
```

`inactive` means one acceptance is stored and the match is waiting for the other person. `active` requires both `SeniorAcceptedAt` and `VolunteerAcceptedAt`. The response has no phone number and no home pin.

| Check | Error |
| --- | --- |
| The caller is a family member, an administrator, or anyone other than the two people on this match. | `403` `forbidden`. The match row stays as it is. |
| The volunteer account is still `Pending` or `Rejected`. | `403` `volunteer_not_approved` |
| `decision` is missing or not `accept` or `decline`. | `400` `invalid_decision` |
| No match has this id. | `404` `match_not_found` |
| The match is not `offered`, and it is not waiting for the second acceptance. | `409` `match_not_ready` |

### 8.2.5 Schedule a call

| | |
| --- | --- |
| **Method and path** | `POST /interactions` |
| **Authentication** | Bearer token. The caller must be the senior or the volunteer on an active match. |

Request body:

```json
{
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "startsAt": "2027-12-06T10:00:00Z"
}
```

Success, `201 Created`:

```json
{
  "interactionId": "9b8a7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d",
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "startsAt": "2027-12-06T10:00:00Z",
  "status": "scheduled",
  "reminderStatus": "pending"
}
```

The call row is stored before the reminder is sent. The phone call itself stays on the normal phone. If FCM or APNs later rejects the reminder, `reminderStatus` becomes `failed` and `status` stays `scheduled`.

| Check | Error |
| --- | --- |
| The caller is not one of the two people on this match. | `403` `forbidden` |
| `matchId` is missing, or `startsAt` is missing or not a future time. | `400` `invalid_interaction` |
| No match has this id. | `404` `match_not_found` |
| The match is not `active`. | `409` `match_not_active`. Nothing is saved. |

### 8.2.6 Cancel or reschedule a call

| | |
| --- | --- |
| **Method and path** | `PATCH /interactions/{interactionId}` |
| **Authentication** | Bearer token. The caller must be the senior or the volunteer on that call's match. |

Cancel:

```json
{ "action": "cancel" }
```

Reschedule:

```json
{
  "action": "reschedule",
  "startsAt": "2027-12-07T10:00:00Z"
}
```

Success after cancel, `200 OK`:

```json
{
  "interactionId": "9b8a7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d",
  "status": "cancelled"
}
```

Success after reschedule sets `status` back to `scheduled` and `reminderStatus` to `pending`. The row is kept.

| Check | Error |
| --- | --- |
| `action` is missing, or reschedule has no future `startsAt`. | `400` `invalid_interaction` |
| No interaction has this id. | `404` `interaction_not_found` |
| The call is `completed` or `missed`. | `409` `interaction_closed` |

### 8.2.7 Create a help request

| | |
| --- | --- |
| **Method and path** | `POST /help-requests` |
| **Authentication** | Bearer token with role `Senior`. |

Request body:

```json
{
  "category": "grocery",
  "description": "Weekly shopping for bread and vegetables",
  "audience": "matched_volunteer"
}
```

`category` is `grocery`, `transportation`, `technology`, or `errand`. `audience` is `matched_volunteer` or `approved_volunteers`. Medical care, medication, money handling, and emergencies are not categories.

Success, `201 Created`:

```json
{
  "helpRequestId": "d4e5f6a7-b8c9-4d0e-1f2a-3b4c5d6e7f80",
  "category": "grocery",
  "audience": "matched_volunteer",
  "status": "open"
}
```

| Check | Error |
| --- | --- |
| The category, audience, or description is missing or not an allowed value. | `400` `invalid_help_request` |
| `audience` is `matched_volunteer` and the senior's match is not `active`. | `409` `match_not_active`. Nothing is saved. |

### 8.2.8 Publish an equipment listing

| | |
| --- | --- |
| **Method and path** | `POST /equipment-listings` |
| **Authentication** | Bearer token with role `Senior`, `Volunteer`, or `Family`. |

Request body:

```json
{
  "listingType": "donate",
  "title": "Walking frame",
  "description": "Folding frame, used indoors",
  "condition": "good",
  "areaName": "Jabal Amman"
}
```

`listingType` is `donate`, `lend`, or `request`. `condition` is `new`, `good`, or `fair`.

Success, `201 Created`:

```json
{
  "listingId": "e7f8a9b0-c1d2-4e3f-8a4b-5c6d7e8f9012",
  "listingType": "donate",
  "status": "available"
}
```

| Check | Error |
| --- | --- |
| A required field is missing, or type or condition is not an allowed value. | `400` `invalid_listing` |

A photo is a second call, `POST /equipment-listings/{listingId}/photos`, with one image file from the listing owner. Success is `201` and `{ "photoId": "..." }`. A caller who does not own the listing receives `403` `forbidden`.

### 8.2.9 File a safety report

| | |
| --- | --- |
| **Method and path** | `POST /reports` |
| **Authentication** | Bearer token with role `Senior` or `Volunteer` on this match, or `Family` with an approved link to that senior. |

Request body:

```json
{
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "description": "The volunteer missed repeated calls and the visit felt unsafe."
}
```

Success, `201 Created`:

```json
{
  "reportId": "f0e1d2c3-b4a5-4968-8776-655443322110",
  "status": "open"
}
```

| Check | Error |
| --- | --- |
| `matchId` or `description` is missing. | `400` `invalid_report` |
| No match has this id, or this caller is not allowed to report it. | `404` `match_not_found` |

## 8.3 Family member

A family member has a separate account. They act for a senior only through a `FamilyLink` whose `ConsentStatus` is `Approved`, and only with the flag that call needs.

| Method | Path | What it does |
| --- | --- | --- |
| GET | `/me` | Returns the family account and each link, with consent and the two flags. Senior phone numbers and home pins are omitted. |
| POST | `/family-links` | Asks a senior for a link. Consent stays `Pending` until the senior approves it. |
| GET | `/family/seniors/{seniorProfileId}` | Returns the linked senior's display name, area, and match status when consent is approved. |
| GET | `/family/seniors/{seniorProfileId}/interactions` | Lists that senior's calls when `CanViewSchedule` is true. |
| POST | `/family/seniors/{seniorProfileId}/interactions` | Schedules a call for that senior when `CanViewSchedule` is true and the match is active. |
| POST | `/family/seniors/{seniorProfileId}/help-requests` | Creates a help request when `CanCreateHelpRequest` is true. |
| GET | `/family/seniors/{seniorProfileId}/help-requests` | Lists that senior's help requests when `CanCreateHelpRequest` is true. |
| POST | `/reports` | Files a safety report on the linked senior's match. Approved consent is enough. The schedule flag is not required. |
| POST | `/equipment-listings` | Publishes the family member's own listing. Same body as [8.2.8](#828-publish-an-equipment-listing). |

Match acceptance, volunteer review, account lock, and the senior's home pin are administrator or senior calls. A family token on those paths receives `403` `forbidden`.

### 8.3.1 Request a link to a senior

| | |
| --- | --- |
| **Method and path** | `POST /family-links` |
| **Authentication** | Bearer token with role `Family`. |

Request body:

```json
{ "seniorPhone": "+962791111111" }
```

The senior phone is used only to find the account. It is not copied onto the family profile and it is not returned.

Success, `201 Created`:

```json
{
  "familyLinkId": "c1d2e3f4-a5b6-4c7d-8e9f-001122334455",
  "seniorDisplayName": "Layla Hassan",
  "areaName": "Jabal Amman",
  "consentStatus": "Pending",
  "canCreateHelpRequest": false,
  "canViewSchedule": false
}
```

| Check | Error |
| --- | --- |
| `seniorPhone` is missing or not a phone number. | `400` `invalid_phone` |
| No senior account has that phone. | `404` `senior_not_found` |
| A pending or approved link already joins this family member and this senior. | `409` `link_exists` |

### 8.3.2 Read the senior's calls

| | |
| --- | --- |
| **Method and path** | `GET /family/seniors/{seniorProfileId}/interactions` |
| **Authentication** | Bearer token with role `Family`. The link to this senior must be `Approved` and `CanViewSchedule` must be true. |

Success, `200 OK`:

```json
{
  "interactions": [
    {
      "interactionId": "9b8a7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d",
      "startsAt": "2027-12-06T10:00:00Z",
      "status": "scheduled",
      "reminderStatus": "sent"
    }
  ]
}
```

| Check | Error |
| --- | --- |
| Consent is missing, pending, or revoked, or `CanViewSchedule` is false. | `403` `forbidden` |
| This family member has no link to that senior profile. | `404` `link_not_found` |

Scheduling uses the same path with `POST` and the same body as [8.2.5](#825-schedule-a-call), except the match id comes from the senior's one match. The match must be `active`. A missing schedule permission returns `403` `forbidden` and saves nothing.

### 8.3.3 Create a help request for the senior

| | |
| --- | --- |
| **Method and path** | `POST /family/seniors/{seniorProfileId}/help-requests` |
| **Authentication** | Bearer token with role `Family`. The link must be `Approved` and `CanCreateHelpRequest` must be true. |

Request body:

```json
{
  "category": "technology",
  "description": "Help setting up a new phone",
  "audience": "approved_volunteers"
}
```

The category and audience values are the same as [8.2.7](#827-create-a-help-request). `CreatedByAccountId` is the family member. `SeniorProfileId` is the linked senior.

Success, `201 Created`:

```json
{
  "helpRequestId": "d4e5f6a7-b8c9-4d0e-1f2a-3b4c5d6e7f80",
  "category": "technology",
  "status": "open",
  "createdByRole": "Family"
}
```

| Check | Error |
| --- | --- |
| Consent is not approved, or `CanCreateHelpRequest` is false. | `403` `forbidden`. Nothing is saved. |
| The category, audience, or description is not allowed. | `400` `invalid_help_request` |
| `audience` is `matched_volunteer` and the senior's match is not `active`. | `409` `match_not_active` |

## 8.4 Volunteer

A volunteer token manages one volunteer profile. Matching starts only after an administrator sets verification to `Approved`. One volunteer can be the match on many seniors' rows. Each senior still has one match row.

| Method | Path | What it does |
| --- | --- | --- |
| GET | `/me` | Returns the volunteer account, profile, home pin, and `verificationStatus`. |
| PUT | `/profiles/volunteer` | Creates or replaces the volunteer profile. New and edited profiles are stored as `Pending` until an administrator reviews them. |
| GET | `/matches` | Lists every match that names this volunteer. Each item has the senior's display name and area, the status, and no phone or home pin. |
| POST | `/matches/{matchId}/acceptances` | Accepts or declines one offered match. Same body and results as [8.2.4](#824-accept-or-decline-the-match). |
| GET | `/interactions?matchId={matchId}` | Lists calls for one match that names this volunteer. |
| POST | `/interactions` | Schedules a call. Same body and results as [8.2.5](#825-schedule-a-call). |
| PATCH | `/interactions/{interactionId}` | Cancels or reschedules. Same body as [8.2.6](#826-cancel-or-reschedule-a-call). |
| GET | `/help-requests` | Lists open requests this approved volunteer may take: their matched seniors, plus requests aimed at approved volunteers in the same area. |
| POST | `/help-requests/{helpRequestId}/acceptance` | Accepts one open request. |
| POST | `/help-requests/{helpRequestId}/completion` | Marks a request this volunteer accepted as completed. |
| POST | `/equipment-listings` | Publishes the volunteer's own listing. Same body as [8.2.8](#828-publish-an-equipment-listing). |
| POST | `/equipment-listings/{listingId}/handovers` | Asks to collect an available listing owned by someone else. |
| POST | `/reports` | Files a safety report on a match that names this volunteer. Same body as [8.2.9](#829-file-a-safety-report). |

### 8.4.1 Update the volunteer profile

| | |
| --- | --- |
| **Method and path** | `PUT /profiles/volunteer` |
| **Authentication** | Bearer token with role `Volunteer`. |

Request body:

```json
{
  "areaName": "Jabal Amman",
  "homeLatitude": 31.9566,
  "homeLongitude": 35.9457,
  "interests": ["reading", "technology"],
  "languages": ["ar", "en"],
  "availability": "weekday_evenings"
}
```

`availability` is `weekday_mornings`, `weekday_evenings`, or `weekends`.

Success, `200 OK`:

```json
{
  "volunteerProfileId": "bb22cc33-dd44-4e55-8f66-778899001122",
  "areaName": "Jabal Amman",
  "verificationStatus": "Pending"
}
```

Saving the profile sets verification to `Pending`. An approved volunteer who changes the profile waits for review again before new suggestions can name them. Matches already `active` stay active.

| Check | Error |
| --- | --- |
| A required field is missing, a language is not `ar` or `en`, or availability is not one of the three values. | `400` `invalid_profile` |

### 8.4.2 Accept a help request

| | |
| --- | --- |
| **Method and path** | `POST /help-requests/{helpRequestId}/acceptance` |
| **Authentication** | Bearer token with role `Volunteer` and verification `Approved`. |
| **Request body** | Empty. |

If `audience` is `matched_volunteer`, the caller must be the volunteer on that senior's active match. If `audience` is `approved_volunteers`, any approved volunteer may accept it.

Success, `200 OK`:

```json
{
  "helpRequestId": "d4e5f6a7-b8c9-4d0e-1f2a-3b4c5d6e7f80",
  "status": "accepted"
}
```

The row stores `AcceptedByVolunteerId`.

| Check | Error |
| --- | --- |
| Verification is `Pending` or `Rejected`. | `403` `volunteer_not_approved` |
| The caller is not the matched volunteer for a `matched_volunteer` request. | `403` `forbidden` |
| No request has this id. | `404` `help_request_not_found` |
| The request is not `open`. | `409` `help_request_closed` |

### 8.4.3 Complete a help request

| | |
| --- | --- |
| **Method and path** | `POST /help-requests/{helpRequestId}/completion` |
| **Authentication** | Bearer token. The caller must be the volunteer in `AcceptedByVolunteerId`. |
| **Request body** | Empty. |

Success, `200 OK`:

```json
{
  "helpRequestId": "d4e5f6a7-b8c9-4d0e-1f2a-3b4c5d6e7f80",
  "status": "completed"
}
```

| Check | Error |
| --- | --- |
| Another volunteer, the senior, or a family member calls this path. | `403` `forbidden`. The status stays `accepted`. |
| The request is not `accepted`. | `409` `help_request_closed` |

### 8.4.4 Arrange equipment collection

| | |
| --- | --- |
| **Method and path** | `POST /equipment-listings/{listingId}/handovers` |
| **Authentication** | Bearer token with role `Senior`, `Volunteer`, or `Family`. The caller must not be the listing owner. |

Request body:

```json
{ "collectionNote": "Friday morning at the community center" }
```

Success, `201 Created`:

```json
{
  "handoverId": "a9b8c7d6-e5f4-4a3b-8c2d-1e0f9a8b7c6d",
  "listingId": "e7f8a9b0-c1d2-4e3f-8a4b-5c6d7e8f9012",
  "status": "arranged"
}
```

The listing status becomes `reserved`.

| Check | Error |
| --- | --- |
| The caller owns the listing. | `403` `forbidden` |
| `collectionNote` is missing. | `400` `invalid_handover` |
| No listing has this id, or an administrator has set it `unavailable`. | `404` `listing_not_found` |
| The listing is not `available`. | `409` `listing_unavailable` |

## 8.5 Administrator

An administrator uses the dashboard. Staff accounts already exist. Public registration cannot choose the role `Admin`.

| Method | Path | What it does |
| --- | --- | --- |
| POST | `/admin/session/codes` | Sends a WhatsApp code when the phone already belongs to an administrator. |
| POST | `/admin/session/verify` | Returns an administrator session token. |
| GET | `/me` | Returns `adminId`, full name, staff title, and both lock flags. |
| GET | `/admin/volunteers?status=Pending` | Lists volunteer profiles waiting for review. Home pins and phones are omitted. |
| POST | `/admin/volunteers/{volunteerProfileId}/review` | Sets verification to `Approved` or `Rejected` and writes an `AdminAction`. |
| GET | `/admin/matches?status=proposed` | Lists suggestions waiting for confirmation. |
| POST | `/admin/matches/{matchId}/confirmation` | Moves a `proposed` match to `offered`. |
| POST | `/admin/matches/{matchId}/end` | Sets the match to `ended` and writes an `AdminAction`. |
| GET | `/admin/accounts?role=Senior` | Lists accounts for one role. The query `role` is optional. |
| POST | `/admin/accounts/{accountId}/lock` | Locks or unlocks an account and records the administrator. |
| GET | `/admin/reports?status=open` | Lists safety reports. |
| POST | `/admin/reports/{reportId}/outcome` | Stores the outcome and writes an `AdminAction`. |
| POST | `/admin/equipment-listings/{listingId}/hide` | Sets the listing to `unavailable` and writes an `AdminAction`. |
| GET | `/admin/actions` | Lists staff actions, newest first. |

### 8.5.1 Request an administrator code

| | |
| --- | --- |
| **Method and path** | `POST /admin/session/codes` |
| **Authentication** | None. The phone must already be an administrator account. |

Request body:

```json
{ "phone": "+962792222222" }
```

Success, `202 Accepted`:

```json
{ "status": "code_sent" }
```

| Check | Error |
| --- | --- |
| `phone` is missing or not a phone number. | `400` `invalid_phone` |
| The phone is not an administrator account. | `403` `forbidden` |
| That administrator is locked. | `403` `account_locked` |
| WhatsApp does not accept the message. | `502` `whatsapp_unavailable` |

`POST /admin/session/verify` uses the same body as [8.1.2](#812-verify-the-code). Success returns `accountId`, `role` `Admin`, and `token`. A wrong code returns `400` `invalid_code` and does not start a session.

### 8.5.2 Review a volunteer

| | |
| --- | --- |
| **Method and path** | `POST /admin/volunteers/{volunteerProfileId}/review` |
| **Authentication** | Bearer token with role `Admin`. |

Request body:

```json
{
  "decision": "approve",
  "notes": "Profile matches the submitted availability."
}
```

`decision` is `approve` or `reject`.

Success, `200 OK`:

```json
{
  "volunteerProfileId": "bb22cc33-dd44-4e55-8f66-778899001122",
  "verificationStatus": "Approved"
}
```

An `AdminAction` row stores the administrator, the volunteer account, the decision, and the note.

| Check | Error |
| --- | --- |
| `decision` is missing or not `approve` or `reject`. | `400` `invalid_review` |
| No volunteer profile has this id. | `404` `volunteer_not_found` |

### 8.5.3 Confirm a suggestion

| | |
| --- | --- |
| **Method and path** | `POST /admin/matches/{matchId}/confirmation` |
| **Authentication** | Bearer token with role `Admin`. |
| **Request body** | Empty. |

Success, `200 OK`:

```json
{
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "status": "offered"
}
```

The row stores `ConfirmedByAdminId`. The senior and the volunteer still accept before the status can become `active`.

| Check | Error |
| --- | --- |
| No match has this id. | `404` `match_not_found` |
| The match is not `proposed`. | `409` `match_not_ready` |
| The named volunteer is not `Approved`. | `409` `volunteer_not_approved`. The match stays `proposed`. |

### 8.5.4 End a match

| | |
| --- | --- |
| **Method and path** | `POST /admin/matches/{matchId}/end` |
| **Authentication** | Bearer token with role `Admin`. |

Request body:

```json
{ "notes": "Senior asked for a different volunteer." }
```

Success, `200 OK`:

```json
{
  "matchId": "6f1c0a2e-7b4d-4e1a-9c33-1a2b3c4d5e6f",
  "status": "ended"
}
```

The senior's one match row remains, with status `ended`. A later suggestion may replace the volunteer on that row. The action is stored in `AdminAction`.

| Check | Error |
| --- | --- |
| `notes` is missing. | `400` `invalid_match_change` |
| No match has this id. | `404` `match_not_found` |
| The match is already `ended` or `declined`. | `409` `match_closed` |

### 8.5.5 Lock or unlock an account

| | |
| --- | --- |
| **Method and path** | `POST /admin/accounts/{accountId}/lock` |
| **Authentication** | Bearer token with role `Admin`. |

Lock:

```json
{ "locked": true, "notes": "Safety report upheld." }
```

Unlock:

```json
{ "locked": false, "notes": "Review completed." }
```

Success after a lock, `200 OK`:

```json
{
  "accountId": "3a1b2c4d-5e6f-4a7b-8c9d-0e1f2a3b4c5d",
  "isLocked": true
}
```

A lock sets `IsLocked`, `LockedAt`, and `LockedByAdminId`, and keeps the account's other rows. If the target account is an administrator, `Admin.IsLocked` is set to the same value. An unlock clears `LockedAt` and `LockedByAdminId`. Both write an `AdminAction`.

| Check | Error |
| --- | --- |
| `locked` or `notes` is missing. | `400` `invalid_lock` |
| No account has this id. | `404` `account_not_found` |
| The caller is locking their own account. | `403` `forbidden` |

### 8.5.6 Record a report outcome

| | |
| --- | --- |
| **Method and path** | `POST /admin/reports/{reportId}/outcome` |
| **Authentication** | Bearer token with role `Admin`. |

Request body:

```json
{
  "outcome": "match_ended",
  "notes": "Match ended and the volunteer account locked."
}
```

`outcome` is `no_action`, `warning_sent`, `match_ended`, or `account_locked`. This call records the outcome. Ending the match and locking the account are the separate calls above.

Success, `200 OK`:

```json
{
  "reportId": "f0e1d2c3-b4a5-4968-8776-655443322110",
  "status": "closed",
  "outcome": "match_ended"
}
```

The report stores `ReviewedByAdminId`. An `AdminAction` stores the note.

| Check | Error |
| --- | --- |
| `outcome` or `notes` is missing, or `outcome` is not one of the four values. | `400` `invalid_outcome` |
| No report has this id. | `404` `report_not_found` |
| The report is already `closed`. | `409` `report_closed` |

### 8.5.7 Hide an equipment listing

| | |
| --- | --- |
| **Method and path** | `POST /admin/equipment-listings/{listingId}/hide` |
| **Authentication** | Bearer token with role `Admin`. |

Request body:

```json
{ "notes": "Listing removed after review." }
```

Success, `200 OK`:

```json
{
  "listingId": "e7f8a9b0-c1d2-4e3f-8a4b-5c6d7e8f9012",
  "status": "unavailable"
}
```

`ReviewedByAdminId` is set, and an `AdminAction` stores the note. A hidden listing is omitted from `GET /equipment-listings`.

| Check | Error |
| --- | --- |
| `notes` is missing. | `400` `invalid_listing` |
| No listing has this id. | `404` `listing_not_found` |
| The listing is already `unavailable`. | `409` `listing_unavailable` |

## 8.6 References

- Section 3, `03-system-context.md`, for the four roles and who may act for a senior.
- Section 6, `06-runtime-view.md`, for registration, match acceptance, and the call reminder.
- Section 7, `07-data-architecture.md`, for accounts, family-link flags, the one match per senior, help requests, equipment, reports, and `AdminAction`.
- Section 1, `01-introduction-goals-requirements.md`, for registration, matching, scheduling, Helping Hands, Community Exchange, and family consent.
- Section 5, `05-building-block-view.md`, for the single JSON API used by the mobile app and the admin dashboard.
