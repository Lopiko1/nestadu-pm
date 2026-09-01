# NestaduPM — system setup

One-time setup, about 20 minutes. You do not need to know how to code.

**How it works:** a script runs every few minutes, reads your channels, and looks for messages where someone is @-tagged. For each one it reads the 2 messages before and 2 after, asks an AI whether that's a real request, and if so adds one line to a Slack List:

| Task | Assignee | Original post |
|---|---|---|
| Order rebar for Lot 7 | Mike | *link back to the Slack message* |

The list is bookmarked to the top of the channel so it's always visible. Check items off when they're done.

**Secrets warning:** you'll create two keys. Treat them like passwords. They go into environment variables on the machine that runs the script — never into Slack, email, or a chat window, including this one.

---

## One correction, up front

You asked to pin the checklist to the top of each **thread**. Slack cannot do that. Bookmarks and pins are both channel-level in Slack's API (`bookmarks.add` requires a `channel_id`; there is no thread equivalent). Nobody can build it — not me, not a developer.

What you get instead is the **channel bookmark bar**: a `📋 OPEN ITEMS` button at the top of the channel, visible on every screen, desktop and mobile, that never scrolls away. That's the real version of what you asked for.

---

## Step 1 — Create the Slack app

1. Go to **https://api.slack.com/apps** → **Create New App** → **From an app manifest**.
2. Pick your Nestadu workspace → **Next**.
3. Delete what's in the box and paste this (**YAML** tab):

```yaml
display_information:
  name: NestaduPM
  description: Auto-captures tagged requests into an always-visible Slack list
  background_color: "#1b3a2f"
features:
  bot_user:
    display_name: NestaduPM
    always_online: true
oauth_config:
  scopes:
    bot:
      - bookmarks:read
      - bookmarks:write
      - channels:read
      - channels:history
      - groups:read
      - groups:history
      - chat:write
      - lists:read
      - lists:write
      - pins:write
      - users:read
      - im:write
settings:
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false
```

4. **Next** → **Create**.
5. Left sidebar → **OAuth & Permissions** → **Install to Workspace** → **Allow**.
6. Copy the **Bot User OAuth Token** (starts with `xoxb-`).

If you see *"requires approval from an admin"*, you're not a workspace admin — send your admin the manifest above and ask them to approve. Normal, just adds a day.

---

## Step 2 — Get an Anthropic API key

This is what tells a real request apart from "thanks, looks good."

1. Go to **https://console.anthropic.com** → sign up → **API Keys** → **Create Key**.
2. Copy it (starts with `sk-ant-`).
3. Add a small amount of credit. At your volume this costs roughly **a few cents a month** — the model used is Haiku, the cheapest one, and each check is a handful of short messages.

---

## Step 3 — Put both keys on your machine

**Windows** (Command Prompt):

```
setx SLACK_TOKEN "xoxb-paste-here"
setx ANTHROPIC_API_KEY "sk-ant-paste-here"
```

Close the window and open a new one — `setx` only affects new windows.

**Mac / Linux:**

```bash
echo 'export SLACK_TOKEN="xoxb-paste-here"' >> ~/.zshrc
echo 'export ANTHROPIC_API_KEY="sk-ant-paste-here"' >> ~/.zshrc
source ~/.zshrc
```

Check both work:

```
python3 nestadu_pm.py doctor
```

You should see your workspace name and `Classifier : OK`.

---

## Step 4 — Set up a channel

Get the channel ID: right-click the channel → **View channel details** → bottom of the panel. Looks like `C0123456789`.

Invite the app (it can't read a channel it isn't in):

```
/invite @NestaduPM
```

Then:

```
python3 nestadu_pm.py setup --channel C0123456789
```

That creates the list, gives the channel access, adds the **📋 OPEN ITEMS** bookmark, and posts a pinned announcement.

> The list has **Task**, **Assignee**, **Original post**, and a **Done** checkbox.
>
> Slack's todo mode also adds a **Due Date** column that the script never fills. Leave it empty — it's harmless. If it bothers you, Slack lets you hide fields from a list view (look for a hide/field option on the column header or in the view settings; the exact wording moves around between Slack releases). Todo mode is worth keeping regardless, because it's what gives everyone the "Assigned to you" view on Slack's Lists page.

---

## Step 5 — Watch it work before you trust it

**Always dry-run first.** This reads your last 24 hours and prints what it *would* add, writing nothing:

```
python3 nestadu_pm.py watch --dry-run --verbose
```

Output looks like:

```
Scanning #field-ops since 2026-08-31 09:00 UTC ...
  skip  Mike           thanks that looks great
  ADD   Mike           Order rebar for Lot 7            https://nestadu.slack.com/...
  ADD   Sara           Call city about permit           https://nestadu.slack.com/...

3 mention(s) examined · would add 2 · skipped 1
```

Run this a few times across a normal week of your real messages. You're checking two things: does it catch the real asks, and does it stay quiet on chatter. When you're happy, drop `--dry-run`:

```
python3 nestadu_pm.py watch
```

---

## Step 6 — Run it automatically

**Mac / Linux** — `crontab -e`:

```
*/5 * * * *  cd /path/to/script && /usr/bin/python3 nestadu_pm.py watch
0 8 * * 1    cd /path/to/script && /usr/bin/python3 nestadu_pm.py digest --channel C0123456789
0 9 * * 1-5  cd /path/to/script && /usr/bin/python3 nestadu_pm.py nag --stale-days 5
```

Every 5 minutes it catches new requests; Monday 8am posts a digest; weekday 9am DMs anyone sitting on something 5+ days old.

**Windows** — Task Scheduler → Create Basic Task → Daily, repeat every 5 minutes → Action *Start a program* → `python`, arguments `nestadu_pm.py watch`, "Start in" set to the script's folder.

The script only needs the machine awake when it runs. If the laptop sleeps overnight it catches up on the next run — nothing is lost, because it remembers where it stopped.

---

## Step 7 — Install the Claude skill (optional)

This step is only for people who want to talk to the system through Claude — ask
"who's overdue in #field-ops?" or "add that to the list" instead of typing
commands. **The system runs fine without it.** Steps 1-6 are the whole product;
this is a convenience layer.

The skill lives at [SKILL.md](SKILL.md) in this folder. It is not installed by
cloning the repo — skills are saved per Claude account.

To install it: open Cowork, point Claude at `docs/SKILL.md` from your clone, and
ask it to save that as a skill named `nestadu-pm`. Claude saves it to your
account, and it persists across sessions.

Each person who wants it installs it separately. It is not shared automatically
by the repo.

> **Keep it in sync.** If you change the script's commands or flags, update
> `docs/SKILL.md` in the same commit and re-save the skill. Otherwise Claude will
> confidently give people instructions for a version of the script that no longer
> exists — a stale skill fails quietly, which is worse than failing loudly.

---

## How the detection actually works

- Finds any message containing an `@mention`.
- Reads that message plus **2 before and 2 after** — exactly what you asked for.
- **Waits 10 minutes** before judging a fresh mention, unless 2 more messages already landed. This is deliberate: "hey @mike" followed 30 seconds later by the actual ask would otherwise be judged on the greeting alone.
- Sends the 5-message window to Claude Haiku, which returns yes/no plus a 4-5 word summary.
- On yes: adds one row, assigned to the tagged person, linked to the original post.
- Remembers every mention it has already judged, so nothing is ever added twice.

Ignored automatically: bot messages, join/leave notices, self-mentions, and tagging the NestaduPM bot itself.

---

## Commands

| Command | Does |
|---|---|
| `doctor` | Checks both keys, tests the classifier, shows configured channels |
| `setup --channel C...` | Creates list + bookmark + announcement |
| `watch [--dry-run] [--verbose]` | Scans and adds. **Always dry-run first** |
| `add --channel C... --task "..." --assignee U...` | Add one by hand |
| `list --channel C...` | Print open items |
| `digest --channel C...` | Post a summary grouped by person |
| `nag --stale-days 5` | DM people sitting on old items |

Everything takes `--help`. `watch`, `digest` and `nag` all take `--dry-run`. The script never deletes anything.

---

## When something breaks

| Error | Fix |
|---|---|
| `not_in_channel` | `/invite @NestaduPM` |
| `missing_scope` | Add it in OAuth & Permissions, then **reinstall the app** — adding a scope alone does nothing |
| `channel_not_found` | Use the `C...` ID, not `#name` |
| `lists_disabled_user_team` | Slack Lists is off or not on your plan |
| `invalid_auth` | Slack token wrong or revoked |
| `Anthropic API error 401` | API key wrong |
| `Anthropic API error 400 credit balance` | Add credit at console.anthropic.com |

If `watch` adds something wrong, just delete the row in Slack. It won't come back — the script remembers it already judged that message.

---

## Two things to watch for

**Auto-add means mistakes reach the list.** You chose no confirmation step, and AI classification is what's holding the quality line. It's good, not perfect. Budget for a weekly 30-second pass deleting anything odd. If you find yourself deleting more than one or two a week, tell me and we'll tighten the prompt or add a one-click confirm.

**Auto-capture solves capture, not commitment.** The old rule was "whoever asks, adds it" — the act of adding was a small act of ownership. Now a bot does it, which means people can end up with tasks they never consciously accepted. Watch for someone quietly accumulating 15 rows they've never looked at. The Monday digest exists partly to make that visible.
