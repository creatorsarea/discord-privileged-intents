# Discord rules — privileged intents

## The 3 privileged intents

| Intent | Gives access to |
|---|---|
| `GUILD_MEMBERS` (Server Members) | `GUILD_MEMBER_ADD/UPDATE/REMOVE` events, full member list, `GUILD_MEMBER_CHUNK`, thread members |
| `GUILD_PRESENCES` (Presence) | Online status, activities (game, Spotify, streaming), platform (desktop/mobile/web) |
| `MESSAGE_CONTENT` | `content`, `embeds`, `attachments`, `components`, `poll` of **other people's** messages, in guilds |

Not privileged and often confused with these: `GUILD_MESSAGES` (receiving the `messageCreate` event with metadata, without the content). It is not part of the form.

## What is still possible WITHOUT an intent (official alternatives)

### Without Guild Members
- `GET /guilds/{id}/members/{user.id}` (Get Guild Member), targeted fetch by ID.
- `GET /guilds/{id}/members/search`, search by username.
- The full `member` object is provided in **every interaction** (slash command, button, select, modal, context menu).
- `approximate_member_count` on the Guild object with `with_counts=true`.
- `Get User` for basic profile data (username, avatar) when no guild-specific member data is needed.
- Enough when: only targeted lookups are needed, no enumeration and no real-time events.

### Without Presence
- `approximate_presence_count` on the Guild object (approximate number of online members).
- `Get User` for a user's basic profile information.
- **Setting the bot's own status or activity does not require the intent.** This is a frequent misunderstanding: `GUILD_PRESENCES` is about reading other people's presence, not about publishing your own.
- Enough when: displaying a presence counter rather than tracking individuals.

### Without Message Content
- Slash commands (arguments already parsed), context menu commands (the targeted message is provided **with** its content), buttons and selects, modals.
- Content stays readable in **DMs**, in messages that **mention the bot**, in **replies to the bot's messages**, and in messages **sent by the bot itself**.
- Enough when: user input goes through structured interactions.

**AutoMod covers a large part of moderation, and Discord expects you to use it.** The docs state that providing "what the AutoMod API already supports is generally not considered a compelling use case for access". AutoMod natively handles keyword filters, keyword presets, spam and mention spam, so a bot that asks for Message Content to block banned words, invite links or obvious spam is asking for something the platform already gives it. A moderation use case only justifies the intent through what AutoMod cannot do, and the justification has to say which part that is.

**Prefix commands are the one case Discord names explicitly.** Its review checklist asks "Is my bot using prefix commands (`!help`, `?play`) that could be migrated to slash commands?", and calls migrating text commands to slash commands "the most common reason developers request the Message Content privileged intent". Slash commands are presented as the direct replacement. So reading message content in order to parse a command prefix is not a justification, it is the textbook denial: the alternative is documented, supported and expected. A bot whose only use of the content is prefix parsing should migrate rather than apply.

## Common denial reasons

1. Generic or non-specific justification ("my bot needs it").
2. A documented alternative above already covers the use case.
3. Incomplete submission: no screenshots, no privacy policy, vague answers about retention.
4. Use case not compliant with the Developer Policy or Terms (scraping, reselling data, training models on user content without a clear legal basis).
5. Intent requested but not used in the code.
6. Message Content requested to parse prefix commands (`!help`, `?play`), which slash commands replace.
7. Moderation use case that duplicates what the AutoMod API already does.
8. Requesting several intents at once when only one is justified. Discord's guidance is to "only request what you need" and to justify each intent on its own.
9. Submission that is unclear or incomplete. Discord warns that this "may result in review delays or denial of your request", so an unanswered field is a risk in itself, not a neutral blank.

## Consequences

- Denial or no submission within 90 days: the intent is revoked.
- A client that keeps declaring the intent receives `[DisallowedIntents]` (gateway close code `4014`) and no longer connects at all. The intent must be removed from the code before the revocation takes effect.
- Re-applying after a denial is possible, with an improved submission.

## Operational requirements to comply with (developer policy)

- Encryption of API data **at rest**.
- Retention limited to what is strictly necessary; deletion on user request.
- Public privacy policy describing the Discord data collected.
