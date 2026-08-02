# Contributing to open-mobile-session

Thanks for considering a contribution. This project is small on purpose —
one skill file plus its documentation — so the bar for changes is mostly
about keeping the checklist safe and correct, not adding features.

## Reporting a problem

Open an [issue](https://github.com/domestof/open-mobile-session/issues)
and include:

- Your Claude Code version (`claude --version`)
- Your OS (macOS, Linux distro, or WSL2)
- Whether the failure happened at a specific step in the checklist, and
  what the skill printed at that point
- Whether `bubblewrap`/`socat` (and, if installed,
  `@anthropic-ai/sandbox-runtime`) are present

## Proposing a change

1. Fork the repo and create a branch off `main`.
2. Make your change to `skills/open-mobile-session/SKILL.md` and/or the
   docs.
3. Open a pull request describing:
   - What problem the change fixes or what it adds
   - Why the current checklist doesn't already handle it
   - Any manual testing you did (this skill drives real processes and a
     real sandbox, so there's no automated test suite — describe what you
     ran and what you saw)

## Guidelines specific to this skill

- Every step in the checklist is meant to **stop rather than guess**. If
  you add a step, keep that property: fail loudly and explain why, don't
  silently work around an unexpected state.
- Don't add flags or commands that aren't checkable against
  `claude remote-control --help` or the docs at code.claude.com — see the
  note at the bottom of the skill file.
- Keep the safety rules (no `sudo`, nothing destructive without asking,
  stay inside the target folder) intact. If a change would weaken one of
  them, explain why in the PR description so it can be discussed
  explicitly rather than merged by accident.

## Code of Conduct

Participation in this project is governed by the
[Code of Conduct](CODE_OF_CONDUCT.md).
