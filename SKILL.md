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

Run these steps in order. Step 0 is the freshness check, steps A to C produce the material, step D produces the file, step E is what the user reads.

### Step 0 — Check the skill is current

This skill describes a form that Discord changes without notice, and its field tree was mapped by hand. A stale copy does not merely go vague, it produces fields that no longer exist. So before anything else, compare the local `VERSION` file with the published one:

```bash
curl -fsS --max-time 5 https://raw.githubusercontent.com/creatorsarea/discord-privileged-intents/main/VERSION
```

Compare it with the `VERSION` file sitting next to this `SKILL.md`.

- **Same value, or the request failed** → say nothing and carry on. No network, a proxy, a firewall or a rate limit are all normal, and none of them is a reason to interrupt the user's task. This check fails open, always.
- **The published value is newer** → tell the user before starting the audit, in one line: the skill is out of date, the form tree may have moved since, and here is how to update. Then ask whether to update now or carry on with the local version. Do not update on your own, and do not refuse to work with an old copy.

The update is a `git pull` in the directory holding this `SKILL.md`. If that directory is not a git repository, the skill was installed by copying files rather than by cloning, and the fix is to re-clone it from the URL above.


### Step A — Audit the code BEFORE writing anything

Never write a justification without checking what the bot actually consumes.

1. Find where the intents are declared (discord.js: `GatewayIntentBits`; discord.py: `discord.Intents`; other libraries: the gateway identify payload).
2. For each privileged intent declared, look for **real consumers**:
   - `GuildMembers` → `guildMemberAdd` / `guildMemberRemove` / `guildMemberUpdate` listeners, `guild.members.fetch()` with no argument, iteration over `members.cache`, reliance on an accurate `guild.memberCount`.
   - `GuildPresences` → `presenceUpdate`, `member.presence`, `user.presence`.
   - `MessageContent` → `messageCreate` handlers reading `message.content` / `embeds` / `attachments`, excluding DMs and messages mentioning the bot, which stay readable without the intent.
   - While reading those `messageCreate` handlers, note separately whether the content is used to **parse a command prefix** (`message.content.startsWith('!')`, a legacy command framework, `command_prefix` in discord.py), and whether the moderation it performs is **already covered by the AutoMod API** (keyword lists, keyword presets, spam, mention spam). See step A bis.
3. Separate what **requires** the intent from what goes through REST:
   - `members.fetch(id)`, `fetchMe()`, `guild.members.search()` → **REST, no intent required**.
   - Member data received inside an interaction (slash command, button, modal) → **no intent required**.
   - What genuinely requires the intent: receiving member/presence **events** in real time, enumerating **all** members, reading the content of arbitrary messages.
4. Conclude with the list of intents that are **actually needed**, and for each one the feature(s) that break without it.
5. **Flag every intent that is declared at startup but has no real consumer in the code.** This is a finding the user must see: a superfluous request weakens the whole submission and is a common denial reason. Recommend removing it from the code rather than requesting it.

Official alternatives to suggest when an intent is not indispensable: see `references/discord-rules.md`, section "What is still possible WITHOUT an intent".

### Step A bis — The two documented dead ends

Two use cases are named in Discord's own documentation as things that do not earn the intent. Neither may appear in the form as a reason to need it, and the user has to be told before anything is written.

#### Prefix commands

If the audit finds that message content is read to parse a command prefix, this is the textbook denial.

Discord names this case explicitly. Its review checklist asks "Is my bot using prefix commands (`!help`, `?play`) that could be migrated to slash commands?", and it calls migrating text commands to slash commands "the most common reason developers request the Message Content privileged intent". Slash commands are the documented, supported replacement, so a request resting on prefix parsing is refused on the grounds that the alternative already exists.

What to do, depending on the audit:

- **Prefix parsing is the only use of message content** → do not fill in the form. Report that the intent is not obtainable for this use case, and that the way out is migrating the commands to application commands.
- **The bot has other genuine uses** (automod, logging, scam detection, triggers on spontaneous messages) → justify the intent on those alone, and **leave the prefix commands out of the answers entirely**. Mentioning them weakens an otherwise valid request, since it hands the reviewer a use case they are told to refuse.

Same rule for Q1: describing the bot as driven by `!` commands undermines the whole submission. Q1 describes what the app does, so name the features, not the prefix syntax used to reach them.

#### Moderation that AutoMod already does

Discord states that providing "what the AutoMod API already supports is generally not considered a compelling use case for access". AutoMod natively blocks keyword lists, keyword presets, spam and mention spam, so blocked words, invite links and obvious spam are not arguments, whatever the bot's own implementation is worth.

For every moderation feature found in step A, ask what AutoMod cannot do in its place: inspecting images or attachments, correlating messages across servers, reputation lookups on links, anything that depends on context rather than on matching a string. That gap is the justification, and it has to be written out.

- **AutoMod covers the whole feature** → say so to the user and recommend AutoMod rules instead of the intent.
- **The feature goes beyond AutoMod** → justify on the gap, and name AutoMod explicitly to show it was considered. A reviewer who sees the boundary spelled out reads an informed request.

### Step B — Map the stored data

The form asks the same "off-platform" questions three times. Answers must be factual and taken from the actual database schema, and from everywhere else the data lands:

- Which data coming from the intent is **persisted outside Discord** (IDs, usernames, message content, presences)?
- **Retention**: more than 30 days, or 30 days or less? Answer according to reality, not according to what sounds better.
- **Encryption at rest**: verify it for real (provider disk encryption, KMS, encrypted columns). TLS in transit is not encryption at rest.
- **Deletion channel**: bot command, privacy email, support form, support server. Give the exact address or URL.
- **If nothing is persisted**, confirm it: Discord asks applicants to state that the data is processed in memory and discarded immediately when it is not stored. A No closes the whole branch of the form, so there is nowhere left to explain it. Put the sentence in the `« Why do you need the <X> intent? »` textarea instead, the only free field of the section. A bare No reads as an oversight, or as something being hidden.

Note that storing a bare member `discord_id` **is** storing API data.

**"In memory" is narrower than it sounds, and the database is not the only place to look.** Before answering No, check the paths that write the data somewhere without anyone thinking of it as storage:

- **Application logs.** A `console.log` or a logger call that includes `message.content`, a username or a user ID writes API data to disk, and to any external log service the output is shipped to.
- **Error reporting** (Sentry and equivalents). A message captured in the context of an exception leaves the server and is retained by a third party.
- **Persisted caches.** A Redis with an append-only file or snapshots, a queue that survives a restart, a file-backed cache.
- **Backups and analytics** built on top of any of the above.

Processing in memory means the data lives in the scope of a function and disappears with it, without touching a disk or a third party. Grep for the intent's data in the logging and error-reporting call sites, not only in the schema. This claim is re-committed to at every yearly re-application, so it must be true rather than convenient.

### Step C — Write the answers

- A justification is a **named feature + the precise data consumed + why the REST or interaction alternative is not enough**. "My bot needs message content" is close to an automatic denial.
- Stay factual and verifiable: these answers are also the basis for next year's re-application.
- **Only request what you need**, and justify each intent separately. A weak third intent drags down the two solid ones, since the submission is reviewed as a whole.
- Leave no field blank or vague. Discord warns that an unclear or incomplete submission "may result in review delays or denial of your request".
- Evidence: screenshots or videos hosted at a public, stable URL. One capture per use case, showing the feature in action, annotated.
- Privacy policy: public URL, reachable without an account, explicitly covering the Discord data collected, the retention and the deletion channel.

### Step D — Deliverable

Produce a `discord-intents-request.md` file **in the project repository** (not in a scratch directory). The format is strict: **question/answer pairs only**, in the order of the form tree. The file must read like the form itself, field by field, and be copy-pasteable without edits. This is the file that will be re-read and amended next year.

**Required format** (shown below with a public privacy policy and off-platform storage, so the Yes branches of `references/form-tree.md` are the ones displayed; produce whichever branch the audit actually supports, and a No hides the fields under it):

````markdown
# Discord Request Intents

Requested intents: **<Intent Name>** (`GatewayIntentBits.<X>`).

---

## Q1. What does your application do?

```
<answer>
```

---

## Q2. Privacy Policy

**Do you have a public Privacy Policy telling your users about their data usage?**

```
Yes
```

**Where is your Privacy Policy available?**

```
<answer>
```

**Please share a link to your Privacy Policy.**

```
<URL>
```

---

## <Name> Intent

### « <exact field label> »

```
<answer>
```

### « Are you storing any API Data off-platform (outside of Discord)? »

```
Yes
```
````

**Rules:**

- **No em dash or en dash** (`—`, `–`), neither in the answers nor in the headings. It reads as generated text, and these answers must read as written by the team. Use a period, a colon, a comma or parentheses instead. Same for other generated-writing tics: no "it's not just X, it's Y", no decorative emphasis.
- Every answer sits in a **fenced code block**, including the Yes/No of the selects. The form's fields are plain textareas, so a fence shows exactly what will be pasted and gives a copy button on GitHub. Headings and field labels stay outside the fences, as ordinary markdown.
- **No markdown inside an answer.** A textarea renders nothing, so `**bold**` and `` `backticks` `` arrive as literal asterisks and backticks in the submission. Write plain sentences, and introduce a sub-part with a plain lead-in line ending in a colon rather than with bold.
- Intent block field labels are copied **verbatim from the form**, in guillemets (`### « … »`). Q1 and Q2 keep their numbering.
- **A select has no text field attached.** `Are you storing … off-platform?`, `… for 30 days or less?`, `Can users opt-out?`, `… train AI Models?`, `… encrypting at rest?` are bare Yes/No: the form offers nowhere to paste a justification. Never write a second explanatory block under a select, it would have nowhere to go. Anything that needs justifying (what is stored, where, why retention exceeds 30 days) belongs in the `« Why do you need the <X> intent? »` textarea, the only free-text field of the section, introduced by a plain lead-in line such as `What we keep from this intent:`.
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
