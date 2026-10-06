# Procedure: how this skill grades a plan package

## Read order

1. Read the issue context first. Note in one line what the issue asks for. Every later check compares the plan against this line, so read it before you see the plan.
2. Read the repro evidence block second. Note the commands that were run, the exact output, and the one behavior the output proves. Do not read the plan yet, so the evidence is not bent to fit it.
3. Read the repo-facts block third. Note the branch, commit, and comment rules, and any AI-use or disclosure rule, and whether that rule covers comments or only pull requests.
4. Read the thread highlights fourth. Note every direction a maintainer gave (an approach to use, an approach to avoid, a question left open).
5. Read the candidate plan fifth. Note its stated cause, its in-scope and not-in-scope lines, its files and approach, its test plan, and its risks and unknowns.
6. Read the plan comment last. Note what it commits to, and whether it mentions the thread, the repro, and any disclosure.

## Evidence gathering

1. For cause-fits-repro: copy the plan's one-sentence cause, then copy the line of repro output that shows the behavior. Record whether the cause explains that line.
2. For scope-bounded: copy the plan's in-scope and not-in-scope statements and the list of files. Compare each item against the one-line ask from the issue. Mark each item as asked-for or extra.
3. For executable: list each file or area the plan names and what it will do there. Mark any file, tool, or access that the repro evidence or repo-facts block says is missing or not allowed.
4. For test-is-decisive: copy the plan's test steps and the result each one expects. Match each step to a repro step or artifact. Mark whether a failing result before the change and a passing result after the change are both named.
5. For thread-and-conventions: put the maintainers' signals from the thread next to the plan comment, and put the repo's stated rules next to the comment. Mark each signal and rule as followed, broken, or not applicable.
6. For unknowns-honest: list every claim of fact in the plan, and mark whether the repro or a quote backs it. List every unknown the plan states.
7. If the part of the package a check needs is not there, write "absent" for that check's evidence. Do not invent it from other parts.

## Check execution

1. Run the checks in this order: cause-fits-repro, scope-bounded, executable, test-is-decisive, thread-and-conventions, unknowns-honest.
2. Grade each check only on its own gathered evidence and its own pass condition. Do not let one check's result change another's. A check may be graded from the notes in Evidence gathering without re-reading the whole package, but go back to the source text if the notes are not enough to quote.
3. Grade pass if the pass condition is clearly met, fail if the fail condition is clearly met.
4. Grade unclear only if the evidence is present but you cannot tell which way it goes after re-reading the relevant part once. Never grade unclear because the plan is long or hard to read.
5. If the evidence for a required check is absent, grade it fail, and say what was missing.
6. Judge outcomes, not shape. Do not fail a plan for its length, its headings, or its tone. Fail it only for what the pass condition names.
7. For every grade, write one sentence that quotes the plan or the evidence that decided it.

## Verdict assembly

1. List the six grades.
2. Apply the verdict rule: accept only if all five required checks (cause-fits-repro, scope-bounded, executable, test-is-decisive, thread-and-conventions) are pass. A preferred check (unknowns-honest) never changes the verdict. An unclear on a required check counts as fail.
3. If the verdict is accept, say so, and quote the plan's cause sentence and its test step as the reason.
4. If the verdict is reject, name the first failed required check in the order above and quote the exact text of the plan, comment, or evidence that made it fail. List any other failed required checks by name only.
5. Output the verdict (accept or reject) in the JSON block, with the grade of every check, and nothing else after it.
