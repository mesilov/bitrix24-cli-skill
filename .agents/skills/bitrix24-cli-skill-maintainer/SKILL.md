---
name: bitrix24-cli-skill-maintainer
description: Use when maintaining the bitrix24-cli-skill repository, changing its skills, references, metadata or documentation, or preparing its issues and pull requests. Do not use for ordinary Bitrix24 task operations.
---

# Maintain bitrix24-cli-skill

Maintain the portable [bitrix24-tasks package](../../../skills/bitrix24-tasks/SKILL.md)
and repository documentation. This maintainer skill stays in this repository;
it is not part of the user-facing package copied from `skills/bitrix24-tasks`.

## Scope and delivery

- Read [README.md](../../../README.md) and the relevant references before editing.
  Preserve unrelated changes and work in an isolated checkout or worktree.
- Check current remote state, the related issue and its milestone. Use the
  user's requested milestone; ask if the release target is unknown.
- Open a pull request against the current intended target branch. For this
  repository's published package, use `main` unless instructed otherwise.
  Do not infer merge or release authorization from a request to edit the skill.
- Preserve the public CLI boundary: verify version and command help before
  documenting capabilities. Check supported flags before enabling newer paths.
  Follow the package's unsupported-operation workflow; do not replace missing
  CLI functionality with HTTP, SDK, MCP or custom CLI extensions.

## Mandatory changelog

Every pull request with repository changes **must** include a substantive
update to [CHANGELOG.md](../../../CHANGELOG.md) in that same pull request.
This applies to skill instructions, references, agent metadata, documentation,
tooling and this maintainer skill itself.

- Add an entry under `Unreleased` using the existing format. Describe the
  delivered behavior or correction and link the related GitHub issue.
- Describe the final scope of this pull request. An existing entry in the base
  branch, whitespace edits or merely mentioning the changelog in the PR body
  do not satisfy this requirement.
- Keep skill changes, CLI implementation, binary verification, portal checks
  and published releases distinct. Do not turn an RC or a closed issue into a
  stable-release claim. Retrospective entries must state their actual scope.
- Before opening the pull request **and immediately before merge**, fetch the
  current target branch and inspect its diff. For a PR targeting `main`, run:

  ```sh
  git fetch origin
  git diff origin/main...HEAD -- CHANGELOG.md
  ```

  Substitute the actual target branch when it differs. Confirm a meaningful
  entry for this PR's changes and issue. Repeat after new commits, rebases or
  conflict resolution. Check the proposed merge result still contains it.
- If the entry is missing or inaccurate, fix it and repeat verification.
  Do not declare the PR ready or merge it, even if other checks pass.

## Verification and reporting

- Validate changed skill frontmatter and all added local Markdown links.
  Check `git diff --check` and inspect the final diff, including new files.
- For capability changes, verify examples against the relevant CLI binary's
  help and public interface. Report exact version/source SHA and what ran.
- Record relevant evidence in [verification documentation](../../../docs/verification.md)
  without overwriting older receipts or expanding their proof boundaries.
- Report local validation, PR state, merge and release separately. After GitHub
  mutations, read back the issue/PR state and milestone before claiming success.
