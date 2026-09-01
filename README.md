# NestaduPM

Auto-captures tagged requests from Slack into an always-visible checklist.

In order to minimize missed tasks and to put everything into a mandatory to-do list, NestaduPM will be listening to messages that sound like instructions, ESPECIALLY if anyone gets tagged, and add those to a list that cannot be put away unless you mark it as complete. 


---

Setup instructions

**It needs to run on your setup**, because it requires an instance that's always open. Not a problem if the Studio where you run NestaduPM is always on anyway.

Three things before it can go live:

1. **Step 1 is already done** — I've created the Slack app. You may just need to approve it, or reinstall it if the permissions changed.
2. **An Anthropic API key** from https://console.anthropic.com, with a few dollars of credit. Can be the same key you're already using for NestaduPM. The readme suggests using Haiku, and we might get away with it, but maybe Sonnet would give us a bit more intelligence. 
3. **Somewhere to run it** — it's a scheduled job on an always-on machine. If that machine sleeps, the next run catches up, so nothing is lost. Ideally this would be running on the same machine where NestaduPM is also always running. 

**Setup instructions:** [docs/system-setup.md](docs/system-setup.md) — I already took care of Step 1. Step 7 is technically optional but I would recommend giving NestaduPM the skill so it works nicely with the app. 

---

## Where everything is

| | |
|---|---|
| [docs/system-setup.md](docs/system-setup.md) | Step-by-step setup.|
| [docs/reference.md](docs/reference.md) | Full documentation - mostly slop... read if you are really curious. |
| [docs/SKILL.md](docs/SKILL.md) | Optional Claude skill, so you can ask about the list in plain English |
| `nestadu_pm.py` | The script itself |
| `test_nestadu_pm.py` | `python3 test_nestadu_pm.py` — 40 tests, no tokens or network needed |
