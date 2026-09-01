# NestaduPM

Auto-captures tagged requests from Slack into an always-visible checklist.

Requests get posted in Slack, nobody owns them, they scroll away. This watches the channels, notices when someone is @-tagged with an actual ask, and puts one line on a Slack List pinned to the top of the channel.

---

This app is a prime candidate to be ran in your local setup, since it will require an instance to be always open.

These are the instructions to set it up in your system:
**[docs/system-setup.md](docs/system-setup.md)**.

I already took care of Step 1, but the rest require Claude API tokens, could be the same you're using for NestaduPM, and most importantly, they require to be in your machine.

Step 7 is optional — it installs a Claude skill so you can ask about the list conversationally instead of typing commands. Steps 1-6 are the whole product.

---

Two things worth reading before it goes live, both at the bottom of the setup guide: it adds tasks automatically with no confirmation step, so occasional wrong entries are expected and need a weekly clear-out; and Slack cannot pin anything to the top of a *thread*, so the always-visible surface is the channel bookmark bar.

Run `python3 test_nestadu_pm.py` to check the install — 40 tests, no network or tokens needed.
