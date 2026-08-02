# open-mobile-session

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/domestof/open-mobile-session?style=social)](https://github.com/domestof/open-mobile-session/stargazers)

A [Claude Code](https://code.claude.com) skill that lets you, from your
phone, quickly spin up a brand-new sandboxed workspace on a real machine
with its own shell and filesystem — wherever you run Claude Code (a
laptop, a home server, your own cloud instance) — instead of the
browser-based claude.ai chat.

![Secure mobile coding: one command away](docs/secure-mobile-coding.png)

## The problem

[Claude Code Remote Control](https://code.claude.com/docs/en/remote-control)
lets you drive a coding session from your phone. But starting one the safe
way means running a long, security-sensitive checklist by hand every single
time: check your Claude Code version, confirm the folder is trusted, write
a sandbox policy, prove the sandbox actually blocks things it should block,
launch the session, and verify it came up correctly. Skip a step — or get
one wrong — and you could end up with a session that has full access to
your machine instead of just the project folder you meant to hand it.

## When it's useful

Any time you want to kick off a Claude Code session — in a brand-new folder
or an existing project — that you can then continue from your phone,
without babysitting a multi-step setup or trusting yourself not to miss a
safety check. You run one command; the skill runs the checklist for you and
refuses to continue if anything looks wrong.

## What it does

It automates the checklist you'd otherwise run by hand every time:

- Check your Claude Code version supports the sandbox credentials policy.
- Confirm the target folder is trusted (or ask you to trust it yourself).
- Write a `.claude/settings.json` sandbox policy that denies access to
  `~/.ssh`, `~/.aws/credentials`, and Claude's own credentials file.
- Prove the sandbox is actually enforced with a probe that must fail
  (`touch ~/sandbox-probe` should be `BLOCKED`, never `LEAK`).
- Launch `claude remote-control --sandbox` detached, in the target folder.
- Poll until the session reports `Connected`, then verify the process is
  alive and its working directory matches the target folder.

Every step is designed to stop rather than guess: wrong Claude Code version,
untrusted folder, sandbox not enforced, or an ambiguous log line all halt
the checklist instead of silently continuing.

The exact instructions Claude follows are in
[`skills/open-mobile-session/SKILL.md`](skills/open-mobile-session/SKILL.md)
— nothing hidden, it's the whole prompt.

## Installation

Copy the skill into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/open-mobile-session
cp skills/open-mobile-session/SKILL.md ~/.claude/skills/open-mobile-session/SKILL.md
```

Or, to enable it for a single project only, copy it into that project's
`.claude/skills/` directory instead.

## Usage

Invoke it from within a Claude Code session:

```
/open-mobile-session /home/user/github/some-project my-session-name
```

Both arguments are optional — if you omit them, the skill asks for a path
and a session name before doing anything. It refuses relative paths, `~`,
or `$HOME`, and refuses names containing quotes or shell metacharacters.

Once launched, open the Claude app on your phone and find the session
under the name you gave it.

To stop it: `kill <PID>` (never `pkill -f` with the session name — it can
match your own shell and take it down too).

## Prerequisites

- Claude Code 2.1.187 or newer, signed in with a Pro/Max/Team/Enterprise
  plan.
- None of `DISABLE_TELEMETRY`, `DO_NOT_TRACK`,
  `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`, or `DISABLE_GROWTHBOOK` set,
  and no `ANTHROPIC_BASE_URL` pointing off the Anthropic API.
- macOS, Linux, or WSL2 — native Windows has no sandbox.
- On Linux: `sudo apt-get install bubblewrap socat`, and optionally
  `npm install -g @anthropic-ai/sandbox-runtime` for the seccomp filter
  (blocks Unix sockets, including `docker.sock`). Restart Claude Code after
  installing.
- On Ubuntu 24.04+, if
  `sysctl kernel.apparmor_restrict_unprivileged_userns` returns `1`, you
  need the `bwrap` AppArmor profile — see the skill file for the exact
  profile to install.

## Safety model

The skill is deliberately conservative about what it will do on your
behalf:

- It stays inside the target folder — nothing outside it, not even to
  read it.
- The only read-only exceptions are Claude Code's own configuration
  (never written to), checking for the sandbox probe file, and reading
  the process table / working directory to verify the launch succeeded.
- Nothing destructive (`rm -rf`, disk operations, force-push,
  uninstalling, stopping services) without asking first and stating
  exactly what will happen. No `sudo`.
- If the sandbox probe doesn't come back `BLOCKED`, the skill stops and
  tells you what's missing — it will not edit configuration to work
  around an unenforced sandbox.

## Further reading

The same "stop rather than guess" discipline behind this skill's checklist
is the subject of *[The Fluency Trap](https://thefluencytrap.com)*, a
business novel by Jordi Clopés. It tells a story to explain Spec-Driven
Development and the problem with mistaking an AI's fluency for
correctness — worth a read if this skill's approach resonates with you.

## Contributing

Issues and pull requests are welcome — see
[CONTRIBUTING.md](CONTRIBUTING.md) for what to include. Participation in
this project is governed by the [Code of Conduct](CODE_OF_CONDUCT.md).

## License

MIT — see [LICENSE](LICENSE).

---

If this saved you from writing that checklist by hand, consider giving the
repo a ⭐ — it helps other people find it.
