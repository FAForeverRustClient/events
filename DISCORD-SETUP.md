# Putting a Discord server's events on the FAF calendar

Written to be handed over. It is in three parts because three different people
can be involved, and none of them needs to do the others' part:

- **Part A** is for whoever can create a Discord application. Anybody can; it
  takes five minutes and does not need any FAF or Dojo permission.
- **Part B** is for whoever runs the Discord server (the Dojo, say). It is one
  click on a link, and it needs **Manage Server** on that server.
- **Part C** is for whoever has write access to this repository.

The order that works best is A, then B and C in either order. Nothing happens
until all three are done, and nothing breaks in the meantime: the workflow runs
every six hours, finds no token or no enabled server, says so, and exits.

---

## Part A: create the bot (anybody, once)

This produces two things: an **invite link** to send to Part B, and a **token**
to send to Part C. One bot serves every server, so this is done once and never
again.

1. Open <https://discord.com/developers/applications> and sign in.
2. **New Application**. Name it something recognisable, for example
   `FAF Calendar`. Accept the terms and create it.
3. Left sidebar, **Bot**.
   - Leave every **Privileged Gateway Intent** switched **off**. The bot reads
     scheduled events, which are metadata, not messages. If you are ever asked
     to enable Message Content for this, something is wrong.
   - **Reset Token**, then copy the token. It is shown once. Send it to Part C
     over something private (a Discord DM to that person is fine; a public
     channel is not). Do not put it in this repository, an issue, or a pull
     request.
4. Left sidebar, **OAuth2** → **URL Generator**.
   - **Scopes:** tick `bot` and nothing else.
   - **Bot Permissions:** tick nothing. Reading scheduled events needs only
     `View Channels`, which every server's default role already grants.
     **Manage Events** is a *write* permission and is deliberately not asked
     for: this bot cannot create, edit or cancel anything.
   - Copy the generated URL at the bottom. That is the invite link for Part B.
5. Optional, and worth it: **Installation** → set **Install Link** to `None`, so
   nobody adds the bot to a server by accident.

Send onwards: the **invite link** to Part B, the **token** to Part C.

---

## Part B: let the bot into the server (the server owner)

You need **Manage Server** on the Discord server. This takes about a minute.

1. Open the invite link from Part A in a browser where you are signed in to
   Discord.
2. Pick the server from the dropdown, and **Authorise**. The permission list
   should be empty. If it asks for anything more than "Add to server", stop and
   ask the person who made the link.
3. **Copy the server id.** Discord: User Settings → Advanced → turn on
   **Developer Mode**. Then right-click the server's icon in the server list →
   **Copy Server ID**. It is an 18 or 19 digit number.
4. **Copy a server invite** you are happy to have in a desktop client, for
   example `https://discord.gg/…`. Prefer a permanent invite with no expiry: it
   is shown on every calendar entry from this server, as the way in for
   somebody who is not a member yet and therefore cannot open the event link.
5. **Check the events themselves.** The bot publishes what Discord's *Events*
   tab holds, so anything not in there will not appear on the calendar:
   - Server dropdown → **Events** → **Create Event**.
   - For a recurring night, set **Repeat** to weekly on the one weekday it
     happens. A rule naming several weekdays at once is published as its next
     occurrence only, because the calendar's weekly rule repeats on one day.
   - Fill in **Start Time** and **End Time**. The end is what tells a reader
     whether it is worth joining at ten.
   - The **Description** becomes the line under the title on the calendar. Two
     sentences is plenty; it is shown as plain text.
   - The bot ignores events that are **completed** or **cancelled**.

Send onwards: the **server id** and the **invite** to Part C.

---

## Part C: switch it on (write access to this repository)

1. **Add the token.** On **this** repository, `FAForeverRustClient/events`:
   <https://github.com/FAForeverRustClient/events/settings/secrets/actions> →
   **New repository secret**.
   - Name: `DISCORD_BOT_TOKEN`, spelled exactly that way.
   - Value: the token from Part A.

   Both halves of that matter, and getting either wrong is silent. A secret on
   the **client** repository is not visible to a workflow in this one: they do
   not share secrets, and nothing warns you. A secret under **another name** is
   not read either. `gh secret list --repo FAForeverRustClient/events` prints
   the names it can actually see, which is the quickest way to check.

   GitHub will not show the value again, and it is not readable by anyone who
   can only read this repository. If it ever ends up somewhere it should not,
   Part A can press **Reset Token** and the old one dies immediately.
2. **Add the server** to [`sources.json`](sources.json):

   ```json
   {
     "guildId": "1230169748889669682",
     "enabled": true,
     "host": "FAF Dojo",
     "origin": "community",
     "category": "meetup",
     "invite": "https://discord.gg/example",
     "rules": [
       { "match": "tournament", "category": "tournament" },
       { "match": "cup", "category": "tournament" },
       { "match": "cgn", "category": "cgn" }
     ]
   }
   ```

   - `host` is printed on the entry as who is running it.
   - `category` is what an event from this server is filed as by default, and
     `rules` override it when the event's name contains the text: the example
     files anything with "tournament" or "cup" in its name as a tournament.
   - `origin` is `community` for anybody who is not FAF itself.
3. **Run it once by hand.** Actions tab → **Mirror Discord scheduled events** →
   **Run workflow**. It finishes in under a minute and its log says how many
   events it found per server. If anything changed it commits `calendar.json`.
4. **Look at the result.** Open `calendar.json`: entries from the bot have ids
   beginning `discord-`. Then open the client's Events tab, where they appear on
   the day and at the time the reader's own clock says.

From then on it runs by itself, four times a day.

---

## When something is not there

**The run is green and the summary says "The mirror did nothing".** The token
is not where the workflow looks. Three ways to get there:

- It was added to the **client** repository rather than to this one. Secrets do
  not cross repositories, and this is the easy mistake to make because both
  repositories are involved in the job.
- It was added under **another name**. The workflow reads `DISCORD_BOT_TOKEN`
  and nothing else.
- It was added as an **environment** or **Dependabot** secret. It has to be a
  repository secret under **Actions**.

`gh secret list --repo FAForeverRustClient/events` settles all three: it prints
the names this workflow can see. An empty list means Part C step 1 has not
landed here.

A run started by hand now fails rather than passing quietly, so this shows up
as a red cross the moment you press the button. The cron still warns and stays
green on purpose.

**It says "No enabled guilds".** Part C step 2 has not been done, or `enabled`
is still `false`, or `guildId` is empty.

**It fails with "Discord answered 401".** The token is wrong, or it was reset in
the developer portal after being added here. Reset it once more and update the
secret.

**It fails with "Discord answered 403" or "404" for a server.** The bot is not
in that server (Part B), or the server id is wrong. A 404 on the
scheduled-events endpoint is what Discord answers for a guild it cannot see you
in, so it usually means the same thing as a 403.

**It succeeds and finds 0 events.** There are no scheduled events on that
server's Events tab, or all of them are completed or cancelled. Part B step 5.

**An event is on Discord but the wrong kind on the calendar.** Its name did not
match any `rules` entry, so it took the server's default `category`. Add a rule.

**A weekly event shows as a single date.** Its Discord rule names more than one
weekday, or repeats daily, or every other month. The calendar can express "every
N weeks" and "the same day each month" and nothing else, so rather than inventing
a series the bot publishes the next occurrence. Split it into one Discord event
per weekday if it should show as a series.
