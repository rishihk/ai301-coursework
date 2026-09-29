# Evidence guide: where proof lives in a reproduction package

## Environment

- Where it lives: eval bundle: the repro report's first block (often
  "Environment:"), read against the issue body's stated version, OS,
  install method, and any platform the thread says matters. Live mode:
  the draft report's environment block, read against the issue on
  GitHub and the repo's setup docs (README, docker-compose.yml,
  pyproject.toml).
- What good looks like: one line or block naming the project version or
  commit, the OS, and every runtime, driver, or build setting the issue
  says changes the behavior. If that differs from the issue's target
  (newer release, older version, other OS), the report says so in words.
  A missing environment record, or a silent version or platform
  deviation, is not sufficient however good the rest is.

## Steps

- Where it lives: eval bundle: the repro report's steps or commands,
  from install or clone to the trigger, plus wherever each input file
  comes from. Live mode: the same in the draft report.
- What good looks like: a stranger with only public access can copy the
  commands in order and reach the trigger. Inputs are pasted inline or
  come from a public source. Steps may be terse. Not followable: a
  private repo or monorepo, an unshared config, or a step like "set up
  the project" with no command.
- Trigger: identify the element that actually causes the bug (the
  issue's specific input, syntax, operator, flag, or a maintainer's
  note on what triggers it) and check the report kept it. Harmless
  mechanics (http vs https, --offline, -v, a local file, a different
  install path) are not trigger changes. A silent change to the
  triggering element means the report tested something else.

## Behavior shown

- Where it lives: eval bundle: the repro report's fenced output blocks
  (terminal output, traces, exit codes, logs, measurements), read
  against the issue body's described symptom and any maintainer note in
  the thread highlights. Live mode: the same in the draft, against the
  issue on GitHub.
- What good looks like: the artifact shows the issue's own symptom:
  same error type and message, same exit behavior (e.g. a panic/crash vs
  a graceful error), same observable effect. A control run (the same
  command without the trigger) strengthens it. A graceful validation
  error, a syntax error, a compile error, garbled output with the
  program still alive, or a banner showing the program runs is NOT the
  issue's behavior if the issue describes a crash, panic, or blank
  state.
- For an issue describing a gap (a missing CI step, check, or script),
  the behavior is the gap: artifacts show the missing piece is absent
  (e.g. a grep of the workflow file, a missing path) and that the
  failure it should catch actually happens when triggered.
- An honest cannot-reproduce also counts: the artifacts show the real
  attempt's outcome, not a different bug.

## Honesty

- Where it lives: the repro report's conclusion, analysis, and
  "expected / actual" lines, and the claim comment, read against the
  artifacts above.
- What good looks like: the headline conclusion (reproduced, not
  reproduced, cause) matches the artifacts shown. Root causes are
  labeled as hypotheses. A cannot-reproduce names what was tried and
  what differed from the reporter's setup. Brief side notes, like a
  one-line control-run observation, are fine and are not the claim
  being judged. Red flags: "exactly the failure the issue describes"
  over a different error, "guaranteed reproducible", "I verified the
  race", or "confirmed on two machines" with nothing shown, or
  generalizing to versions or platforms not tested.

## Comms

- Where it lives: eval bundle: the claim comment, and the repo-facts
  block's "bug reports" and "contribution policy" lines (including any
  AI-use policy). Live mode: the draft claim, the repo's CONTRIBUTING /
  AI policy files on GitHub, and scope.md house rules.
- What good looks like (claim): names this issue's specifics (symptom,
  file, trigger) and the next step; promises investigation or a report,
  never a fix, PR, or date. Interchangeable "please assign me, I'll fix
  it in 2 days" boilerplate or a bare +1 is a fail.
- What good looks like (AI policy): treat every package as AI-assisted.
  If the policy requires disclosing AI use for issues or comments, one
  of the comments must actually disclose it (tool and extent); silence
  does not count. If the policy only asks for disclosure in pull
  requests, only asks that comments be written in the contributor's own
  words, or states no AI policy, do not invent a disclosure requirement;
  fail only if the comments visibly break a stated rule.
