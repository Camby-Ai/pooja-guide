# Pooja Setup Guide

Pooja is a builder. Most small teams lose a week between "the client said yes" and "it is live", because the person who sold the work is also the person who has to build it, usually late at night. Pooja takes the build: every client gets their own folder and a status file that is always current, she builds the automation, tests it before anyone sees it, and hands over a go live checklist when it is ready. If a build depends on signed terms or a change of scope, she stops and says so first. This guide sets her up on Hermes Agent, a free and open source agent from Nous Research that runs on your own machine.

The software is free. The model behind it is not always free, and the real work is writing down what "done" means for each client before she starts.

**Checked on 2026-11-21.** Commands and limits change. Sources are at the bottom.

**How this guide was made:** every command below was copied from the Hermes Agent documentation on that date. We have not test-run them on a fresh machine. If a step does not behave the way it says, see "If a step breaks" near the end: your AI can patch it for you in a minute.

## What Pooja does

- Gives every client their own build folder, so nothing from one client leaks into another.
- Keeps a status file in each folder that is always current: what is built, what is tested, what is blocked.
- Builds the automation from a short scope you write, and nothing outside it.
- Tests before anyone sees it, and writes down what she tested and what she could not.
- Writes a go live checklist when a build is ready, for whoever takes it live.
- Stops and tells you first when a build depends on signed terms or a change of scope.
- Writes one short status roll-up each weekday morning across every client.
- Does not go live, touch a client's real accounts, or send anything to a client. You do.

## What you need before you start

| You need | Why | Cost | Checked |
|---|---|---|---|
| A computer that stays on: macOS, Linux, Windows or WSL2 | The morning roll-up only runs while Hermes' gateway is running | Yours already | 2026-11-21 |
| Git and curl | The installer uses them | Free | 2026-11-21 |
| Hermes Agent | Runs Pooja | Free, MIT licence | 2026-11-21 |
| A model with at least 64,000 tokens of context | Hermes rejects smaller models at startup | Depends on the provider you pick. Not measured here | 2026-11-21 |
| A written scope for one client | She builds what is written, so it has to be written | Yours already | 2026-11-21 |
| Test accounts, not live ones | So a test never touches a real customer | Depends on the tools you build on | 2026-11-21 |
| Telegram (optional) | To get the status roll-up on your phone | Free | 2026-11-21 |

On a brand new install the setup wizard may offer a "Free, start now" choice that needs no account. The Hermes docs say it is shown only when it is available, so do not plan around it.

## The honest timeline

| When | What | Your time |
|---|---|---|
| Day 1 | Install Hermes and get one normal chat working | The Hermes docs say under 5 minutes. Allow 30 |
| Day 1 | Create Pooja and give her a builds folder | 20 minutes |
| Day 1 | Write the scope and the first status file for one client | 30 to 60 minutes. This is the part that decides everything |
| Day 1 to 2 | Turn on her file and terminal tools and run one build by hand | 1 to 2 hours |
| Day 2 | Turn on the morning status roll-up | 20 minutes |
| Week 1 | Read the roll-up, check her tests, take builds live yourself | 15 minutes a day |

The slow part is not the install. It is the scope. A builder with a vague brief builds the wrong thing quickly. Pooja is only as good as the page that says what the client is getting and how you will know it works.

Three limits to know before you start:

- She runs commands on your computer. Hermes asks you before a dangerous command, and that check is your safety net, so read what it shows you before you approve.
- Keys and passwords stay out of her folders. Give her test accounts. Live keys go in at go live, by you.
- In our own team she builds in Claude Code. This guide has her build with Hermes' own file and terminal tools, which do the same job for a small automation.

## Setup, in this order

### 1. Install Hermes and get one chat working

Do not add anything else until a plain conversation works. The Hermes quickstart is blunt about this, and it is right.

On macOS and Windows the Hermes docs recommend the Hermes Desktop installer from https://hermes-agent.nousresearch.com/. For a command line only install on macOS, Linux or WSL2:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Reload your shell, pick a model, and start a chat:

```bash
source ~/.zshrc
hermes setup
hermes
```

You should see: a welcome banner naming your model, and a reply to a simple question. If replies are empty or broken, run `hermes model` again and confirm the provider, the model and the login. If anything else looks off, run `hermes doctor`.

### 2. Create Pooja

Every agent in Hermes gets its own profile: its own settings, memory and identity. Never point two agents at the same profile.

```bash
hermes profile create pooja --description "Builder. One folder and one status file per client. Builds client automations from a written scope, tests before anyone sees them, writes a go live checklist, and stops when a build depends on signed terms or a scope change. Never goes live. The owner does."
```

Creating the profile also creates a `pooja` command. Check it:

```bash
pooja chat
```

You should see: the same kind of chat as step 1, now running as the `pooja` profile.

### 3. Give Pooja her identity

In Hermes, who an agent is lives in a file called `SOUL.md` inside that agent's home. For a profile named `pooja` that is:

```text
~/.hermes/profiles/pooja/SOUL.md
```

Hermes creates a starter file there. Replace its contents with the text in "Pooja's instructions" below. Keep it about who she is and how she talks. Your rules and your clients go in different files, in the next step.

You should see: after starting a new `pooja chat`, she introduces herself as your builder when asked who she is.

### 4. Build her desk: the rules, the status file, the checklist

This is the step everybody rushes, and it is the one that makes Pooja useful.

Make a folder for her to work from, with one folder inside it for a template client:

```bash
mkdir -p ~/pooja-builds/clients/_template/build ~/pooja-builds/clients/_template/tests
```

Put three files in it. Hermes loads `AGENTS.md` from the folder you start a chat in, and Pooja copies the other two for every new client.

`~/pooja-builds/AGENTS.md`, the rules:

```text
# How Pooja works these builds

- clients/ holds one folder per client. To start a client, copy clients/_template and rename it. Never put two clients in one folder.
- Each client folder has SCOPE.md (what they are getting, written by me), STATUS.md (kept current by you), build/ and tests/.
- Build only what SCOPE.md says. If something is not in it, stop and ask me. Do not guess what the client meant.
- Update STATUS.md every time you finish a piece of work, before you tell me about it.
- Test every build before you call it ready. Write what you tested, what passed, and what you could not test, in STATUS.md.
- Use test accounts and made-up data only. Never use a client's live account, live customers or real money in a test.
- If a build depends on signed terms, a price, or a change to the scope, stop. Write "Blocked" and the reason in STATUS.md and tell me first.
- When a build is ready, copy GO_LIVE.md into the client folder and fill it in for whoever takes it live.
- Never go live, never send anything to a client, and never store a password, key or token in these folders.
```

`~/pooja-builds/clients/_template/STATUS.md`, the status file:

```text
# STATUS: {client name}

- Stage: {scoping, building, testing, ready, blocked, live}
- Last updated: {date and time}
- Built so far: {what exists in build/}
- Tested: {what you ran, and the result}
- Not tested yet: {what and why}
- Blocked on: {signed terms, scope change, an account, or "nothing"}
- Next step: {the one next thing, and who does it}
```

`~/pooja-builds/clients/_template/GO_LIVE.md`, the go live checklist:

```text
# Go live checklist: {client name}

- [ ] Terms are signed and the scope matches what was built
- [ ] Every test in STATUS.md passes
- [ ] Test accounts and made-up data are swapped for the client's live accounts, by a person
- [ ] The client knows the go live day and who to call
- [ ] Someone watches the first real run
- [ ] How to switch it off is written here: {steps}
- [ ] STATUS.md says "live", with the date
```

You should see: when you run `pooja chat` from inside `~/pooja-builds` and ask how she starts a new client, she answers from `AGENTS.md` and names the three files.

### 5. Turn on her file and terminal tools

Pooja builds by writing files and running commands. Open the tools menu for her profile and confirm the file and terminal toolsets are on:

```bash
hermes -p pooja tools
```

Start a new `pooja chat` after changing her tools. An old conversation keeps its old permissions.

Hermes checks every command against a list of dangerous patterns and asks you before it runs one. Leave that on. It is the difference between a builder and an accident.

You should see: in a new chat from inside `~/pooja-builds`, she can create a file in a client folder and read it back to you.

### 6. Schedule the morning status roll-up

Hermes has a built-in scheduler. Give the job Pooja's folder so she reads every client's status when she runs. The folder must be written as a full path.

```bash
hermes -p pooja cron create "weekdays at 7am" \
  "Status roll-up. Read STATUS.md in every folder under clients/ except _template. For each client give one line: stage, what changed since yesterday, and what is blocked. Put anything blocked at the top with the reason. Build nothing and change nothing. Keep it under 150 words." \
  --workdir /Users/you/pooja-builds \
  --name "Build status"
```

Schedules run from a gateway, and each profile has its own. Start Pooja's, and check that her scheduler is alive:

```bash
pooja gateway start
hermes -p pooja cron status
```

`pooja gateway install` keeps it running after a restart.

By default the roll-up is saved on your machine. To get it on your phone, connect Telegram first with `hermes gateway setup`, then create the job with `--deliver telegram` added.

One thing to check on your machine: whether your times come out in your own timezone. Hermes reads the schedule in the profile's `timezone:` setting. Add `--timezone America/Los_Angeles` (or your zone) to pin it.

You should see: a status roll-up waiting for you the next weekday morning.

### 7. Run your first build, by hand

Copy `clients/_template` to a new folder for one real client and write `SCOPE.md` in it yourself: what they are getting, in plain words, and how you will know it works. Then from inside `~/pooja-builds` run `pooja chat` and send her this:

```text
Read clients/{client folder}/SCOPE.md. Tell me in five lines what you are going to build and how you will test it, and list anything in the scope that is unclear. Wait for my OK. Then build it in build/, write the tests in tests/, run them with made-up data only, and update STATUS.md. If anything depends on signed terms or a change of scope, stop and tell me first. Do not go live.
```

You should see: a short plan and her questions first. After your OK, files in `build/` and `tests/`, a `STATUS.md` that says what passed and what was not tested, and nothing live.

## Pooja's instructions

Paste this into `~/.hermes/profiles/pooja/SOUL.md`. Change the parts in braces.

```text
# Who you are

You are Pooja. You are the builder for {your business}.

Your job is to turn a signed scope into a working automation that is ready to go live: every client has their own folder, every folder has a status file that is always current, every build is tested before anyone sees it, and nothing ships that was not in the scope.

# How you work

- One client, one folder. Nothing from one client is ever copied into another.
- You build what the scope says. When it is unclear, you ask before you build.
- You keep the status file current. If it is not in STATUS.md, it did not happen.
- You test before you say ready. You say what you tested, what passed, and what you could not test.
- You test with made-up data and test accounts only.
- You stop when a build depends on signed terms, a price, or a change of scope, and you say so first.
- You hand over with a go live checklist. You never go live yourself.
- You never store a password, key or token in a build folder.

# How you talk

- Calm, precise and warm. A steady engineer, never rushed.
- Lead with the status: ready, building, testing or blocked.
- One line per client: stage, what changed, what is next.
- When you ask a question, ask exactly one.
```

This persona was written from Pooja's job description for this guide. It is not a copy of the instructions she runs on in our own team.

## First test

Run `pooja chat` from inside `~/pooja-builds` and send her this:

```text
Give me the status of every client under clients/, one line each, from their STATUS.md files only. Then tell me which builds are blocked and on what, and which ones have something not tested yet. If a folder has no STATUS.md, say so. Build nothing.
```

A good answer lists only client folders that exist, takes every word from each `STATUS.md`, names what is blocked and what is untested, and tells you plainly when a file is missing instead of filling it in.

If she says a build is tested when `STATUS.md` does not say so, tighten the testing line in `AGENTS.md` and ask her to run the tests again.

## When it goes wrong

| What you see | Why | Fix |
|---|---|---|
| Empty or broken replies | Model or login is wrong | `hermes model`, then a plain chat before anything else |
| She does not know your clients | The chat was not started in `~/pooja-builds` | Start `pooja chat` from that folder |
| She cannot create or run anything | The file or terminal toolset is off, or the chat is older than the change | `hermes -p pooja tools`, then a new chat |
| She built something that was not asked for | `SCOPE.md` is vague | Rewrite the scope, and keep the "ask before you build" rule in `AGENTS.md` |
| "Ready" but it does not work | The tests did not cover the real path | Add the missing case to `tests/`, and make her list what is not tested |
| Hermes keeps asking before commands | That is the dangerous command check doing its job | Read the command, then approve it or say no |
| No morning roll-up | Pooja's gateway is not running, or the job is missing its folder | `hermes -p pooja cron status`, `pooja gateway start`, and recreate the job with `--workdir` |
| The roll-up arrives at the wrong hour | The schedule is running in another timezone | Recreate the job with `--timezone` set to your zone |
| Anything else | | `hermes doctor` |

## If a step breaks

Hermes changes fast, so a command here can drift from what your version expects. You do not need us to fix it. Paste this into Hermes itself, or into Claude or ChatGPT, with the error underneath:

```text
I am following a setup guide for a builder agent on Hermes Agent. This step failed. Here is the step, the exact command I ran, and the full error. Check the current Hermes Agent docs at https://hermes-agent.nousresearch.com/docs, tell me what changed, and give me the corrected command.
```

`hermes doctor` is worth running first. It catches most setup problems on its own.

## What it costs to run

- Hermes Agent: free, MIT licence.
- The model: billed by whichever provider you choose in `hermes model`. Building and testing use far more of the model than a morning roll-up does. Not measured here, so no number is given.
- The tools you build on (a CRM, a texting service, a calendar) bill separately. Check each one's current price before you promise a client anything.

## Want it set up for you?

CAMBY Ai sets Pooja up and runs her for you: https://mikecamby.com

## Credit

Pooja runs on [Hermes Agent](https://github.com/NousResearch/hermes-agent), built by Nous Research and released under the MIT licence. Hermes is not ours. This guide is a walkthrough for using it to build and test client automations. Not affiliated with or endorsed by Nous Research, Anthropic or Telegram.

## Sources

All read on 2026-11-21 from the Hermes Agent repository and its documentation.

- https://github.com/NousResearch/hermes-agent : install command, first-run commands, licence.
- https://hermes-agent.nousresearch.com/docs/getting-started/quickstart : install paths, `hermes setup`, the 64,000 token minimum, the "Free, start now" option, `hermes doctor`.
- https://hermes-agent.nousresearch.com/docs/user-guide/profiles : `hermes profile create`, `--description`, one profile per agent, the per-profile command.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/personality : `SOUL.md`, where it lives and what belongs in it.
- https://hermes-agent.nousresearch.com/docs/user-guide/which-file-does-what : `SOUL.md` versus `AGENTS.md`.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/tools : the file and terminal toolsets, turning toolsets on and off.
- https://hermes-agent.nousresearch.com/docs/user-guide/security : the dangerous command check and how approvals work.
- https://hermes-agent.nousresearch.com/docs/user-guide/features/cron : `cron create`, "weekdays at", `--workdir`, `--deliver`, `--name`, `--timezone`.
- https://hermes-agent.nousresearch.com/docs/user-guide/messaging : `hermes gateway setup`, `start`, `install`, `status`.
- https://hermes-agent.nousresearch.com/docs/user-guide/multi-profile-gateways : one gateway per profile, `<profile> gateway start`, `hermes -p <profile> cron status`.
