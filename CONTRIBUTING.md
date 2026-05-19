# Contributing

Quick guide for working in this repo. Read once at the start of the summer.

## Branches

- `main` is protected. No direct pushes. All changes go through PR.
- Feature branches: `<type>/<initials>-<short-desc>`
  - `feat/mm-rabbitmq-queue`
  - `fix/jh-airtable-rate-limit`
  - `chore/dg-update-deps`
- Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`

## Commits

Short, present-tense, lowercase. "add user table" not "Added user table." Group related changes.

## Pull requests

1. Open against `main` with a clear title and the template filled in.
2. Mark Draft while you're still working. Switch to Ready when you want review.
3. Request review from your project lead first. Muji is the backstop, not the first reviewer.
4. Address feedback in new commits, no force-pushing during review (it loses comment threads).
5. Squash and merge when approved and CI is green.
6. Delete the branch after merge.

## Review expectations

- Project leads: review within 24 hours.
- Muji: architecture and infra changes; up to 48 hours given time zones.
- Reviewers mark suggestions vs. blockers clearly. Ask questions when something's unclear.

## Code style

Match the existing patterns. Run the linter before opening the PR.

AI tooling is welcome and expected. Understand what it generated before you commit it.

## Secrets

Never commit real API keys, tokens, or credentials. `.env.local` is in `.gitignore`. If you accidentally commit a secret, tell Muji immediately so we can rotate the key and rewrite history.
