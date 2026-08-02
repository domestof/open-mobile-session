# Security Policy

This project is a single Claude Code skill that writes a sandbox policy,
launches a detached process, and grants it Remote Control access to a
folder on your machine. Given what it touches, security reports are taken
seriously.

## Supported versions

There's no versioning beyond the `main` branch — it's the only branch, and
the only one that gets fixes.

## Reporting a vulnerability

Please do **not** open a public issue for a security problem — a public
issue is a working exploit disclosure. Instead, use one of:

- GitHub's [private vulnerability reporting](https://github.com/domestof/open-mobile-session/security/advisories/new)
  for this repository, or
- Email jordi@thefluencytrap.com with details and, if possible, how to
  reproduce it.

Please include:

- What the checklist did wrong, and what it should have done instead
- Whether it involves the sandbox policy itself, the probe step, or
  something outside the skill's stated scope (e.g. a bug in Claude Code or
  the sandbox runtime, which should also be reported to Anthropic)
- Your Claude Code version and OS

You'll get an acknowledgment as soon as possible, and credit in the fix
unless you'd rather stay anonymous.
