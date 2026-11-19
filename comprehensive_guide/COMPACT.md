
# just create 

`skills/popover-api/SKILL.md`

with ( you can use `npx skills init popover-api`)

```
---
name: popover-api
description: Use this skill every time one desire to place one element (tooltip, menu, popover, modal etc) in relation to some other element on the page. Always use the modern native browser popover api, not css positioning hacks.
license: GPL-2.0-only
metadata:
  author: stopsopa
  version: "1.0"
  website: https://github.com/stopsopa/skill-popover-api
---
```

next to that file in the same directory put whatever you want to be part of that skill and you good to go

from now on:

```

npx skills add stopsopa/skill-popover-api --list
  
  # to test

npx skills add stopsopa/skill-popover-api
  
  # to install

```

# multiple skills in one repo

to install them all

```
npx skills add stopsopa/skill-popover-api --skill '*'
```

what happens depending on `--skill`:

| command | repo with 1 skill | repo with many skills |
| --- | --- | --- |
| `npx skills add owner/repo` | installs it | **asks** - shows a checklist, you tick which ones |
| `npx skills add owner/repo --skill tooltip-api` | - | only that one, no checklist |
| `npx skills add owner/repo --skill '*'` | installs it | **all**, no checklist |
| `npx skills add owner/repo -y` | installs it | all, no questions at all |
| `npx skills add owner/repo --all` | installs it | all skills, to all agents, no questions |

so: no `--skill` = "let me choose", `--skill '*'` = "all, don't ask which" (still asks about agent + scope).

only skill folders are copied - nothing else from the repo, no npm packages.

# daily usage (in a project that uses skills)

```
npx skills list       # what is installed + state (tracked / modified / missing)
npx skills check      # is there a newer version? any local edits?
npx skills update     # take newer versions
npx skills update popover-api   # only one
npx skills remove popover-api
```

# update

```
npx skills check
npx skills update -y
git diff
git add .agents .claude skills-lock.json
git commit -m "update skills"
```

nothing updates by itself. `update` overwrites local edits in the skill folder.

author side: just edit `skills/popover-api/*`, commit, push. No re-publish.

# skills-lock.json

like `package-lock.json` but for skills. One entry per skill: where from + exact git commit + hash of files.

- written by `add` / `update` / `remove` - never edit by hand
- always commit it
- `check` uses it to see what is outdated
- gives everybody the same version

fresh clone of a project with `skills-lock.json`:

```
npm install -g skills            # once per machine (or use npx everywhere)
skills install -a claude-code    # restore exactly what the lock says, no questions
skills list
```

(if `.agents/skills` + `.claude/skills` are committed too, skills already work after clone - `install` is just a check)

CI:

```
npx skills install --frozen-lockfile   # fail if files != lock
```

details: [UPDATE.md](UPDATE.md), [HOW_TO_INSTAL.md](HOW_TO_INSTAL.md), [HOW_DOES_IT_WORK.md](HOW_DOES_IT_WORK.md)

