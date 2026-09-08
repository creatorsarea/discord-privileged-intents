---
name: discord-privileged-intents
description: Fill in or review the Discord "Request Intents" form (privileged gateway intents — Server Members, Presence, Message Content), audit which intents a bot actually uses, and prepare the justifications, screenshots and data storage answers. Use whenever privileged intents, the 10,000 user threshold, the yearly re-application, or an intent denial/revocation by Discord comes up.
---

# Discord — Privileged Intents Review

## 1. Policy context (June 2026)

- The review threshold is no longer "100 servers" but **10,000 unique users** able to see the app, across all servers.
  - `< 10,000 users` → intents can be toggled freely in the Developer Portal, no form.
  - `>= 10,000 users` → Discord notifies the developer; **90 days** to submit the form.
- **Yearly re-application**: any access already granted must be requested again every year (notice is sent). Answers must therefore be archived and versioned in the project.
- **Bot verification at 100 servers is still a separate, unchanged process.**
- During the review the bot keeps working and can still join new servers.
- On denial or non-submission the intent is revoked, and code that still declares it fails the gateway handshake with **`[DisallowedIntents]`**.

Sources: `references/sources.md`.

## 2. Execution procedure

Run these steps in order. Steps A to C produce the material, step D produces the file, step E is what the user reads.

### Step A — Audit the code BEFORE writing anything

Never write a justification without checking what the bot actually consumes.

1. Find where the intents are declared (discord.js: `GatewayIntentBits`; discord.py: `discord.Intents`; other libraries: the gateway identify payload).
2. For each privileged intent declared, look for **real consumers**:
   - `GuildMembers` → `guildMemberAdd` / `guildMemberRemove` / `guildMemberUpdate` listeners, `guild.members.fetch()` with no argument, iteration over `members.cache`, reliance on an accurate `guild.memberCount`.
   - `GuildPresences` → `presenceUpdate`, `member.presence`, `user.presence`.
   - `MessageContent` → `messageCreate` handlers reading `message.content` / `embeds` / `attachments`, excluding DMs and messages mentioning the bot, which stay readable without the intent.
3. Separate what **requires** the intent from what goes through REST:
   - `members.fetch(id)`, `fetchMe()`, `guild.members.search()` → **REST, no intent required**.
   - Member data received inside an interaction (slash command, button, modal) → **no intent required**.
   - What genuinely requires the intent: receiving member/presence **events** in real time, enumerating **all** members, reading the content of arbitrary messages.
4. Conclude with the list of intents that are **actually needed**, and for each one the feature(s) that break without it.
5. **Flag every intent that is declared at startup but has no real consumer in the code.** This is a finding the user must see: a superfluous request weakens the whole submission and is a common denial reason. Recommend removing it from the code rather than requesting it.

Official alternatives to suggest when an intent is not indispensable: see `references/discord-rules.md`, section "What is still possible WITHOUT an intent".

### Step B — Map the stored data

The form asks the same "off-platform" questions three times. Answers must be factual and taken from the actual database schema:

- Which data coming from the intent is **persisted outside Discord** (IDs, usernames, message content, presences)?
- **Retention**: more than 30 days, or 30 days or less? Answer according to reality, not according to what sounds better.
- **Encryption at rest**: verify it for real (provider disk encryption, KMS, encrypted columns). TLS in transit is not encryption at rest.
- **Deletion channel**: bot command, privacy email, support form, support server. Give the exact address or URL.

Note that storing a bare member `discord_id` **is** storing API data.

### Step C — Write the answers

- A justification is a **named feature + the precise data consumed + why the REST or interaction alternative is not enough**. "My bot needs message content" is close to an automatic denial.
- Stay factual and verifiable: these answers are also the basis for next year's re-application.
- Evidence: screenshots or videos hosted at a public, stable URL. One capture per use case, showing the feature in action, annotated.
- Privacy policy: public URL, reachable without an account, explicitly covering the Discord data collected, the retention and the deletion channel.

### Step D — Deliverable

Produce a `discord-intents-request.md` file **in the project repository** (not in a scratch directory). The format is strict: **question/answer pairs only**, in the order of the form tree. The file must read like the form itself, field by field, and be copy-pasteable without edits. This is the file that will be re-read and amended next year.

**Required format:**

```markdown
# Discord Request Intents

Requested intents: **<Intent Name>** (`GatewayIntentBits.<X>`) and **<Intent Name>** (`GatewayIntentBits.<Y>`).

---

## Q1. What does your application do?

> <answer>

---

## Q2. Privacy Policy

**Do you have a public Privacy Policy?**

> Yes

**Where is your Privacy Policy available?**

> <answer>

**Please share a link to your Privacy Policy.**

> <URL>

---

## <Name> Intent

### « <exact field label> »

> <answer>

### « Are you storing any API Data off-platform (outside of Discord)? »

> Yes
```

**Rules:**

- **No em dash or en dash** (`—`, `–`), neither in the answers nor in the headings. It reads as generated text, and these answers must read as written by the team. Use a period, a colon, a comma or parentheses instead. Same for other generated-writing tics: no "it's not just X, it's Y", no decorative emphasis.
- Every answer is a **blockquote** (`>`), including the Yes/No of the selects.
- Intent block field labels are copied **verbatim from the form**, in guillemets (`### « … »`). Q1 and Q2 keep their numbering.
- **A select has no text field attached.** `Are you storing … off-platform?`, `… for 30 days or less?`, `Can users opt-out?`, `… train AI Models?`, `… encrypting at rest?` are bare Yes/No: the form offers nowhere to paste a justification. Never write a second explanatory blockquote under a select, it would have nowhere to go. Anything that needs justifying (what is stored, where, why retention exceeds 30 days) belongs in the `« Why do you need the <X> intent? »` textarea, the only free-text field of the section, under a dedicated subheading such as `**What we keep from this intent.**`.
- `---` separator between top-level sections (Q1, Q2, each intent).
- **Nothing else in the file**: no audit table, no submission checklist, no internal note, no `TODO`, no warning. The step A audit and any blocking points are reported **in the reply to the user**, not in the deliverable.
- The only `<...>` placeholders left in the file are the **screenshot URLs the user still has to provide**, left as `` `<url>` `` inside the caption line, which is already written out.
- Only produce the fields actually displayed according to `references/form-tree.md`: an intent that is not being requested does not appear at all.

### Step E — Reply to the user

The reply is where everything that must not pollute the deliverable goes. It must contain:

1. **The audit result**: which intents are needed and for which features, and above all **which intents the code declares at startup without ever using them**, with the recommendation to remove them from the code.
2. **The list of screenshots to take**, one line per capture, phrased so the user knows exactly what to display and capture (which feature, in which state, what must be visible on screen). Each of these captures matches a `` `<url>` `` placeholder in the file, so the user can send the links back and the placeholders can be filled in.
3. Anything still missing or blocking: unverified retention, encryption at rest not confirmed, privacy policy not public.

## 3. Form tree

Full field structure and conditional branching: `references/form-tree.md`. Re-read it before writing, so as to produce only the fields actually displayed for the given answers.

## 4. Answer patterns

Reusable phrasing per intent and per question: `references/answer-patterns.md`.
