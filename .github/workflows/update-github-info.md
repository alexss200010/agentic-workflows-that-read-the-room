---
name: update-github-info
description: Refresh the GitHub Info page with useful updates from the GitHub Blog and Changelog for Mona to review.
on:
  workflow_dispatch:
permissions:
  contents: read
tools:
  edit:
  web-fetch:
network:
  allowed:
    - defaults
    - github.blog
    - github.com
safe-outputs:
  create-pull-request:
    allowed-files:
      - site/content/github-info.md
    reviewers:
      - mona
    draft: false
---

Follow these steps in order:

1. Read `notes/mona-notes.md` and follow Mona's editorial guidance.
2. Use `web-fetch` to read both the GitHub Blog at https://github.blog/latest/ and the GitHub Changelog at https://github.blog/changelog/.
3. Update `site/content/github-info.md` with concise, practical information that helps developers learn GitHub faster. Preserve the page's existing editorial angle and themes, attribute every update to its GitHub Blog or Changelog source, and include only claims supported by the fetched sources.
4. If you made a meaningful update, use the configured `safe-outputs.create-pull-request` output (`create_pull_request`) to open a pull request for Mona to review. Include a short change summary and links to the sources in the pull request description. Do not write directly to `main` or use another write mechanism.

If the sources contain no relevant new information or the page needs no meaningful change, make no edit and use `noop` instead of opening a pull request.
