# NestaduPM

Auto-captures tagged requests from Slack into an always-visible checklist.

Requests get posted in Slack, nobody owns them, they scroll away. This watches our channels, notices when someone is @-tagged with an actual ask, and puts one line on a Slack List pinned to the top of the channel.

| Task | Assignee | Original post | Done |
|---|---|---|---|
| Order rebar for Lot 7 | Mike | *link back to the message* | ☐ |

---

## John — this one's for you

**It needs to run on your setup**, because it requires an instance that's always open. That's the main reason it's landing with you rather than sitting on my laptop.

Three things before it can go live:

1. **Step 1 is already done** — I've created the Slack app. You may just need to approve it, or reinstall it if the permissions changed.
2. **An Anthropic API key** from https://console.anthropic.com, with a few dollars of credit. Can be the same key you're already using for NestaduPM. This is what tells a real request apart from "thanks, looks good." Runs to cents per month at our volume.
3. **Somewhere to run it** — it's a scheduled job on an always-on machine. If that machine sleeps, the next run catches up, so nothing is lost.

**Setup instructions:** [docs/system-setup.md](docs/system-setup.md) — about 20 minutes, click by click. Step 7 is optional.

**Before you let it write anything,** run `python3 nestadu_pm.py watch --dry-run --verbose` over a few days of our real messages. It prints what it *would* add without touching Slack. You're checking that it catches the real asks and stays quiet on chatter.

Two things I'd want you to know going in: it adds tasks **automatically with no confirmation step**, so the odd wrong entry is expected and needs a weekly clear-out — and Slack **cannot pin anything to the top of a thread**, so the always-visible surface is the channel bookmark bar instead. Both are explained in the reference.

---

## Where everything is

| | |
|---|---|
| [docs/system-setup.md](docs/system-setup.md) | Step-by-step setup. Start here. |
| [docs/reference.md](docs/reference.md) | Full documentation — how it works, all commands, **security notes**, limitations, tuning |
| [docs/SKILL.md](docs/SKILL.md) | Optional Claude skill, so you can ask about the list in plain English |
| `nestadu_pm.py` | The script itself |
| `test_nestadu_pm.py` | `python3 test_nestadu_pm.py` — 40 tests, no tokens or network needed |

**Keep this repository private.** Nothing in it is secret, but it maps our internal setup.

---

Internal Nestadu tooling. Not licensed for outside use.
