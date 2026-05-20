# Dependabot Review Skill (thoughtbot)

A [Claude Code][claude-code] skill that reviews Dependabot gem upgrade pull
requests to assess impact, breaking changes, and merge readiness — either one
PR at a time or as a consolidated audit across every open Dependabot PR in the
repository.

[claude-code]: https://docs.anthropic.com/en/docs/claude-code
[thoughtbot]: https://thoughtbot.com

## Quick links

- **[Dependabot][dependabot]** - GitHub's automated dependency update tool
- **[RubyGems.org][rubygems]** - Canonical source for gem metadata and versions
- **[Keep a Changelog][keepachangelog]** - Changelog conventions used by most gems

[dependabot]: https://docs.github.com/en/code-security/dependabot
[rubygems]: https://rubygems.org
[keepachangelog]: https://keepachangelog.com

## Table of contents

- [Overview](#overview)
- [Installation](#installation)
- [Usage](#usage)
  - [Single-PR review](#single-pr-review)
  - [Audit mode](#audit-mode)
  - [Posting findings back to PRs](#posting-findings-back-to-prs)
- [Requirements](#requirements)
- [Output](#output)
- [Contributing](#contributing)
- [License](#license)
- [About thoughtbot](#about-thoughtbot)

## Overview

This skill reviews Dependabot pull requests and produces a concise, scannable
verdict covering:

- Bump type (patch, minor, major) and what it implies for risk
- Changelog highlights between the old and new version
- Breaking changes, deprecations, and security fixes
- Codebase impact — what the gem touches and which features depend on it
- Ecosystem compatibility — other gems that depend on the one being bumped
- A clear merge recommendation (`Merge`, `Verify`, `Investigate`, or `Hold`)

It works in two modes:

1. **Single-PR mode** — paste a Dependabot PR URL and get a full review.
2. **Audit mode** — discover every open Dependabot PR in the current repo,
   analyze them one by one, and produce a consolidated triage report sorted by
   ease of merge, with security-flagged PRs floated to the top.

Optionally, the skill can post each review back to its PR as a collapsible
comment, so the work lives where the team actually reviews code — not just in
the chat transcript.

## Installation

Copy the skill directory to your Claude Code skills folder:

```bash
cp -r dependabot-review-thoughtbot ~/.claude/skills/
```

Or clone directly:

```bash
git clone https://github.com/thoughtbot/dependabot-review-thoughtbot ~/.claude/skills/dependabot-review-thoughtbot
```

## Usage

Invoke the skill inside a Claude Code session. You should be at the root of the
Rails project whose Dependabot PRs you want to review so the skill can search
the codebase for gem usage.

### Single-PR review

Paste a Dependabot PR URL (or reference one PR):

```
review https://github.com/my-org/my-app/pull/9170
```

The skill parses the URL, fetches the PR diff, pulls the changelog between the
old and new version, searches the codebase for usage, and returns a structured
review.

Trigger phrases that also activate single-PR mode include mentions of a gem
version bump, "is this Dependabot PR safe to merge?", or any GitHub PR URL with
"bump" in the title.

### Audit mode

Audit every open Dependabot PR in the current repo:

```
/dependabot-review-thoughtbot
```

Or any of the phrases that trigger it directly:

- "review all open dependabot PRs"
- "which dependabot PRs are ready to merge"
- "audit our dep upgrades"
- "go through the open dep PRs"
- "check dependabot"

The skill discovers the PRs via `gh pr list --author "app/dependabot"` — you do
not need to paste URLs. It produces:

1. A **summary table** (PR number, gem, bump, type, age, verdict, why) sorted
   worklist-style: `Merge → Verify → Investigate → Hold`, with stalest PRs
   rising inside each bucket and security-flagged PRs floated to the top.
2. A **Details** section with a condensed per-PR review.
3. An **Overall recommendation** block grouped by verdict so you can work
   top-to-bottom.

### Posting findings back to PRs

After presenting the review in chat, the skill asks whether to post the review
as a comment on the PR(s):

- **Single-PR mode:** yes / no
- **Audit mode:** yes / no / selective (pick specific PR numbers)

Comments use a collapsible `<details>` block with the verdict and one-line
reason above the fold, and include an invisible marker so re-runs can detect
and skip PRs that already have a prior review comment.

No comment is ever posted without explicit confirmation.

## Requirements

- [`gh`](https://cli.github.com/) CLI installed and authenticated against the
  repository you are auditing (used to list PRs, read diffs, and post comments)
- A Rails project checkout — the skill greps the codebase to assess impact
- Network access to fetch changelogs from GitHub and RubyGems.org

## Output

Each review follows this structure:

```
## Dependabot Review: `gem_name` (old_version -> new_version)

### Bump Type
[patch/minor/major] — [one line: what this means for risk]

### What Changed
[Changelog highlights: breaking changes first, then deprecations,
security fixes, and notable changes.]

### Breaking Changes in This Codebase
[Only if breaking changes actually affect this repo — each with
the affected file and a concrete fix suggestion.]

### Codebase Impact
[Grouped list of what this gem touches: "Payments: captures
charges via checkoutcom jobs", etc.]

### Recommendation
[Merge / Verify / Investigate / Hold, plus 1-3 sentences of
reasoning and anything a human should verify that CI won't catch.]
```

Multi-gem PRs get one section per gem and a single combined recommendation at
the end. Audit-mode per-PR sections use the same structure but condensed to
15-25 lines each.

## Contributing

Contributions are welcome! If you'd like to improve the review heuristics,
refine the audit table, or add detection for new changelog sources:

1. Fork the repository
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## License

This skill is open source and available under the [MIT License](LICENSE).

## About thoughtbot

![thoughtbot](https://thoughtbot.com/thoughtbot-logo-for-readmes.svg)

This skill was built at [thoughtbot][thoughtbot] to make dependency upgrade
review faster and more consistent across the Rails projects we maintain.

The names and logos for thoughtbot are trademarks of thoughtbot, inc.

We love open source software!
See [thoughtbot's other projects][community].

[community]: https://thoughtbot.com/community?utm_source=github
