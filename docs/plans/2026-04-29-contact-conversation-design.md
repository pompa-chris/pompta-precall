# Design: Contact Conversation Page

## Overview

A new `conversation.html?id=X` page showing an Aloware-style SMS thread for each contact. Trainees read pre-written conversation histories that demonstrate what AC messaging looks like at each pipeline stage, then practice composing their own messages via a text input.

Linked from the contacts table via a small chat icon next to each contact name.

## Layout

2-panel: Chat thread (left, ~70%) + condensed HubSpot sidebar (right, ~30%).

```
+-----------------------------------+-------------------------+
| Thread Header                     | Sidebar Header          |
| (name, phone, status, <- Back)    | (avatar, name, phone)   |
+-----------------------------------+-------------------------+
|                                   | Contact Info card        |
| Message bubbles (scrollable)      | (status, owner, HA,     |
| - System (purple, centered)       |  appt date, timezone)   |
| - Client (green, left-aligned)    |                         |
| - AC (blue, right-aligned)        | Pre-Call Checklist       |
| - Caller (orange, right-aligned)  |                         |
| - User sent (teal, right)         | NTQ Summary             |
|                                   |                         |
+-----------------------------------+ Notes (editable)        |
| Text input + Send button          |                         |
| [Reset conversation]              |                         |
+-----------------------------------+-------------------------+
```

## Conversation Data (Phased)

### Tier 1 - Full threads (5 contacts)

Jammie Castle, Marcus Reeves, Daniel Okafor, Robert Fitzgerald, Frank DeLuca

- 8-15 messages each showing complete arc for their stage
- Includes: automated intro, client reply, AC intro, .verify sequence, NTA delivery, edification, confirmation (as far as their stage dictates)

### Tier 2 - Medium threads (~10 contacts)

Suzette Sommers, Sharon Wood, Linda Kowalski, Teresa Huang, Angela Petrov, William Tran, Priya Sharma, Gregory Walsh, Rachel Nguyen, Diane Moretti

- 3-6 messages showing partial progress

### Tier 3 - Minimal threads (~10 contacts)

Ravikanth Bandari, Kevin Park, Jasmine Brooks, Patrick O'Brien, Maria Gonzalez, Brian Chambers, Stephanie Kim, Anthony Russo, Nancy Bergstrom, Carlos Mendez

- Just the automated message + maybe 1 client reply (or no reply for S1-NR)

## Message Types & Styling

| Type             | Class    | Alignment | Color            | Avatar   |
| ---------------- | -------- | --------- | ---------------- | -------- |
| System/Automated | `au`     | center    | purple bg        | SYS      |
| Client           | `cl`     | left      | white bg, border | Initials |
| AC (Texter)      | `ac`     | right     | blue bg          | B        |
| Caller           | `caller` | right     | orange bg        | J        |
| User-sent        | `us`     | right     | teal bg          | You      |

## Text Input Behavior

- Trainee types message, presses Enter or clicks Send
- Message appends as "sent" bubble (teal, right-aligned)
- Sent messages persist in localStorage: key `ac-convo-{contactId}`, value `[{text, time}]`
- "Reset" button clears user-sent messages only (pre-written thread remains)
- No AI feedback or scoring

## Contacts Table Integration

- Small speech-bubble SVG icon (14px, gray, blue on hover) added next to contact name in `.contact-cell`
- Links to `conversation.html?id={contactId}`
- Does not replace existing name link to contact-detail.html

## Sidebar Content

Condensed version of contact-detail fields:

- Avatar + name + phone
- CC Confirmation Status (read-only pill)
- Health Advisor (closerOwner)
- Scheduled Appt Date
- Time Until Call (computed)
- Pre-Call Checklist (read-only pip dots)
- NTQ % (if available)
- Editable notes textarea (persisted to localStorage alongside messages)

## Data Storage

- Conversation scripts: hardcoded in HTML as JS array (same pattern as live-testing.html)
- User messages: localStorage key `ac-convo-{id}` as `{msgs: [{text, time}], notes: ""}`
- Contact data: reads from existing `ac-hubspot-contacts` localStorage (shared with contacts.html)

## Tech Stack

Same as rest of project: static HTML/CSS/JS, no framework, no build step. DOM-safe rendering (createElement/textContent, no innerHTML with dynamic data).
