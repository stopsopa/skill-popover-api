# How to publish a skill to skills.sh

Example: https://www.skills.sh/stopsopa/skill-popover-api

## Short version

There is **no "publish" command, no account and no form** (see the [official FAQ](https://www.skills.sh/docs/faq)).

1. Put the skill in a **public GitHub repo** (ours: [stopsopa/skill-popover-api](https://github.com/stopsopa/skill-popover-api)).
2. Install it once with `npx skills add <owner>/<repo>`.
3. That install sends anonymous telemetry, and skills.sh lists the skill automatically.

## Step by step

### 1. Create the skill

In your repo create `skills/<skill-name>/SKILL.md` (or let the CLI make a template: [`npx skills init <skill-name>`](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)). Full authoring guide: [creating-skills.md](https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md).

This repo ([browse on GitHub](https://github.com/stopsopa/skill-popover-api/tree/main/skills/popover-api)):

```
skills/
  popover-api/
    SKILL.md
    assets/popover.css
```

Minimum content of `SKILL.md` (real one: [SKILL.md](https://github.com/stopsopa/skill-popover-api/blob/main/skills/popover-api/SKILL.md)):

```md
---
name: popover-api
description: Use this skill every time one desire to place one element (tooltip, menu, popover, modal etc) in relation to some other element on the page.
---

...instructions for the agent...
```

Rules (spec: [agentskills.io/specification](https://agentskills.io/specification)):

- `name` and `description` are required.
- `name`: lowercase, hyphens allowed, same as the folder name.
- `description`: say clearly **when** the skill should be used - the agent decides based on this line.

### 2. Push to a public GitHub repo

```sh
git add skills
git commit -m "add popover-api skill"
git push
```

The repo must be public. Private repos are not sent to skills.sh.

### 3. Check the CLI can see it

```sh
npx skills add stopsopa/skill-popover-api --list
```

You should see `popover-api` on the list. If not, the folder layout or frontmatter is wrong.

### 4. Install it once (this is the "publish")

In any project:

```sh
npx skills add stopsopa/skill-popover-api
```

Make sure [telemetry](https://www.skills.sh/docs/cli) is not turned off (`DISABLE_TELEMETRY` and `DO_NOT_TRACK` must not be set), otherwise nothing is reported.

### 5. Find it on skills.sh

- Repo page: https://www.skills.sh/stopsopa/skill-popover-api
- Skill page: https://www.skills.sh/stopsopa/skill-popover-api/popover-api
- Search: https://www.skills.sh/search?q=popover-api

The address is always `skills.sh/<github-owner>/<repo>/<skill-name>`.

It may take a moment before it appears.

## How it works under the hood

- skills.sh stores no files. It only knows "repo X has skill Y, installed N times" from CLI telemetry.
- The telemetry is anonymous: skill name, skill files, timestamp. No personal data.
- Every later install by anybody adds to the install counter, which decides the position on the [leaderboard](https://www.skills.sh).
- When somebody installs your skill, the files come straight from [GitHub](https://github.com/stopsopa/skill-popover-api), not from skills.sh.

## Good to add to your README

```sh
npx skills add stopsopa/skill-popover-api
```

See how this repo does it: [README](https://github.com/stopsopa/skill-popover-api#readme).

Another walkthrough: [publish-skill by johnlindquist](https://github.com/johnlindquist/skill-publish-skill/blob/main/skills/publish-skill/SKILL.md).

More: [HOW_DOES_IT_WORK.md](HOW_DOES_IT_WORK.md), releasing changes: [UPDATE.md](UPDATE.md)
