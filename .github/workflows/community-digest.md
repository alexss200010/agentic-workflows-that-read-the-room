---
name: community-digest
description: Publish a periodic narrative digest of repository activity as a GitHub Discussion
on:
  schedule:
    - cron: "0 9 * * 1-5"
  slash_command:
    name: digest
permissions:
  contents: read
  issues: read
  pull-requests: read
  discussions: read
engine: copilot
tools:
  github:
    mode: gh-proxy
    toolsets: [default]
safe-outputs:
  create-discussion:
    close-older-discussions: true
    category: announcements
timeout-minutes: 30
evals:
  - id: operational_value
    question: "Does the output clearly demonstrate that the repository's recent activity has been translated into a brief, actionable digest that shows the current state of development for the last 24 hours?"
  - id: scope_coverage
    question: "YES if the digest covers recent branch activity, authorship, merged work, and PR/issue activity from the last 24 hours."
  - id: repo_verification
    question: "YES if the digest states whether relevant commits, branches, and merges still exist in the repository and calls out stale or missing work."
  - id: brevity
    question: "YES if the digest stays to a single brief screen of bullets and avoids fluff or narrative filler."
  - id: discussion_format
    question: "YES if the discussion post is written as a concise GitHub Discussion update intended for a community digest."
---

This repository is a GitHub Skills exercise for a lightweight Astro-based site. The codebase is small and documentation-focused: README.md is the main entry point; there is no AGENTS.md or CONTRIBUTING.md in the repo root; workflow files live under `.github/workflows`, and the project appears to be mostly static content with small automation around GitHub. Treat the GitHub repository itself as the source of truth, prefer read-only repo and activity checks, and keep the output brief, factual, and easy to scan.

Goal:
Give me a short daily digest of repository activity for the previous 24 hours, so I can see where development stands without opening GitHub: which branches got commits and who authored them, what was merged, which pull requests and issues were opened or cross-referenced, and whether everything that was done actually exists in the repo (commits reachable, branches not deleted, merges present in the target branch). Keep it brief: one screen, bullet points, no fluff.

Requirements:
- Publish a GitHub Discussion summarizing the previous 24 hours of repo activity.
- Run every weekday morning.
- Allow a manual `/digest` slash command for on-demand status checks.
- Prefer a brief bullet list with no fluff.
- Include branch authorship, merged work, new/updated PRs and issues, and a simple verification check that referenced commits/branches/merges still exist in the repository.
- If there is no significant activity, say so clearly and briefly.

steps:
  - name: Pre-fetch repository activity
    run: |
      echo "Fetching recent repository activity for the last 24 hours..."
      mkdir -p /tmp/gh-aw/agent
      gh api repos/${{ github.repository }}/commits --paginate --jq '.[] | {sha: .sha, author: .commit.author.name, date: .commit.author.date, message: .commit.message}' > /tmp/gh-aw/agent/commits.json
      gh api repos/${{ github.repository }}/branches --paginate > /tmp/gh-aw/agent/branches.json
      gh api repos/${{ github.repository }}/pulls?state=all\&sort=updated\&direction=desc\&per_page=100 > /tmp/gh-aw/agent/pulls.json
      gh api repos/${{ github.repository }}/issues?state=all\&sort=updated\&direction=desc\&per_page=100 > /tmp/gh-aw/agent/issues.json
      echo "Repository activity fetched."

prompt: |
  You are the repo activity digest agent for this repository.

  Repository context:
  - This is a GitHub Skills exercise for a lightweight Astro site.
  - The repo is small, documentation-heavy, and uses GitHub Issues/PRs as the core activity surface.
  - Default branch is `main`.
  - Keep the digest short, precise, and useful for a daily status check.
  - Prefetched activity files are available under `/tmp/gh-aw/agent/` (`commits.json`, `branches.json`, `pulls.json`, `issues.json`).

  Intent:
  Give me a short daily digest of repository activity for the previous 24 hours, so I can see where development stands without opening GitHub: which branches got commits and who authored them, what was merged, which pull requests and issues were opened or cross-referenced, and whether everything that was done actually exists in the repo (commits reachable, branches not deleted, merges present in the target branch). Keep it brief: one screen, bullet points, no fluff.

  Success criteria:
  - summarise the last 24 hours only
  - highlight branch activity and author names
  - identify merged work and relevant PRs/issues
  - mention cross references when present
  - state whether the repo activity is still valid and discoverable in the repository
  - keep it to a brief, readable bullet list

  Output format:
  - Write a GitHub Discussion post that is a short daily digest.
  - Use bullet points only.
  - If there was no material activity, say "No material repo activity in the last 24 hours."

  Repo verification rules:
  - Check whether branch names still exist.
  - Check whether commit SHAs are reachable from the default branch if a landing branch is referenced.
  - Check whether PRs or merges referenced in the digest still appear in the repo.
  - If something is missing, call it out plainly.

safe-outputs:
  create-discussion:
    close-older-discussions: true
    category: announcements
