# Prompt: update this PM repo from the template

Paste the following into a PM thread on a repo that was cloned from `pm-co-pilot-template`. The thread will fetch the template, compare file-by-file, and apply the relevant changes.

---

You are the PM thread for a repo that was cloned from `https://github.com/AquiGorka/pm-co-pilot-template`. Update this repo to the current state of the template.

## Setup

If `template` is not already a git remote, add it:

```bash
git remote add template https://github.com/AquiGorka/pm-co-pilot-template
```

Fetch:

```bash
git fetch template main
```

## Compare file-by-file, not commit-by-commit

The template's history may have been rewritten (force-pushed). Do not assume your local clone shares ancestry with `template/main`. Compare current state file-by-file:

```bash
git diff main template/main -- <file>
```

For an overview of which files differ:

```bash
git diff --name-status main template/main
```

## Conflict resolution: who wins for which file

| Files | Source of truth | What to do on conflict |
|---|---|---|
| `CLAUDE.md` | **template** | Overwrite local with template version. |
| `prompts/README.md` | **template** | Overwrite. |
| `metrics.md` | **template** | Overwrite. |
| `update-from-template-prompt.md` | **template** | Overwrite. |
| `feedback/README.md`, `project/README.md`, `reference/README.md`, `user/README.md` | **template** | Overwrite (these are format docs, not rule content). |
| `workflows/README.md`, `skills/README.md` (if present) | **template** | Overwrite. |
| `integrations/_template.md` | **template** | Overwrite. |
| `commits.md`, `weekly-update-format.md`, `board-structure.md`, `milestones.md` | **template** | Overwrite unless this project has explicit customisation documented in commit history; in that case prefer local and surface the diff. |
| `README.md` | hybrid | Never auto-overwrite. The structure is template-owned, the content is project-customised. Review the diff and cherry-pick structural improvements (new sections, reordering) without disturbing project-specific content. |
| `actionables.md`, `workstreams.md`, `decisions.md` | **local** | Never overwrite. The template version is a placeholder. |
| `daily/*` | **local** | Never overwrite. |
| `prompts/<topic>-<N>-prompt.md`, `prompts/<topic>-<N>-progress.md` | **local** | Never overwrite. These are project work artifacts. |
| `feedback/*.md` (rule files, not the README), `project/*.md`, `reference/*.md`, `user/*.md` | **local** | Never overwrite. If the template introduces a new rule file the project does not have, surface it as a recommendation. |
| `integrations/*.md` (project-specific service docs) | **local** | Never overwrite. |
| `skills/*.md`, `workflows/*.md` | **local** | Never overwrite. |
| `.env`, `.env.enc` | **local** | Never overwrite. |

**Rule of thumb:** if the file's purpose is to define the schema or convention (it ends in `README.md`, or is one of the named template docs above), the template wins. If the file's purpose is to hold this project's state or content, local wins.

## Apply

For each file where the template wins and there is a diff, copy the template version into the working tree:

```bash
git checkout template/main -- <file>
```

For each file where local wins, do nothing.

For `README.md`, open both versions and merge by hand: pull in structural improvements, leave project-specific content alone.

## Recommendations from new template content

If the template introduces a new rule or pattern that this project has not adopted yet (a new `feedback/<rule>.md`, a new `workflows/<pattern>.md`, a new convention in `commits.md`), do **not** auto-apply. Surface as a recommendation in your final report and let the user decide.

## Constraints

- Do not introduce em-dashes anywhere in any file you write or update.
- Stop and ask if anything is ambiguous. Do not guess.
- When done, stage the changes with `git add -A`, commit with a message like `chore: sync from pm-co-pilot-template@<commit-sha>` (use the SHA of `template/main`), and push.

End by reporting: which template-owned files were overwritten, which local-owned files were left alone, which template recommendations are pending the user's decision.
