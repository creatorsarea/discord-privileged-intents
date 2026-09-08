# Answer patterns — Request Intents form

These templates are **skeletons to fill in with facts verified in the code**, never to copy as they are. Every `<...>` must disappear.

The prose around the placeholders is an example too, not a formula. Bots differ: some have a dashboard and some do not, some store nothing beyond a guild ID. Keep a sentence only if it is true of the bot being audited, and drop or rewrite it otherwise. A claim that does not match the app is worse than a missing one, since the reviewer checks the answers against the bot.

The deliverable contains **question/answer pairs only**, answers in blockquotes: see the required format in `SKILL.md`, step D. Anything that is not an answer to a form field (audit, checklist, TODO, warnings) is told to the user, not written in the file.

## Q1 — What does your application do?

Three blocks, 150 to 400 words:
1. One positioning sentence: who publishes the bot, for which audience, on how many servers.
2. A bullet list of the main features (named as they appear in the bot's UI or public documentation).
3. One sentence on the data model: what is stored and why.

> `<Bot name>` is the official Discord bot of `<product / brand>`, used by `<N>` communities to `<value delivered>`. Its main features are:
> - `<feature 1>`: `<what the user sees>`
> - `<feature 2>`: ...
>
> The bot is configured through `<how it is actually configured: slash commands, a web dashboard, a config file, ...>`. It stores only the configuration data required to deliver these features (`<list: guild IDs, channel IDs, ...>`).

## Q2 — Privacy Policy

- "Where is your Privacy Policy available?" → describe the location, not just the URL, and name only the places where it is really reachable: *"On our public website, linked from `<the bot's App Directory listing, the /help command, the dashboard footer, ...>`."*
- "Please share a link" → direct, public URL, no authentication.

## Server Members Intent — « Why do you need the Guild Members intent? »

One block per use case, repeated:

> **`<Feature>`**: we listen to `<GUILD_MEMBER_ADD | GUILD_MEMBER_UPDATE | GUILD_MEMBER_REMOVE>` in order to `<concrete action>`. Without the intent we would `<consequence: feature broken / unviable REST polling / etc.>`. The REST endpoints `Get Guild Member` and `Search Guild Members` are not sufficient here because `<reason: real time needed / no ID known in advance / volume>`.

Use cases that are typically accepted: reward roles synced on join or role change, welcome messages, invite tracking, leveling systems, moderation (raid protection), ticket systems.

## Presence Intent — « Why do you need the Guild Presences intent? »

> **`<Feature>`**: we read `presence.activities` to `<e.g. assign a "Live" role while a member is streaming on Twitch>`. `approximate_presence_count` is insufficient because we need `<per-member / per-activity>` data.

"Can users opt-out of having their Presence data tracked?" → answer Yes only if a real mechanism exists (command, dashboard setting, role-based opt-in), and describe it in the justification field.

## Message Content Intent — « Why do you need the Message Content intent? »

> **`<Feature>`**: `<e.g. Auto-moderation: we scan message content for phishing links, invite spam and blacklisted terms, and delete the offending message.>` This cannot be done with slash commands, context menus or modals because `<why the interaction path does not cover it>`.

The second sentence is the one the reviewer weighs, so it has to name the real obstacle. For automod and logging it is usually that the content is written spontaneously by members and never submitted through an interaction. For other use cases it is something else entirely, and copying that phrasing onto a bot it does not describe is worse than writing nothing.

"Will the message content data be used to train machine learning or AI Models?" → answer No if that is the case; if Yes, explain precisely in the justification (purpose, anonymisation, legal basis, opt-out).

## The "off-platform data" block (identical for all 3 intents)

- **"Are you storing any API Data off-platform?"** — Yes as soon as a Discord ID is written to a database. Do not answer No on the grounds that "they are only IDs".
- **"Are you storing API Data for 30 days or less?"** — answer according to the real retention. A No is perfectly acceptable when justified in the text field (e.g. server configuration must persist as long as the bot is installed).
- **"How do users contact you to request deletion of their activity data?"** —
  > Users can request deletion by `<channel 1: email privacy@..., a ticket on our support server <invite>, the /forgetme command>`. Requests are processed within `<N>` days. Data is also deleted automatically when `<the bot is removed from the server / the account is deleted>`.

  The last sentence goes in only if such an automatic deletion actually exists in the code. Drop it otherwise, rather than promising a cleanup that never runs.
- **"Are you encrypting the data that you store at rest?"** — Yes only if verified (provider disk or volume encryption, encrypted columns, KMS). TLS in transit does not count.

## « Please provide links to screenshots and/or videos »

> `<url>` — `<intent>` / `<feature>`: `<what the capture shows>`

One line per capture, with a stable public URL. The URL stays as `` `<url>` `` until the user provides the real link, and every such line must also appear in the reply to the user as a capture to take. Check that the links open in a private window.
