# Prompt: update this PM repo from the template

Paste the following into a PM thread on a repo that was cloned from `pm-co-pilot-template`. The thread will fetch the template, review what is new, and cherry-pick the relevant changes into the local repo.

---

You are the PM thread for a repo that was cloned from `https://github.com/AquiGorka/pm-co-pilot-template`. Update this repo to the current state of the template.

Read your own `README.md` first, specifically the section titled **"Updating from the Template"**. That section documents the protocol: add the template as a `template` git remote if not already present, `git fetch template main`, review recent template commits since the last sync, and cherry-pick what is relevant. **Do not merge or rebase the template into this repo. Adapt, do not auto-apply.**

Apply the protocol step by step:

1. List the template commits since the last sync. If there has never been a sync, list the template commits since this repo's clone date.
2. For each commit, inspect the diff and decide: pull in, skip, or partial.
3. For each diff you cherry-pick, summarise what changed, where it lands, and why you applied it.
4. For each diff you skip, summarise why (most likely the file is already customised in this repo, or the change is project-specific to another project).
5. Stop and ask if anything is ambiguous. Do not guess.
6. When done, commit with a message like `chore: sync from pm-co-pilot-template@<commit-sha>` and push.

Constraints:

- Do not modify project-specific files (this project's `actionables.md`, `thread.md`, `workstreams.md`, `decisions.md`, `daily/`, anything under `prompts/`, etc.). The template's versions of those are placeholders and not relevant once the project has its own state.
- Do not pull in template content that conflicts with this project's existing rules. If a `feedback/` (or `project/`, `reference/`, `user/`) entry already exists locally, prefer the local version unless the template version is genuinely better and the local one is stale.
- Do not introduce em-dashes anywhere in any file you write or update.
- If the template introduces a new rule or convention this project has not adopted yet, surface it as a recommendation rather than silently applying it.

End by reporting: which commits were applied, which were skipped (with reason), and whether anything is pending the user's decision.
