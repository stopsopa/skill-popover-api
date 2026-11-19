# How to publish multiple skills in one GitHub repo

Example repo: https://www.skills.sh/stopsopa/skill-popover-api (today it has one skill: `popover-api`).
Below we pretend we add a second one, `tooltip-api`.

Read [PUBLISH.md](PUBLISH.md) first - publishing many skills works the same way, there is just more than one folder.

## Short version

1. One folder per skill inside `skills/`, each with its own `SKILL.md`.
2. Every skill needs a **different `name:`**.
3. **No `SKILL.md` in the repo root.**
4. Push, then install each skill once - each one gets its own page on [skills.sh](https://www.skills.sh).

## 1. Folder layout

```
skills/
  popover-api/
    SKILL.md
    assets/popover.css
  tooltip-api/
    SKILL.md
```

Create the second one by hand or with the CLI ([`init`](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)):

```sh
cd skills
npx skills init tooltip-api
```

Each `SKILL.md` has its own header:

```md
---
name: tooltip-api
description: Use this skill every time a tooltip should be shown next to an element.
---
```

Many skills? You can group them, up to 3 levels deep:

```
skills/
  css/
    popover-api/SKILL.md
    tooltip-api/SKILL.md
  js/
    drag-and-drop/SKILL.md
```

## 2. Rules (why things go wrong)

They all come from the [exact search sequence](HOW_DOES_IT_WORK.md#exact-search-sequence) of the CLI:

| Rule | What happens if you break it |
| --- | --- |
| No `SKILL.md` in the repo root | the CLI takes the root one as the **only** skill and never looks into `skills/` |
| Unique `name:` in every `SKILL.md` | the first one found wins, the other is silently ignored |
| `name:` = folder name | not enforced by the CLI, but it is in the [spec](https://agentskills.io/specification) and avoids confusion |
| No skill inside another skill's folder | the CLI does not look deeper once it finds a `SKILL.md` |
| Each skill carries its own files | only the skill's own folder is installed - a file shared from `../common/` will be missing at the user's side |
| `name` + `description` in every `SKILL.md` | a skill without them is skipped with a warning |

Skills kept outside the usual folders (e.g. `tools/skills/`)? Tell the CLI with a [`skills.sh.json`](https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md) file in the repo root:

```json
{ "paths": ["./tools/skills"] }
```

A skill that should not be offered to users (work in progress, internal helper) - hide it:

```md
---
name: my-draft
description: ...
metadata:
  internal: true
---
```

## 3. Push and check

```sh
git add skills
git commit -m "add tooltip-api skill"
git push

npx skills add stopsopa/skill-popover-api --list
```

`--list` must show **both** skills. If one is missing, check the rules above.

## 4. Publish = install each skill once

skills.sh learns about skills only from install telemetry ([FAQ](https://www.skills.sh/docs/faq)), and it counts **per skill**. A skill nobody installed is not listed.

So install all of them once:

```sh
npx skills add stopsopa/skill-popover-api --skill '*'
```

or pick them one by one:

```sh
npx skills add stopsopa/skill-popover-api --skill popover-api
npx skills add stopsopa/skill-popover-api --skill tooltip-api
```

(Telemetry must be on - `DISABLE_TELEMETRY` and `DO_NOT_TRACK` not set.)

## 5. What you get on skills.sh

| Page | Address |
| --- | --- |
| Repo page - lists all skills, "2 skills", total installs | https://www.skills.sh/stopsopa/skill-popover-api |
| Skill page | https://www.skills.sh/stopsopa/skill-popover-api/popover-api |
| Skill page | `https://www.skills.sh/stopsopa/skill-popover-api/tooltip-api` |

Each skill has its own install counter. The repo page shows the sum.

## 6. How users install from a multi-skill repo

```sh
npx skills add stopsopa/skill-popover-api                      # asks which skills (multi-select)
npx skills add stopsopa/skill-popover-api --skill tooltip-api  # just one
npx skills add stopsopa/skill-popover-api --skill '*'          # all
npx skills add https://github.com/stopsopa/skill-popover-api/tree/main/skills/tooltip-api   # by folder url
```

Each installed skill gets its own folder in `.agents/skills/` and its own entry in `skills-lock.json`.

Put the one-skill command in your README for every skill - people usually want one, not all.

## 7. Updating

Same as with one skill ([UPDATE.md](UPDATE.md)): edit, commit, push.

- The repo has one git history, but the CLI looks at **each skill's folder separately**. Changing `tooltip-api` does not make `popover-api` show as outdated for users.
- A git tag (`v1.1.0`) covers the whole repo, so it is one version number for all skills in it. If you need separate versions per skill, use tags like `tooltip-api-v1.1.0` or separate repos.
- Adding a new skill later: repeat steps 1, 3 and 4 for it. Existing skills and their counters are untouched.

## One repo or many?

| | One repo, many skills | One repo per skill |
| --- | --- | --- |
| Maintenance | one place, one README, one license | more repos to look after |
| Install | user picks with `--skill` | shorter command |
| Versions / tags | shared | separate |
| Good for | related skills (a "pack") | unrelated skills |

More: [HOW_DOES_IT_WORK.md](HOW_DOES_IT_WORK.md), [HOW_TO_INSTAL.md](HOW_TO_INSTAL.md)
