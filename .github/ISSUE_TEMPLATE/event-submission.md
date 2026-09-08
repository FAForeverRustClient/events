---
name: Add an event
about: Something happening in FAF that belongs on the client's calendar.
title: "Event: "
labels: []
---

<!--
The easy way: the FAForever client's Events tab has a "Suggest an event"
button. Fill that form in and it opens this issue with the block below already
filled in, converted to UTC, and correctly spelled. Then you press submit and
you are done.

By hand is fine too. Fill in the block below and delete what does not apply.
Everything except "title" and "startsAt" is optional.

  startsAt   UTC. Either a timestamp (2026-10-04T18:00:00Z) or a bare date
             (2026-10-04) for something that is a day rather than a moment.
             The client shows it in each reader's own time zone.
  category   patch | tournament | meetup | cgn | ladderPool | other
  origin     official (FAF itself) | community (anybody else)
  recurrence {"weekly": {"interval": 1}} for weekly, 2 for fortnightly,
             or "monthly" for the same day every month. Leave it out for a
             one-off.
  links      plain https addresses only.

Tournaments and released patches do NOT belong here: the client already reads
those from FAF's own services and shows them on the calendar by itself.
-->

```json
{
  "title": "",
  "summary": "",
  "category": "meetup",
  "origin": "community",
  "host": "",
  "startsAt": "",
  "endsAt": "",
  "links": [{ "label": "", "url": "" }]
}
```

<!--
A check runs on this issue and comments whether the block reads cleanly. A
maintainer then adds the "approved" label, which commits it and closes this.
-->
