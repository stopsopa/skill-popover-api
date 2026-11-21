
# example
https://github.com/stopsopa/gcp-css

# TLDR

```sh

npx skills init page
npx skills add stopsopa/stopsopa.github.io -s page -a claude-code -y
  # use to install and also to the same to update

npx -y skills@1.9.0 experimental_install --frozen-lockfile
# to bring all skills based on skills-lock.json  
# -y                 npx flag (not skills): auto-answers "Ok to proceed?" when skills@1.9.0 isn't in the npm cache, so the command never waits for input
# --frozen-lockfile  fail instead of modifying skills-lock.json if the restore doesn't match it (like `npm ci` vs `npm install`); already the default in CI

  # to fix potential issues with --frozen-lockfile
    NODE_OPTIONS= npx -y skills@1.9.0 experimental_install --no-frozen-lockfile

```

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

# to test
    npx skills add stopsopa/skill-popover-api --list
  
# to install

    npx skills add stopsopa/skill-popover-api
    npx skills add stopsopa/skill-popover-api -a claude-code
    # With no questions at all:
        npx skills add stopsopa/skill-popover-api -a claude-code -y

    # For all your projects instead of just this one, add -g:
        npx skills add stopsopa/skill-popover-api -a claude-code -g

    npx skills add stopsopa/stopsopa.github.io -s page -a claude-code -y
    

After the project install you should have:
    .agents/skills/popover-api/     <- the real files
    .claude/skills/popover-api      <- link to them; this is what Claude Code reads
    skills-lock.json  

```

# test locally before push

```
npx skills add . --list
```

- `.` = "use this folder as the source" instead of a GitHub repo (any local path works: `./`, `../other-repo`)
- `--list` = only show which skills the CLI finds, install nothing
- run it in the repo root

why useful: it is the same search the CLI does for users, but on your **uncommitted** files. So before `git push` you see:

- is every skill found? (wrong folder -> not on the list)
- is `name` + `description` ok? (broken header -> skill skipped with a warning)
- with many skills: are all listed? (same `name:` twice -> only the first one shows)

`npx skills add stopsopa/skill-popover-api --list` checks the same, but for what is already **pushed** to GitHub.

real local install without push (e.g. to try the skill in another project):

```
cd ../some-project
npx skills add ../skill-popover-api
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

`--skill '*'` vs `-y`:

| question the CLI asks | `--skill '*'` | `-y` |
| --- | --- | --- |
| which skills? | all | all |
| which agents? | **asks** | detected agents + universal `.agents/skills` |
| project or global? | **asks** | project |
| symlink or copy? | **asks** | symlink |
| proceed? | **asks** | yes |

`--skill '*'` answers one question, `-y` answers all of them with defaults.

fully explicit, no questions:

```
npx skills add stopsopa/skill-popover-api --skill '*' -a claude-code -y
```

# daily usage (in a project that uses skills)

```
npx skills list       # what is installed + state (tracked / modified / missing)
npx skills check      # is there a newer version? any local edits?
npx skills update     # take newer versions
npx skills update popover-api   # only one
npx skills remove popover-api
```

# global skills (your user dir in the OS)

project = `./.agents/skills` (this repo only), global = `~/.agents/skills` (all your projects).

every command works on the project by default. Add `-g` for global:

```
npx skills list -g                  # show global skills
npx skills list -g -a claude-code   # ... for one agent only
npx skills add owner/repo -g        # install globally
npx skills update -g                # update global ones
npx skills remove popover-api -g    # remove a global one
```

or just look at the folders:

```
ls -la ~/.agents/skills/    # the real copies
ls -la ~/.claude/skills/    # Claude Code's links to them
```

# update

```
npx skills check
npx skills update -y
git diff
git add .agents .claude skills-lock.json
git commit -m "update skills"
```

what `update` touches:

| command | updates |
| --- | --- |
| `npx skills update` | **asks** project or global, then all outdated skills there |
| `npx skills update -p` | only this project (`./.agents/skills`, `./skills-lock.json`) - best for daily use |
| `npx skills update -g` | only global (`~/.agents/skills`) |
| `npx skills update -y` | no question: project if you are in a project dir, otherwise global |
| `npx skills update popover-api` | only that skill |

- only skills recorded in the lock file - hand-made skill folders are left alone
- skills already up to date are skipped
- project and global are separate, each has its own lock file

nothing updates by itself. `update` overwrites local edits in the skill folder.

author side: just edit `skills/popover-api/*`, commit, push. No re-publish.

# remove

| command | removes from |
| --- | --- |
| `npx skills remove popover-api` | **this project**: `./.agents/skills/popover-api`, links like `./.claude/skills/popover-api`, entry in `./skills-lock.json` |
| `npx skills remove popover-api -g` | **your user dir**: `~/.agents/skills/popover-api` + links like `~/.claude/skills/popover-api` |

- default = project. Global is untouched without `-g`
- installed in both? removing one leaves the other, agent still sees it
- local only - nothing changes on GitHub or skills.sh
- in a project: commit the result (deleted files + `skills-lock.json`)

where is it installed?

```
npx skills list       # project
npx skills list -g    # global
```

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

is `-a` needed? no. Files **always** go to `./.agents/skills`. `-a` only decides which extra agent links are made:

| command | `.agents/skills` | extra links |
| --- | --- | --- |
| `skills install` | yes | agents last picked with `skills add`, else agents detected on your machine |
| `skills install -a claude-code` | yes | `.claude/skills/<name>` |
| `skills install -a opencode` (or `pi`) | yes | none - they read `.agents/skills` directly |

- want only `.agents/skills`, nothing else -> `skills install -a opencode`
- Claude Code does **not** read `.agents/skills` - it needs the `.claude/skills` link (`-a claude-code`)
- `skills install --dry-run` shows what would be created, changes nothing

CI:

```
npx skills install --frozen-lockfile   # fail if files != lock
```

details: [UPDATE.md](UPDATE.md), [HOW_TO_INSTAL.md](HOW_TO_INSTAL.md), [HOW_DOES_IT_WORK.md](HOW_DOES_IT_WORK.md)

