# How to update a skill

Example: https://www.skills.sh/stopsopa/skill-popover-api

There are two different "updates":

1. **Author side** - you changed the skill and want the world (and skills.sh) to have the new version.
2. **User side** - a skill installed in your project is old and you want the new version.

---

## 1. Author: release a new version of the skill

### What to do

Edit the files and push to the default branch of the [repo](https://github.com/stopsopa/skill-popover-api). That's all.

```sh
# edit skills/popover-api/SKILL.md or skills/popover-api/assets/popover.css
git add skills
git commit -m "popover-api: improve positioning rules"
git push
```

Optional but nice:

```sh
git tag v1.1.0
git push --tags
```

A [tag](https://github.com/stopsopa/skill-popover-api/tags) lets users install an exact version: `npx skills add stopsopa/skill-popover-api@v1.1.0`.

### How it works under the hood

- There is nothing to re-publish. skills.sh does not hold a copy of the skill to replace - [GitHub](https://github.com/stopsopa/skill-popover-api) is the source of truth.
- Every new `npx skills add stopsopa/skill-popover-api` clones the repo at that moment, so it gets the newest commit right away.
- The [page on skills.sh](https://www.skills.sh/stopsopa/skill-popover-api/popover-api) shows content taken from the GitHub repo. It may lag behind a little; the docs do not say how often it refreshes. The install counter is not reset.
- The "version" of a skill is simply the **[git commit](https://github.com/stopsopa/skill-popover-api/commits/main)**. `metadata.version` in [`SKILL.md`](https://github.com/stopsopa/skill-popover-api/blob/main/skills/popover-api/SKILL.md) is only a label for humans - bump it if you like, but the CLI ignores it.
- Don't rename the skill folder or `name:` in an update. For the CLI and skills.sh that is a different skill (new page, counter from zero, users' updates stop working).

---

## 2. User: update a skill installed in your project

### What to do

```sh
npx skills check     # is there anything newer?
npx skills update    # download the newer version
```

Variants (all options: [commands.md](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)):

```sh
npx skills update popover-api   # only this skill
npx skills update -p            # only project skills
npx skills update -g            # only global skills
npx skills update -y            # no questions
```

Then review and commit the change (skill files + `skills-lock.json`):

```sh
git diff
git add .agents skills-lock.json
git commit -m "update popover-api skill"
```

### How it works under the hood

When you installed the skill, the CLI wrote an entry to [`skills-lock.json`](https://github.com/vercel-labs/skills#readme) with:

- **source** - `stopsopa/skill-popover-api`
- **commit** - the git commit the files were taken from
- **computedHash** - a hash of the installed files

`npx skills check`:

1. Reads the lock file.
2. Asks GitHub what the skill folder looks like now (via the GitHub API, see [troubleshooting.md](https://github.com/vercel-labs/skills/blob/main/docs/troubleshooting.md)).
3. If GitHub has something newer than the recorded commit -> "update available".
4. It also re-hashes your local files. If they differ from `computedHash` -> "has local changes" (somebody edited the skill by hand).
5. Exit code is 1 when anything is outdated or changed - handy in CI.

`npx skills update`:

1. Downloads the skill again from GitHub.
2. Replaces the files in `.agents/skills/popover-api/`.
3. Moves `commit` and `computedHash` in `skills-lock.json` forward.

The symlinks (e.g. `.claude/skills/popover-api`) point to that folder, so every agent sees the new version immediately.

### Things to know

- **Nothing updates by itself.** You stay on the old version until you run `update`.
- **Local edits get overwritten** by `update`. If you need custom changes, [fork the repo](https://github.com/stopsopa/skill-popover-api/fork) and install from your fork.
- **Teammates** don't run `update`. They just `git pull` your commit. (If the skill files are not committed, [`npx skills install`](https://github.com/vercel-labs/skills/blob/main/docs/commands.md) restores exactly what the lock file says.)
- **Global skills** (`-g`) work the same way, only the files and the lock file live in your home directory.
- **Pinned to a tag** (`@v1.0.0`)? To move to another one, add it again: `npx skills add stopsopa/skill-popover-api@v1.1.0`.

### If update does not work

Remove and add again - this always gives the newest version:

```sh
npx skills remove popover-api
npx skills add stopsopa/skill-popover-api
```

Still stuck: [troubleshooting.md](https://github.com/vercel-labs/skills/blob/main/docs/troubleshooting.md), [skills.sh FAQ](https://www.skills.sh/docs/faq).

More: [HOW_DOES_IT_WORK.md](HOW_DOES_IT_WORK.md)
