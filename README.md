# The FAForever community calendar

One document, `calendar.json`, read by the Events tab of the
[FAForever Rust client](https://github.com/FAForeverRustClient/FAForeverRustClient).
Adding an event here is a commit, not a client release: every client picks it up
on its next visit to the tab.

**Format:** [`docs/events-catalogue.md`](https://github.com/FAForeverRustClient/FAForeverRustClient/blob/develop/docs/events-catalogue.md)
in the client repository is the contract. Read it before editing by hand.

---

## What belongs in here

Only what a person knows and no service does: a club's game night, a CGN date, a
meetup, a patch date somebody has been told.

**Not** tournaments and **not** released patches. The client already has both:
tournaments come from FAF's tournament service and patches from the changelog
index, and they appear on the calendar on their own. Adding them here would
show everybody two of each.

## Adding an event by hand

Open an issue with the event form, or send a pull request against
`calendar.json` directly. The smallest useful entry is three fields:

```json
{
  "id": "cgn-43",
  "title": "Community Game Night 43",
  "startsAt": "2026-10-04T18:00:00Z"
}
```

Write times in **UTC**. The client converts them to each reader's own zone. A
bare date (`2026-10-04`) means a whole day and is never converted, which is what
a patch release wants.

## The Discord bot

`sources.json` lists the servers whose **guild scheduled events** are mirrored
into `calendar.json` automatically. That is the structured metadata Discord
already holds for an event: a name, a start, an end and a recurrence rule. The
bot reads it and writes catalogue entries, so a game night that is a recurring
event on its own server needs nobody to keep a date up to date in two places.

It reads scheduled events and nothing else. No message content, so no privileged
intent, and nothing the bot sees is anybody's conversation.

### Enabling a server

1. **Invite the bot.** Discord developer portal, OAuth2 URL generator, scope
   `bot`, no permissions ticked. Reading a guild's scheduled events needs
   `View Channels`, which the default role already has; **Manage Events** is a
   write permission and is deliberately not requested.
2. **Add the token.** Repository settings, Secrets and variables, Actions, a new
   secret named `DISCORD_BOT_TOKEN`. Only somebody with write access here can do
   this, and nobody else can read it back.
3. **Fill in the server.** Its guild id and invite in `sources.json`, and
   `enabled` set to `true`.

Until step 2 and step 3 are both done the workflow runs and does nothing,
deliberately: a red run every six hours would train everybody to ignore it.

### What it does with a recurrence rule

The catalogue understands "every N weeks" and "the same day every month". A
Discord rule that says anything else (daily, yearly, several weekdays at once,
every other month) is published as its **next occurrence with no rule**, which
is visibly one entry rather than invisibly the wrong series. Split such an event
into one per weekday on Discord if it should show as a series.

### Where the bot lives

`scripts/events-bot.mjs` in the client repository, with its tests, because that
is where the calendar format is defined and where CI reviews changes to it. The
workflow here checks that repository out to run it, so there is one copy of the
mapping rather than two that drift.

## Guardrails

`validate.yml` parses `calendar.json` on every push and pull request and refuses
an entry without a title or a start, or a link that is not plain `https`. The
client reads this document leniently (one bad entry is dropped rather than
emptying the tab), so a mistake here is quiet: the check is what makes it loud.
