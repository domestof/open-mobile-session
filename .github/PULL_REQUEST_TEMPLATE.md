## What does this change fix or add?

## Why doesn't the current checklist already handle this?

## Manual testing

This skill drives real processes and a real sandbox — there's no automated
test suite. Describe what you ran and what you saw.

## Safety checklist

- [ ] Every step still stops rather than guesses (fails loudly, doesn't
      silently work around unexpected state).
- [ ] No new flag or command was added without checking it against
      `claude remote-control --help` or code.claude.com/docs.
- [ ] The safety rules are intact: no `sudo`, nothing destructive without
      asking first, stays inside the target folder. If this PR touches one
      of them, I've explained why above.
