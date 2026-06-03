# Contact Conversation Page Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add an Aloware-style SMS conversation page for each contact, linked from the contacts table, showing pre-written message histories and a text input for practice.

**Architecture:** New static HTML page (`conversation.html`) with 2-panel layout (chat + sidebar), hardcoded conversation scripts per contact (tiered by richness), user messages persisted to localStorage. Chat icon added to contacts table linking to this page.

**Tech Stack:** Static HTML/CSS/JS, localStorage, DOM-safe rendering (createElement/textContent only)

---

### Task 1: Create conversation.html — Shell (HTML + CSS)

**Files:**

- Create: `conversation.html`

**Step 1: Create the file with full CSS and HTML structure**

The page has:

- Same site nav as other pages
- 2-panel layout: `.chat-panel` (left, flex:1) + `.side-panel` (right, 300px)
- Chat panel: header, scrollable messages area, input area
- Side panel: contact card, info fields, notes textarea

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1.0" />
    <title>Conversation — Dr. Pompa</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link
      href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700&family=IBM+Plex+Sans:wght@300;400;500;600&family=IBM+Plex+Mono:wght@400;500&display=swap"
      rel="stylesheet"
    />
    <style>
      :root {
        --s9: #1b2a31;
        --s8: #243841;
        --s7: #2f4a56;
        --s6: #3f5b68;
        --s5: #5a7682;
        --s4: #89a0ab;
        --s3: #b5c6cd;
        --s2: #dce5e9;
        --s1: #edf1f3;
        --t7: #1e8f86;
        --t6: #2db3a6;
        --t5: #4ec5b9;
        --t4: #87d8d0;
        --t2: #c9eae6;
        --t1: #e6f5f2;
        --paper: #f7f3ed;
        --bone: #fbf9f4;
        --ink: #1a262c;
        --ink2: #3d5059;
        --hs-blue: #0091ae;
        --hs-blue-light: #e5f5f8;
        --hs-dark: #33475b;
        --hs-gray: #516f90;
        --hs-border: #cbd6e2;
        --hs-bg: #f5f8fa;
        --hs-white: #ffffff;
        --purple: #7b1fa2;
        --purple-bg: rgba(123, 31, 162, 0.06);
        --purple-bd: rgba(123, 31, 162, 0.15);
        --blue: #1565c0;
        --blue-bg: rgba(21, 101, 192, 0.06);
        --blue-bd: rgba(21, 101, 192, 0.15);
        --green: #2e7d32;
        --green-bg: #ffffff;
        --green-bd: #dce5e9;
        --orange: #e65100;
        --orange-bg: rgba(230, 81, 0, 0.06);
        --orange-bd: rgba(230, 81, 0, 0.15);
        --teal: #1e8f86;
        --teal-bg: rgba(30, 143, 134, 0.08);
        --teal-bd: rgba(30, 143, 134, 0.2);
      }
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }
      body {
        background: var(--hs-bg);
        color: var(--hs-dark);
        font-family: "IBM Plex Sans", system-ui, sans-serif;
        line-height: 1.6;
        overflow: hidden;
        height: 100vh;
      }
      .site-nav {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        height: 64px;
        background: var(--bone);
        border-bottom: 1px solid var(--s2);
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0 40px;
        z-index: 100;
      }
      .nav-logo {
        display: flex;
        align-items: center;
        gap: 14px;
        text-decoration: none;
      }
      .nav-logo img {
        height: 36px;
      }
      .nav-links {
        display: flex;
        gap: 8px;
      }
      .nav-link {
        font-family: "IBM Plex Mono", monospace;
        font-size: 13px;
        font-weight: 500;
        letter-spacing: 0.06em;
        text-transform: uppercase;
        text-decoration: none;
        color: var(--s5);
        padding: 8px 16px;
        border-radius: 6px;
        transition: all 0.2s;
      }
      .nav-link:hover,
      .nav-link.active {
        color: var(--t7);
        background: var(--t1);
      }

      .app {
        display: flex;
        height: calc(100vh - 64px);
        margin-top: 64px;
      }

      /* Chat Panel */
      .chat-panel {
        flex: 1;
        display: flex;
        flex-direction: column;
        min-width: 0;
      }
      .chat-header {
        padding: 14px 24px;
        border-bottom: 1px solid var(--hs-border);
        background: var(--hs-white);
        display: flex;
        align-items: center;
        gap: 16px;
      }
      .chat-back {
        font-size: 13px;
        color: var(--hs-blue);
        text-decoration: none;
        display: flex;
        align-items: center;
        gap: 4px;
      }
      .chat-back:hover {
        text-decoration: underline;
      }
      .chat-header-info {
        display: flex;
        align-items: center;
        gap: 12px;
        flex: 1;
      }
      .chat-header-name {
        font-family: "Outfit", sans-serif;
        font-size: 18px;
        font-weight: 600;
        color: var(--hs-dark);
      }
      .chat-header-phone {
        font-size: 13px;
        color: var(--hs-gray);
      }
      .chat-header-pill {
        display: inline-block;
        padding: 2px 10px;
        border-radius: 12px;
        font-size: 11px;
        font-weight: 600;
      }
      .chat-reset {
        font-size: 12px;
        color: var(--hs-gray);
        background: none;
        border: 1px solid var(--hs-border);
        padding: 5px 12px;
        border-radius: 4px;
        cursor: pointer;
      }
      .chat-reset:hover {
        border-color: var(--hs-blue);
        color: var(--hs-blue);
      }

      .messages {
        flex: 1;
        overflow-y: auto;
        padding: 24px;
        display: flex;
        flex-direction: column;
        gap: 16px;
      }
      .msg {
        display: flex;
        gap: 10px;
        max-width: 72%;
      }
      .msg.au {
        align-self: center;
        max-width: 85%;
      }
      .msg.cl {
        align-self: flex-start;
      }
      .msg.ac {
        align-self: flex-end;
        flex-direction: row-reverse;
      }
      .msg.caller {
        align-self: flex-end;
        flex-direction: row-reverse;
      }
      .msg.us {
        align-self: flex-end;
        flex-direction: row-reverse;
      }
      .msg-avatar {
        width: 30px;
        height: 30px;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 11px;
        font-weight: 600;
        color: #fff;
        flex-shrink: 0;
      }
      .msg-body {
        display: flex;
        flex-direction: column;
        gap: 2px;
      }
      .msg-bubble {
        padding: 12px 16px;
        border-radius: 12px;
        font-size: 13px;
        line-height: 1.7;
      }
      .msg-bubble.au {
        background: var(--purple-bg);
        border: 1px solid var(--purple-bd);
      }
      .msg-bubble.cl {
        background: var(--green-bg);
        border: 1px solid var(--green-bd);
      }
      .msg-bubble.ac {
        background: var(--blue-bg);
        border: 1px solid var(--blue-bd);
      }
      .msg-bubble.caller {
        background: var(--orange-bg);
        border: 1px solid var(--orange-bd);
      }
      .msg-bubble.us {
        background: var(--teal-bg);
        border: 1px solid var(--teal-bd);
      }
      .msg-meta {
        font-size: 10px;
        color: var(--hs-gray);
        font-family: "IBM Plex Mono", monospace;
      }
      .msg-sender {
        font-size: 11px;
        font-weight: 600;
        margin-bottom: 2px;
      }
      .msg-sender.au {
        color: var(--purple);
      }
      .msg-sender.cl {
        color: var(--green);
      }
      .msg-sender.ac {
        color: var(--blue);
      }
      .msg-sender.caller {
        color: var(--orange);
      }
      .msg-sender.us {
        color: var(--teal);
      }

      .input-area {
        padding: 16px 24px;
        border-top: 1px solid var(--hs-border);
        background: var(--hs-white);
        display: flex;
        gap: 10px;
        align-items: flex-end;
      }
      .input-area textarea {
        flex: 1;
        font-family: "IBM Plex Sans", system-ui, sans-serif;
        font-size: 14px;
        padding: 10px 14px;
        border: 1px solid var(--hs-border);
        border-radius: 8px;
        resize: none;
        min-height: 42px;
        max-height: 120px;
        line-height: 1.5;
      }
      .input-area textarea:focus {
        outline: none;
        border-color: var(--hs-blue);
        box-shadow: 0 0 0 2px rgba(0, 145, 174, 0.1);
      }
      .input-area button {
        font-family: "IBM Plex Sans", system-ui, sans-serif;
        font-size: 13px;
        font-weight: 600;
        padding: 10px 20px;
        background: var(--teal);
        color: #fff;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        transition: background 0.15s;
      }
      .input-area button:hover {
        background: #177a72;
      }

      /* Side Panel */
      .side-panel {
        width: 300px;
        min-width: 300px;
        border-left: 1px solid var(--hs-border);
        background: var(--hs-white);
        overflow-y: auto;
        display: flex;
        flex-direction: column;
      }
      .side-header {
        padding: 20px;
        border-bottom: 1px solid var(--hs-border);
        display: flex;
        align-items: center;
        gap: 12px;
      }
      .side-avatar {
        width: 44px;
        height: 44px;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 16px;
        font-weight: 600;
        color: #fff;
        flex-shrink: 0;
      }
      .side-name {
        font-family: "Outfit", sans-serif;
        font-size: 15px;
        font-weight: 600;
        color: var(--hs-dark);
      }
      .side-phone {
        font-size: 12px;
        color: var(--hs-gray);
      }
      .side-section {
        padding: 16px 20px;
        border-bottom: 1px solid var(--hs-border);
      }
      .side-section h4 {
        font-size: 11px;
        font-weight: 600;
        text-transform: uppercase;
        letter-spacing: 0.05em;
        color: var(--hs-gray);
        margin-bottom: 10px;
      }
      .side-field {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 4px 0;
        font-size: 13px;
      }
      .side-field-label {
        color: var(--hs-gray);
      }
      .side-field-value {
        color: var(--hs-dark);
        font-weight: 500;
        text-align: right;
        max-width: 60%;
      }
      .side-pips {
        display: flex;
        gap: 3px;
        align-items: center;
      }
      .side-pip {
        width: 8px;
        height: 8px;
        border-radius: 50%;
        background: var(--hs-border);
      }
      .side-pip.on {
        background: var(--hs-blue);
      }
      .side-notes {
        padding: 16px 20px;
        flex: 1;
        display: flex;
        flex-direction: column;
      }
      .side-notes h4 {
        font-size: 11px;
        font-weight: 600;
        text-transform: uppercase;
        letter-spacing: 0.05em;
        color: var(--hs-gray);
        margin-bottom: 8px;
      }
      .side-notes textarea {
        flex: 1;
        min-height: 100px;
        font-family: "IBM Plex Sans", system-ui, sans-serif;
        font-size: 12px;
        padding: 10px;
        border: 1px solid var(--hs-border);
        border-radius: 6px;
        resize: vertical;
        line-height: 1.5;
      }
      .side-notes textarea:focus {
        outline: none;
        border-color: var(--hs-blue);
      }

      .status-pill-wr {
        background: #fff3cd;
        color: #856404;
      }
      .status-pill-r {
        background: #d4edda;
        color: #155724;
      }
      .status-pill-ready {
        background: #cce5ff;
        color: #004085;
      }
      .status-pill-confirmed {
        background: #d1ecf1;
        color: #0c5460;
      }
      .status-pill-nc {
        background: #f8d7da;
        color: #721c24;
      }
      .status-pill-canceled {
        background: #e2e3e5;
        color: #383d41;
      }
      .status-pill-blank {
        background: #f5f5f5;
        color: #999;
      }

      @media (max-width: 900px) {
        .side-panel {
          display: none;
        }
        .msg {
          max-width: 88%;
        }
      }
    </style>
  </head>
  <body>
    <nav class="site-nav">
      <a href="index.html" class="nav-logo"
        ><img src="logo.png" alt="Dr. Pompa"
      /></a>
      <div class="nav-links">
        <a href="index.html" class="nav-link">Home</a>
        <a href="quiz.html" class="nav-link">Quiz</a>
        <a href="simulator.html" class="nav-link">Simulator</a>
        <a href="live-testing.html" class="nav-link">Live Testing</a>
        <a href="contacts.html" class="nav-link active">Contacts</a>
      </div>
    </nav>
    <div class="app">
      <div class="chat-panel">
        <div class="chat-header" id="chatHeader"></div>
        <div class="messages" id="messages"></div>
        <div class="input-area" id="inputArea"></div>
      </div>
      <div class="side-panel" id="sidePanel"></div>
    </div>
    <script>
      // JS will be added in Task 2 & 3
    </script>
  </body>
</html>
```

**Step 2: Verify**

Open `conversation.html` in browser. Should see the nav, empty 2-panel layout, no errors in console.

**Step 3: Commit**

```bash
git add conversation.html
git commit -m "feat: add conversation.html shell with 2-panel layout CSS"
```

---

### Task 2: Add Conversation Data

**Files:**

- Modify: `conversation.html` (inside `<script>` tag)

**Step 1: Add the CONVERSATIONS object**

This is the hardcoded message data. Each contact ID maps to an array of messages. Message format:

```javascript
{sender: "Name", text: "...", time: "May 1, 10:08 AM ET", type: "automated|client|ac|caller"}
```

Write the full CONVERSATIONS object with:

- **Tier 1** (8-15 messages each): c1 (Jammie Castle - S1-NR, shows automated + client reply + AC intro + .verify but no response yet), c5 (Marcus Reeves - Responsive, full thread through NTA delivery), c7 (Daniel Okafor - Ready, full thread through checklist complete), c9 (Robert Fitzgerald - Confirmed, full arc including confirmation), c13 (Frank DeLuca - P1 priority, shows caller escalation)
- **Tier 2** (3-6 messages each): c3, c4, c6, c8, c10, c11, c12, c16, c18, c19
- **Tier 3** (1-2 messages): c2, c14, c15, c17, c20, c21, c22, c23, c24, c25

The conversations should use realistic AC messaging patterns from the live-testing page:

- Automated intro: "[AUTOMATED] Hey [Name], Dr Pompa's Team here. I saw you booked a consultation with us, glad to be able to help. What is the main health challenge on your mind right now?"
- AC intro pattern: acknowledge health concern, introduce self, mention HA name and date
- .verify pattern: ask timezone, confirm appointment in their local time
- NTA delivery: frame assessment as what helps HA prepare
- Edification: build trust in HA before the call
- Confirmation: day-of/day-before check-in

Use contact data from `contacts.html` seed (names, closerOwner as HA, meetingStartDate for appointment references).

**Step 2: Verify**

Open browser console, confirm `CONVERSATIONS` object exists with 25 entries and no syntax errors.

**Step 3: Commit**

```bash
git add conversation.html
git commit -m "feat: add conversation scripts for all 25 contacts (tiered)"
```

---

### Task 3: Add Rendering JS

**Files:**

- Modify: `conversation.html` (inside `<script>` tag)

**Step 1: Add helper functions and rendering logic**

Required functions:

- `getContactId()` — parse `?id=X` from URL
- `getContact()` — load contact from `ac-hubspot-contacts` localStorage
- `getSaved(id)` — load user messages/notes from `ac-convo-{id}` localStorage
- `saveSent(id, text)` — append user message to localStorage
- `saveNotes(id, text)` — save notes to localStorage
- `resetConvo(id)` — clear user messages from localStorage, re-render
- `renderHeader(contact)` — render chat header with name, phone, status pill, back link, reset button
- `renderMessages(contact, convo, saved)` — render all message bubbles (pre-written + user-sent)
- `renderInput()` — render textarea + send button with Enter key handler
- `renderSidebar(contact)` — render condensed contact info, checklist pips, notes
- `statusPillClass(status)` — return CSS class for status pill
- `avatarColor(name)` — deterministic color from name (same algo as contacts.html)
- `initials(first, last)` — get initials
- `fmtDate(iso)` — format ISO date to readable
- `timeUntil(iso)` — compute "X days" until meeting

Key rendering details:

- Messages scroll to bottom on load
- Auto-resize textarea on input
- Enter sends (Shift+Enter for newline)
- User messages get timestamp from `new Date()`
- Notes auto-save on input (debounced)
- "Back to Contacts" link preserves active view tab via localStorage

**Step 2: Verify**

Open `conversation.html?id=c9` in browser:

- Should see Robert Fitzgerald's full conversation thread
- Sidebar shows his contact info
- Can type and send a message (appears as teal bubble)
- Reset clears sent messages
- Refresh preserves sent messages

**Step 3: Commit**

```bash
git add conversation.html
git commit -m "feat: add conversation rendering, text input, localStorage persistence"
```

---

### Task 4: Add Chat Icon to Contacts Table

**Files:**

- Modify: `contacts.html` (in `buildRow()` function, inside the contact-cell td)

**Step 1: Add chat icon SVG next to contact name**

In the `buildRow()` function, after the contact name anchor (`<a class="contact-name">`), append a small chat icon that links to `conversation.html?id={contactId}`.

The icon should be:

- 14px speech bubble SVG
- Color: `var(--hs-gray)` default, `var(--hs-blue)` on hover
- Styled as an `<a>` tag with `class="chat-icon"`
- Has `title="View conversation"`
- Small margin-left from the name

Add CSS for `.chat-icon`:

```css
.chat-icon {
  display: inline-flex;
  align-items: center;
  margin-left: 6px;
  color: var(--hs-gray);
  transition: color 0.15s;
  vertical-align: middle;
}
.chat-icon:hover {
  color: var(--hs-blue);
}
.chat-icon svg {
  width: 14px;
  height: 14px;
}
```

The SVG (speech bubble):

```html
<svg
  viewBox="0 0 24 24"
  fill="none"
  stroke="currentColor"
  stroke-width="2"
  stroke-linecap="round"
  stroke-linejoin="round"
>
  <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z" />
</svg>
```

**Step 2: Verify**

Open `contacts.html`, confirm each contact row shows a small chat icon next to the name. Click it — should navigate to `conversation.html?id=cX`.

**Step 3: Commit**

```bash
git add contacts.html
git commit -m "feat: add chat icon link to conversation page in contacts table"
```

---

### Task 5: Deploy and Verify

**Step 1: Deploy to Vercel**

```bash
mkdir -p .vercel/output/static && cp *.html *.png .vercel/output/static/ && echo '{"version":3}' > .vercel/output/config.json && vercel deploy --prebuilt --prod --yes
```

**Step 2: End-to-end verification on production**

1. Open `pompa-precall.vercel.app/contacts.html`
2. Verify chat icons appear next to all contact names
3. Click a chat icon for Robert Fitzgerald (c9)
4. Verify full conversation thread loads with all message types
5. Verify sidebar shows correct contact info
6. Type a message and send — verify it appears as teal bubble
7. Refresh — verify sent message persists
8. Click Reset — verify only user messages clear
9. Click "Back to Contacts" — verify returns to contacts page
10. Test a Tier 3 contact (e.g., c2 Ravikanth) — should show minimal thread

**Step 3: Commit any fixes, redeploy if needed**
