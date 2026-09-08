# Discord Developer Portal — "Request Intents" form tree

Form tested at: `https://discord.com/developers/applications/.../request-additional-intents`
(tested without ever clicking Submit)

## General structure

The form has an always-visible "Application Details" section (description textarea plus a "Privacy Policy" select), then a "Privileged Gateway Intents" section with 3 independent checkboxes (not exclusive, they can be combined) each revealing its own block of sub-questions, and finally the acknowledgement and submit.

## Full tree

```
"Request Intents" FORM

├── Q1: Textarea "What does your application do?" (always visible, no branching)

├── Q2: Select "Do you have a public Privacy Policy telling your users about their data usage?" [Yes/No]
│   ├── IF "Yes" →
│   │   ├── Q2a: Textarea "Where is your Privacy Policy available?"
│   │   └── Q2b: Textarea "Please share a link to your Privacy Policy."
│   └── IF "No" → (nothing more)

├── "Privileged Gateway Intents" SECTION (3 independent checkboxes, combinable)
│
│   ├── CHECKBOX "Server Members Intent"
│   │   └── IF checked → "Server Members Intent" block
│   │       ├── Textarea "Why do you need the Guild Members intent?"
│   │       ├── Textarea "Please provide links to screenshots and/or videos that demonstrate your use case"
│   │       └── Select "Are you storing any API Data off-platform (outside of Discord)?" [Yes/No]
│   │           ├── IF "Yes" →
│   │           │   ├── Select "Are you storing API Data for 30 days or less?" [Yes/No] (no sub-field, whatever the answer)
│   │           │   ├── Textarea "How do users contact you to request deletion of their activity data?"
│   │           │   └── Select "Are you encrypting the data that you store at rest, as is required by our developer policy?" [Yes/No] (no sub-field)
│   │           └── IF "No" → (nothing more)
│   │
│   ├── CHECKBOX "Presence Intent"
│   │   └── IF checked → "Presence Intent" block
│   │       ├── Textarea "Why do you need the Guild Presences intent?"
│   │       ├── Textarea "Please provide links to screenshots and/or videos that demonstrate your use case"
│   │       ├── Select "Can users opt-out of having their Presence data tracked?" [Yes/No] (no sub-field)
│   │       └── Select "Are you storing user activity data off-platform (outside of Discord)?" [Yes/No]
│   │           ├── IF "Yes" →
│   │           │   ├── Select "Are you storing user activity data for 30 days or less?" [Yes/No] (no sub-field)
│   │           │   ├── Textarea "How do users contact you to request deletion of their activity data?"
│   │           │   └── Select "Are you encrypting the data that you store at rest, as is required by our developer policy?" [Yes/No] (no sub-field)
│   │           └── IF "No" → (nothing more)
│   │
│   └── CHECKBOX "Message Content Intent"
│       └── IF checked → "Message Content Intent" block
│           ├── Select "Can users opt-out of having their message content data tracked?" [Yes/No] (no sub-field)
│           ├── Select "Are you storing message content data off-platform (outside of Discord)?" [Yes/No]
│           │   ├── IF "Yes" →
│           │   │   ├── Select "Are you storing user message content data for 30 days or less?" [Yes/No] (no sub-field)
│           │   │   ├── Textarea "How do users contact you to request deletion of their activity data?"
│           │   │   └── Select "Are you encrypting the data that you store at rest, as is required by our developer policy?" [Yes/No] (no sub-field)
│           │   └── IF "No" → (nothing more)
│           ├── Select "Will the message content data be used to train machine learning or AI Models?" [Yes/No]
│           │   └── (static note attached: "If yes, please explain how in the text box below..." which refers to the "Why do you need..." textarea below; this is NOT an extra hidden field, that textarea is always displayed)
│           ├── Textarea "Why do you need the Message Content intent?" (always visible once the checkbox is checked)
│           └── Textarea "Please provide links to screenshots and/or videos that demonstrate your use case" (always visible once the checkbox is checked)

└── Acknowledgement: Checkbox "By clicking Submit, you certify..." (always visible, no branching)
```

## Things worth knowing

- The 3 intent checkboxes (Server Members, Presence, Message Content) are **independent and combinable**: checking several at once displays several blocks simultaneously, with no interaction between them.
- The "off-platform data" pattern is **identical and repeated three times** (once per intent), with exactly the same 3 cascading sub-questions: 30 days or less, deletion contact, encryption at rest.
- Those second-level sub-questions (30 days, deletion contact, encryption) **have no further branching** of their own, whichever Yes/No is picked.
- The form was **never submitted** during testing (the Submit button was never clicked).
- Some text next to a select (e.g. "If yes, please explain how in the text box below...") is a **static note** pointing at a field already visible further down, and does not reveal any new hidden field.

---
*Document produced from manual testing of the Discord Developer Portal form, 2026-07-27.*
