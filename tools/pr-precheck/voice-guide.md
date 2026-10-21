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


## Plan comments (added in unit 3)

- A plan is a proposal, not a promise. Write "I plan to..." and "I'd like to...", not "this will fix..." until the test has run.
- Name one thing I am unsure about, and say how I will find out. Wrong: "This should be straightforward." Right: "I have not yet checked whether the migrated database has other differences after this one is fixed; I'll run alembic check again and report it."
- If a maintainer or another contributor already suggested a direction, say whether I am following it and why, in one sentence.
- If my plan differs from another plan on the same issue, say what differs. Do not write "same approach as above".
- Never give dates or timelines for when it will be done.
- Never post a plan comment that I did not run through plan-check first.

## PR title and description (added in unit 4)

- The title says what the change does to this specific thing, in plain words. Wrong: "Fix issue" or "Updates". Right: a title that names the file, command, or behavior I changed.
- The description claims only what the diff contains. Never write "exactly as planned", "no other changes", or "docs updated" unless I checked the diff for it. Wrong: "Implements the plan exactly." Right: "Implements the plan's step 1 and 2; step 3 is left out, see Notes for Reviewers."
- Paste real command output in Testing, not a summary. Wrong: "tests pass". Right: the command and its result lines.
- Tick only the checkbox I actually ran. The CI box stays unticked.
- Say in my own words how I used AI on this change, under Notes for Reviewers.
- If I changed or left out something from my plan, say so in the description and in plan.md under Deviations.
- Never open a PR I did not run through pr-precheck first.
