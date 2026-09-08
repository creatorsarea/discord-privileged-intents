# discord-privileged-intents

An agent skill that helps you fill in Discord's **Request Intents** form: the review you have to go
through to keep the privileged gateway intents (Server Members, Presence, Message Content) once your
app reaches 10,000 unique users.

The skill does four things:

1. **Audits your code** to find out which privileged intents your bot really consumes, and which ones
   it declares at startup without ever using. A requested intent that nothing in the code uses is one
   of the most common denial reasons.
2. **Keeps you off the documented denial paths.** Two of them are named in Discord's own docs and
   sink a lot of submissions: justifying Message Content with prefix commands, when the review
   checklist asks whether your `!help` could be a slash command, and justifying it with moderation
   that the AutoMod API already covers, which Discord says "is generally not considered a compelling
   use case for access". The skill spots both in your code and tells you before a single answer is
   written.
3. **Writes the answers** to every field of the form, in the order of the actual form tree, into a
   `discord-intents-request.md` file in your repository. Question and answer pairs only, nothing else,
   so it can be copy-pasted field by field, and re-read next year when Discord asks you to apply again.
4. **Tells you which screenshots to take.** Each capture the form needs is left as a `<url>`
   placeholder in the file, and listed in the agent's reply so you know exactly what to display and
   capture. Send the links back and the placeholders get filled in.

## Install

Clone it into your skills directory.

Personal skills, available in every project:

```bash
git clone https://github.com/creatorsarea/discord-privileged-intents.git \
  ~/.claude/skills/discord-privileged-intents
```

Project skills, committed with the repository and shared with your team:

```bash
git clone https://github.com/creatorsarea/discord-privileged-intents.git \
  .claude/skills/discord-privileged-intents
```

The layout is the standard one, so it works with any agent that reads `SKILL.md` skills, not only
Claude Code.

## Staying up to date

Discord changes the form without notice, and the field tree in here was mapped by hand, so an old copy is
actively misleading rather than merely incomplete. On each run the skill compares its local `VERSION` file
with the one published here, and warns you if yours is behind. It asks before doing anything, and never
updates on its own.

That check is a single request to `raw.githubusercontent.com`. It fails open: no network, a proxy or a
firewall simply skips it, and your task carries on. Nothing about your code or your bot leaves the machine.

To update:

```bash
git -C ~/.claude/skills/discord-privileged-intents pull
```

## Use

Open the agent in the repository of the bot you are applying for, then ask for it:

```
Prepare the Discord privileged intents request for this bot.
```

The skill loads on its own whenever privileged intents, the 10,000 user threshold, the yearly
re-application, or an intent denial come up.

## What is in here

| File | Contents |
|---|---|
| `SKILL.md` | Policy context, the execution procedure, and the required format of the deliverable |
| `references/form-tree.md` | Every field of the form and its conditional branching, mapped from the live form |
| `references/discord-rules.md` | What each intent unlocks, what REST alternatives cover without it, common denial reasons |
| `references/answer-patterns.md` | Reusable phrasing per intent and per question |
| `references/sources.md` | Discord's official documentation on the review |
| `VERSION` | Release date of this copy, used by the freshness check |

## Accuracy

The form tree was mapped by hand from the live Developer Portal on 2026-07-27, without ever
submitting the form. Discord changes it from time to time. If you find a field that has moved,
appeared or disappeared, a pull request on `references/form-tree.md` is welcome.

Nothing here is legal advice, and no skill can guarantee an approval. What it does guarantee is that
you answer the actual questions with facts taken from your own code.

## License

MIT, see [LICENSE](LICENSE).
