# How to install a skill

Example: https://www.skills.sh/stopsopa/skill-popover-api

Requires [Node.js](https://nodejs.org) (the [CLI docs](https://github.com/vercel-labs/skills#readme) say 22.20 or later).

## Install

Run this in the root of your project:

```sh
npx skills add stopsopa/skill-popover-api
```

The CLI asks a few questions (which skill, which agent, project or global, symlink or copy).
The defaults are fine.

To only look at what the repo offers, without installing:

```sh
npx skills add stopsopa/skill-popover-api --list
```

## Useful flags

Full list: [commands.md](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)

| Flag | Meaning |
| --- | --- |
| `--skill popover-api` | install only this skill from the repo |
| `-a claude-code` | install for this agent only ([list of agents](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)) |
| `-g` | global: available in all your projects (goes to your home directory) |
| `--copy` | real copies instead of symlinks |
| `-y` | no questions, accept defaults |

Example without any questions:

```sh
npx skills add stopsopa/skill-popover-api --skill popover-api -a claude-code -y
```

## Project or global?

| | Project (default) | Global (`-g`) |
| --- | --- | --- |
| Where | `./.agents/skills/` in the project | `~/.agents/skills/` in your home dir |
| Who gets it | everyone who clones the project (if you commit it) | only you, in every project |

## What appears in your project

```
.agents/skills/popover-api/     <- the real files
.claude/skills/popover-api      <- symlink to the above (one per chosen agent)
skills-lock.json                <- where the skill came from + which commit + files hash
```

Commit all of these, so your teammates get the same skill in the same version.

Which agent / scope / method to pick so it works in Claude Code, Antigravity CLI, OpenCode and Pi at once: [HOW_DOES_IT_WORK.md](HOW_DOES_IT_WORK.md#which-options-to-choose).

## Cloned a project that already has `skills-lock.json`

`skills-lock.json` is the list of skills the project uses (source + exact commit + files hash) - like `package-lock.json`. More: [the role of skills-lock.json](HOW_DOES_IT_WORK.md#the-role-of-skills-lockjson).

You do **not** run `npx skills add ...` again. You restore what the lock file says:

```sh
# 1. get the project
git clone git@github.com:your-org/your-project.git
cd your-project

# 2. Node.js is needed (22.20 or later)
node -v

# 3. install the skills CLI (once per machine)
npm install -g skills
skills --version

# 4. bring in all skills listed in skills-lock.json
skills install -a claude-code

# 5. check
skills list
```

Notes:

- Step 3 is optional. Without it, put `npx` in front: `npx skills install -a claude-code`, `npx skills list`.
- `skills install` (no repo name after it) = "restore from `skills-lock.json`". Full name of the command: [`experimental_install`](https://github.com/vercel-labs/skills/blob/main/docs/commands.md). It asks no questions.
- It downloads each skill at the **exact commit** saved in the lock file into `.agents/skills/` and creates the agent links (`.claude/skills/...`).
- `-a claude-code` says which agents to link. OpenCode, Pi and Antigravity CLI read `.agents/skills/` directly, so they need nothing extra. Without `-a` the CLI uses the agents it detects on your machine.
- Files already present and matching the lock are left alone. `--force` reinstalls everything, `--dry-run` only shows what would happen.
- If the project has a `package.json`, run `npm install` first.

### When the skill files are committed too

If the repo already contains `.agents/skills/` and `.claude/skills/` (the recommended way), the skills work right after `git clone` - nothing to install.
Running `skills install` is then just a harmless check. `skills list` should show every skill as `tracked`.

### In CI

```sh
npx skills install --frozen-lockfile
```

Fails (exit code 1) when the skills on disk and `skills-lock.json` don't match. In CI this is the default.

### After that

```sh
skills check     # anything newer upstream?
skills update    # take it - then commit the changed files + skills-lock.json
```

Details: [UPDATE.md](UPDATE.md)

## Check, use, remove

```sh
npx skills list                  # what is installed
npx skills remove popover-api    # uninstall
```

To use it, just ask your agent (more prompts in the [README](https://github.com/stopsopa/skill-popover-api#readme), all positioning classes on the [demo page](https://stopsopa.github.io/skill-popover-api/demo/index.html)), e.g.:

```
Use popover api skill to render element with id popover on top of button with class trigger.
```

## Pin a specific version

```sh
npx skills add stopsopa/skill-popover-api@v1.0.0 --skill popover-api
```

`@v1.0.0` is a git tag (or a full commit sha) in the [skill repo](https://github.com/stopsopa/skill-popover-api/tags). It works only if the author created such a tag.

## Don't want telemetry?

What is collected: [skills.sh/docs/cli](https://www.skills.sh/docs/cli)

```sh
DISABLE_TELEMETRY=1 npx skills add stopsopa/skill-popover-api
```

(Such an install is not counted on [skills.sh](https://www.skills.sh/stopsopa/skill-popover-api).)

Trouble? See [troubleshooting.md](https://github.com/vercel-labs/skills/blob/main/docs/troubleshooting.md).

More: [HOW_DOES_IT_WORK.md](HOW_DOES_IT_WORK.md), updating: [UPDATE.md](UPDATE.md)
