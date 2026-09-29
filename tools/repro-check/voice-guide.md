# Voice guide: how I talk upstream

## Who I am in threads

I'm Rishi, a data engineer and CodePath AI301 student making my first
contributions to Path Review. When I comment, I'm reporting what I ran
and what I saw, from my own machine. Readers can expect exact commands
and real output from me, not guesses dressed up as findings.

## Rules I write by

### Rule: promise the next step, not the fix

I only promise the investigation or report I'm about to do. No fixes,
PRs, or dates until the work exists.

- Wrong: "I'll have a PR up by Friday that adds the migration check."
- Right: "I'll reproduce the gap on a fresh database and post what I find here first."

### Rule: name this issue's specifics

Every comment names something only true of this issue: the file, the
command, or the symptom. If it could be pasted on any issue, rewrite it.

- Wrong: "Hi! I'd love to work on this, please assign it to me."
- Right: "I'd like to work on the missing migration check in `.github/workflows/ci.yml`."

### Rule: say what I saw, not what I think it means

Stated outcomes match the pasted output. Guesses about cause are
labeled as guesses.

- Wrong: "This is definitely broken and CI is totally unsafe."
- Right: "`grep` finds no alembic step in ci.yml; output below."

### Rule: flag differences from the issue

If my version, OS, or setup differs from the issue's, I say so.

- Wrong: (silently testing on a different branch)
- Right: "Tested on my fork at commit `<sha>`, Python 3.11 on macOS 26."

## Things I never post

- A fix date or "should be quick" estimate.
- "Same as above, can confirm" or any repro I didn't run myself.
- Words like "guaranteed", "definitely", or "rigorous" about my own work.
- A comment I haven't run through repro-check first.
