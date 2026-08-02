---
name: open-mobile-session
description: |
  Use when the user asks to open, spawn or start a new sandboxed Claude Code
  session drivable from their phone via Remote Control — in a brand-new
  folder or in an existing project folder. Triggers: "obre'm una sessió pel
  mòbil", "engega una sessió sandboxejada", "spawn a mobile session", "vull
  treballar en aquest projecte des del mòbil". Replaces pasting the long
  "One Claude Code Session Can Start Another" prompt: this skill runs the
  same checklist directly — version gate, trust check, sandbox policy,
  verification probe that must fail, launch, poll, final PID/cwd check.
---

# Open a sandboxed Remote Control session

Run this checklist yourself, in order, in the session where this skill was
invoked. Stop at the first step that fails — never skip a step to "save
time".

**Arguments.** `$ARGUMENTS` may contain an absolute path and, optionally, a
session name, e.g. `/home/user/github/my-project my-project-mobile`.

- If no path was given, ask for it. Refuse a relative path, `~`, or `$HOME` —
  ask again instead.
- If no name was given, ask for it (a good default is the folder's
  basename, but confirm before using it). Refuse a name containing quotes or
  shell metacharacters — ask again instead.

**Rules, no exceptions.** Stay inside that path: nothing outside it, not even
to read it. Three narrow exceptions, all read-only: you may read Claude
Code's own configuration (never write to it), you may check for the
existence of `~/sandbox-probe` in step 5, and you may read the process table
and the server's working directory in step 7. Nothing destructive without
asking first and stating exactly what will happen: `rm -rf`, disks,
force-push, uninstalling, stopping services. No `sudo`. Keep away from
system config, systemd, and other people's files. If anything is unclear or
can't be undone, stop and ask.

**Steps:**

1. Run `claude --version`. Below 2.1.187, stop: the credentials part of the
   policy below would be silently ignored.
2. Read the `projects` entries in `~/.claude.json` and confirm that the
   given path, or one of its ancestors, has `hasTrustDialogAccepted: true`.
   If none does, stop: the user has to run `claude` there once at a keyboard
   and accept the trust dialog themself. Do not edit that file.
3. Check the folder:
   - If it doesn't exist, create it.
   - If it exists and is empty, continue.
   - If it exists and has content, show what's there. If it already has a
     `.claude/settings.json` with `sandbox.enabled: true`, treat the policy
     as already provisioned and skip straight to step 5 — don't overwrite a
     working config. Otherwise, stop and ask for confirmation before writing
     anything: mention explicitly that the policy below applies to *every*
     future session in that folder, not just this one.
4. Write `.claude/settings.json` with exactly this policy:
   ```json
   {
     "permissions": { "ask": ["Bash(docker *)"] },
     "sandbox": {
       "enabled": true,
       "failIfUnavailable": true,
       "allowUnsandboxedCommands": false,
       "excludedCommands": ["docker *"],
       "credentials": { "files": [
         { "path": "~/.ssh", "mode": "deny" },
         { "path": "~/.aws/credentials", "mode": "deny" },
         { "path": "~/.claude/.credentials.json", "mode": "deny" }
       ]}
     }
   }
   ```
5. Check the sandbox is real with a probe that has to fail. Inside the
   folder, run:
   `claude -p 'Run this and report the literal output: touch ~/sandbox-probe && echo LEAK || echo BLOCKED'`
   You need `BLOCKED`, and no file at `~/sandbox-probe`. Anything else means
   nothing is being enforced: stop and say which package is missing (see
   Prerequisites below). Don't edit configuration to work around it.
6. `cd` into the folder and start the server there, detached, substituting
   the confirmed name for NAME:
   `nohup claude remote-control --sandbox --permission-mode default --name "NAME" > rc.log 2>&1 < /dev/null &`
   `--sandbox` is documented at code.claude.com/docs/en/sandboxing but
   missing from `claude remote-control --help` on 2.1.220 — the parser
   accepts it regardless. Do not drop it.
7. Poll `rc.log` for up to 60 seconds until it contains `Connected`. An empty
   log is a failure, not a pass. If it says `Workspace not trusted`, stop:
   the user has to accept the folder themself at a keyboard. If it never
   says `Connected`, stop and show the last lines of the log.
8. Only then: confirm with `ps` (using the real `claude remote-control`
   PID, not a wrapping shell PID) that the process is alive, confirm its
   working directory matches the folder (Linux: `ls -l /proc/<PID>/cwd`;
   macOS: `lsof -a -p <PID> -d cwd`), and report the PID and the URL from
   the log.

Don't invent flags. Every flag above is checkable: `--name` and
`--permission-mode` are in `claude remote-control --help`; `--sandbox` is at
the sandboxing doc URL in step 6. If adding anything else, check it against
code.claude.com/docs first.

**Prerequisites** (only relevant if step 5 fails):

- Claude Code 2.1.187+, `claude` signed in with a Pro/Max/Team/Enterprise
  plan, none of `DISABLE_TELEMETRY` / `DO_NOT_TRACK` /
  `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` / `DISABLE_GROWTHBOOK` set, and
  no `ANTHROPIC_BASE_URL` pointing off the Anthropic API.
- macOS, Linux or WSL2 — native Windows has no sandbox.
- On Linux: `sudo apt-get install bubblewrap socat`, optionally
  `npm install -g @anthropic-ai/sandbox-runtime` for the seccomp filter
  (blocks Unix sockets, including `docker.sock`); restart Claude Code after
  installing.
- On Ubuntu 24.04+: if `sysctl kernel.apparmor_restrict_unprivileged_userns`
  returns `1`, install the `bwrap` AppArmor profile:
  ```bash
  sudo tee /etc/apparmor.d/bwrap <<'EOF'
  abi <abi/4.0>,
  include <tunables/global>
  profile bwrap /usr/bin/bwrap flags=(unconfined) {
    userns,
    include if exists <local/bwrap>
  }
  EOF
  sudo systemctl reload apparmor
  ```

**Success looks like:** probe returns `BLOCKED`; log says `Connected` with a
`https://claude.ai/code?environment=env_…` URL (an *environment* URL, not a
per-session one); PID alive; cwd matches. Tell the user to open the Claude
app and find the session under the name they gave.

**Stopping it:** `kill <PID>`. Never `pkill -f` with the session name — the
pattern can match the user's own shell and take it down with it.

This checklist is distilled from the author's own notes on safely bridging
Claude Code's sandbox and Remote Control features, with the full rationale
for every flag and the sandbox's actual coverage — consult
code.claude.com/docs if something here needs more context than this skill
gives.
