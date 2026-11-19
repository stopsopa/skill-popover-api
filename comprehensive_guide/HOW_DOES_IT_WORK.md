# How does it all work

Example used everywhere in this guide: https://www.skills.sh/stopsopa/skill-popover-api

## The three pieces

| Piece | What it is | In our example |
| --- | --- | --- |
| **GitHub repo** | The only place where the skill really lives | https://github.com/stopsopa/skill-popover-api |
| **[`skills` CLI](https://github.com/vercel-labs/skills)** | An [npm tool](https://www.npmjs.com/package/skills) (`npx skills ...`) that copies skills from a repo into your project | `npx skills add stopsopa/skill-popover-api` |
| **[skills.sh](https://www.skills.sh)** | A website: a catalog + [leaderboard](https://www.skills.sh/docs) of skills. It does **not** host the skill | https://www.skills.sh/stopsopa/skill-popover-api |

The most important thing to understand: **skills.sh is not a registry like npm.**
Nothing is uploaded to it. The source of truth is always the [GitHub repo](https://github.com/stopsopa/skill-popover-api).

```
 author                                    user
   │                                         │
   │ git push                                │ npx skills add stopsopa/skill-popover-api
   ▼                                         ▼
┌──────────────┐      git clone       ┌─────────────┐
│ GitHub repo  │ ───────────────────► │ skills CLI  │ ──► files in your project
└──────────────┘                      └─────────────┘
                                             │ anonymous telemetry ("this skill was installed")
                                             ▼
                                      ┌─────────────┐
                                      │  skills.sh  │  catalog page + install counter
                                      └─────────────┘
```

## What a skill is

A skill is just a folder with a `SKILL.md` file (plus optional extra files). The format is an open standard: [agentskills.io](https://agentskills.io/specification).

In this repo ([SKILL.md](https://github.com/stopsopa/skill-popover-api/blob/main/skills/popover-api/SKILL.md), [popover.css](https://github.com/stopsopa/skill-popover-api/blob/main/skills/popover-api/assets/popover.css)):

```
skills/
  popover-api/
    SKILL.md            <- instructions for the AI agent
    assets/popover.css  <- extra file the instructions refer to
```

`SKILL.md` starts with a small header (frontmatter). Only `name` and `description` are [required](https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md):

```md
---
name: popover-api
description: Use this skill every time one desire to place one element ... in relation to some other element on the page. ...
---
```

- `name` - the id of the skill (lowercase, hyphens allowed).
- `description` - the agent reads this to decide **when** to load the skill. This is the most important line.

Other fields (`license`, `metadata.version`, ...) are just information. The CLI does not use them.

## What happens when somebody runs `npx skills add stopsopa/skill-popover-api`

1. The CLI turns `stopsopa/skill-popover-api` into the GitHub repo and clones it.
2. It searches the repo for `SKILL.md` files in a fixed order - see [Exact search sequence](#exact-search-sequence) below. It finds `skills/popover-api/SKILL.md`.
3. It asks which skill(s), which [agent(s)](https://github.com/vercel-labs/skills/blob/main/docs/agents.md), which scope (project or global) and which method (symlink or copy) - see [Which options to choose](#which-options-to-choose) below.
4. It copies the skill folder to `.agents/skills/popover-api/` - this is the one real copy.
5. For each chosen agent that does not read `.agents/skills/` by itself it makes a symlink to that copy, e.g. `.claude/skills/popover-api` -> `.agents/skills/popover-api` (or a real copy if you choose [`--copy`](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)).
6. It writes [`skills-lock.json`](https://github.com/vercel-labs/skills#readme) - a note saying "popover-api came from this repo, from this commit, and its files have this hash".
7. It sends an anonymous "this skill was installed" ping ([telemetry](https://www.skills.sh/docs/cli)). That ping is what makes the skill show up on skills.sh and what increases the install counter.

## Exact search sequence

Taken from the CLI source: [`discoverSkills` in src/skills.ts](https://github.com/vercel-labs/skills/blob/main/src/skills.ts) (docs: [creating-skills.md](https://github.com/vercel-labs/skills/blob/main/docs/creating-skills.md)).

1. **Repo root.** Is there a `SKILL.md` directly in the root? If yes -> that is the only skill, **stop here**.
2. Otherwise go through these folders, **in this order**, and collect every skill found:

   | # | Folder | How deep |
   | --- | --- | --- |
   | 1 | repo root - direct subfolders only (`<name>/SKILL.md`) | 1 level |
   | 2 | `skills/` | up to 3 levels |
   | 3 | `skills/.curated/` | up to 3 levels |
   | 4 | `skills/.experimental/` | up to 3 levels |
   | 5 | `skills/.system/` | up to 3 levels |
   | 6 | every agent folder, alphabetically: `.adal/skills/`, `.agents/skills/`, `.aider-desk/skills/`, ... `.claude/skills/`, ... `.opencode/skills/`, ... `.pi/skills/`, ... (about 60, full list below) | up to 3 levels |
   | 7 | folders listed in `skills.sh.json` -> `"paths"` (file in the repo root) | up to 3 levels |
   | 8 | folders declared in `.claude-plugin/marketplace.json` and `.claude-plugin/plugin.json` | 1 level |

3. **Fallback.** Only if nothing was found so far: search the whole repo, max 5 levels deep.

Rules that apply all the time:

- "Up to 3 levels" means `skills/<name>/`, `skills/<category>/<name>/` or `skills/<category>/<category>/<name>/`.
- Once a folder has a `SKILL.md`, the CLI does not look deeper inside it.
- Never entered: `node_modules`, `.git`, `dist`, `build`, `__pycache__`.
- Two skills with the same `name:` -> **the first one found wins**, the later one is ignored.
- A `SKILL.md` without `name` or `description` is skipped with a warning.
- A skill with `metadata.internal: true` is hidden.
- `--full-depth` changes two things: a root `SKILL.md` no longer stops the search, and the fallback (step 3) always runs.

For this repo: step 1 - no `SKILL.md` in the root. Row 1 - no skill directly in a root subfolder. Row 2 - `skills/popover-api/SKILL.md` found.

So the safest place for your skill is `skills/<skill-name>/SKILL.md`.

<details>
<summary>Full list for row 6 (read from the CLI source in November 2026 - it grows with every new agent)</summary>

`.adal/skills`, `.agents/skills`, `.aider-desk/skills`, `.augment/skills`, `.autohand/skills`, `.bob/skills`, `.claude/skills`, `.cline/skills`, `.codeartsdoer/skills`, `.codebuddy/skills`, `.codemaker/skills`, `.codestudio/skills`, `.codex/skills`, `.commandcode/skills`, `.continue/skills`, `.cortex/skills`, `.crush/skills`, `.devin/skills`, `.factory/skills`, `.forge/skills`, `.github/skills`, `.goose/skills`, `.grok/skills`, `.hermes/skills`, `.iflow/skills`, `.inferencesh/skills`, `.jazz/skills`, `.junie/skills`, `.kilo/skills`, `.kilocode/skills`, `.kimchi/skills`, `.kiro/skills`, `.kode/skills`, `.lingma/skills`, `.mcpjam/skills`, `.minimax/skills`, `.moxby/skills`, `.mux/skills`, `.neovate/skills`, `.ona/skills`, `.opencode/skills`, `.openhands/skills`, `.pi/skills`, `.pochi/skills`, `.posit/assistant/skills`, `.qoder/skills`, `.qwen/skills`, `.reasonix/skills`, `.roo/skills`, `.rovodev/skills`, `.tabnine/agent/skills`, `.terramind/skills`, `.tinycloud/skills`, `.trae/skills`, `.vibe/skills`, `.windsurf/skills`, `.zcode/skills`, `.zencoder/skills`, `agent/skills`, `data/skills`

The list is built from [src/agents.ts](https://github.com/vercel-labs/skills/blob/main/src/agents.ts) and sorted alphabetically.

</details>

## Which options to choose

### The idea: "universal" agents

`.agents/skills/` is a shared folder that many agents read **by themselves**. The CLI calls them *universal* agents ([src/agents.ts](https://github.com/vercel-labs/skills/blob/main/src/agents.ts), [agents.md](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)).
OpenCode, Pi, Antigravity, Codex, Cursor, Gemini CLI and others are universal.

**Claude Code is not** - it reads only `.claude/skills/`. So for Claude Code the CLI adds a symlink `.claude/skills/popover-api` -> `.agents/skills/popover-api`.

### Table: what each harness needs

| Harness | `-a` id | Reads `.agents/skills/` by itself? | Project folder | Global folder (`-g`) | What to pick in the "Which agents" question |
| --- | --- | --- | --- | --- | --- |
| Claude Code CLI | `claude-code` | no | `.claude/skills/` (symlink) | `~/.claude/skills/` (symlink) | **tick "Claude Code"** |
| Antigravity CLI | `antigravity-cli` | yes | `.agents/skills/` | `~/.gemini/config/skills/` (symlink) | nothing in a project. For global: pass `-a antigravity-cli` |
| OpenCode | `opencode` | yes | `.agents/skills/` | `~/.agents/skills/` | nothing - already in the locked "Universal (.agents/skills)" group |
| Pi | `pi` | yes | `.agents/skills/` | `~/.agents/skills/` | nothing - already in the locked "Universal (.agents/skills)" group |

(Antigravity CLI is universal but is not shown in the "Universal" group on screen. In a project it works anyway, because the files always land in `.agents/skills/`.)

### The most universal choice

| Question | Pick | Why |
| --- | --- | --- |
| Which agents | **Claude Code** (the Universal group is always included) | the only one of the four that needs its own folder |
| Installation scope | **Project** | all four read the project folders the same way; global folders differ per harness |
| Installation method | **Symlink (Recommended)** | one real copy in `.agents/skills/`, everything else points to it |

The same without any questions:

```sh
npx skills add stopsopa/skill-popover-api --skill popover-api -a claude-code antigravity-cli opencode pi -y
```

Result - works in all four harnesses:

```
.agents/skills/popover-api/     <- real files: OpenCode, Pi, Antigravity CLI read this
.claude/skills/popover-api      <- symlink to the above: Claude Code reads this
skills-lock.json
```

On Windows, or when symlinks cause trouble (some zip/sync tools break them), use `--copy` - then each folder gets real files.

## The role of `skills-lock.json`

It is for skills what `package-lock.json` is for npm packages. It lives in the project root and is written by the CLI - don't edit it by hand.

For every installed skill it remembers:

| Field | Meaning | In our example |
| --- | --- | --- |
| source | where the skill came from | `stopsopa/skill-popover-api` |
| `commit` | the exact git commit the files were taken from | - |
| `computedHash` | hash of the installed files | - |

What it is used for:

- **Same version for everybody** - `npx skills install` on another machine brings back exactly the recorded commit, not "the newest".
- **Update check** - `npx skills check` compares the recorded commit with GitHub ([UPDATE.md](UPDATE.md)).
- **Detecting hand edits** - if the files no longer match `computedHash`, `npx skills list` / `check` say "modified" / "has local changes".
- **CI guard** - `npx skills install --frozen-lockfile` fails when files and lock don't match.

**Always commit `skills-lock.json`.**

Just cloned a project that has this file? See [HOW_TO_INSTAL.md](HOW_TO_INSTAL.md#cloned-a-project-that-already-has-skills-lockjson).

## How the agent uses the skill

(Details for Claude Code: [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills))

- On start the agent reads only `name` + `description` of every installed skill (cheap).
- When your prompt matches the description, the agent loads the full `SKILL.md` and follows it.
- Files next to it (like `assets/popover.css`) are read only when `SKILL.md` points to them.

## Where to go next

- Official docs: [skills.sh/docs](https://www.skills.sh/docs), [FAQ](https://www.skills.sh/docs/faq), [CLI repo](https://github.com/vercel-labs/skills)
- [HOW_TO_INSTAL.md](HOW_TO_INSTAL.md) - install a skill in your project
- [PUBLISH.md](PUBLISH.md) - get your own skill on skills.sh
- [UPDATE.md](UPDATE.md) - release a new version and pull it into a project
