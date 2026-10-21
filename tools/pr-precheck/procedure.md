# Procedure: how this tool grades a pull request package

In eval mode, every part below is a section of the bundle. In live mode, the same part is read from the file or command that references/evidence-guide.md names for it. The order is the same in both modes.

## Read order

1. Read the issue and its thread first. Note in one line what the issue asks for. Every later comparison starts from this line.
2. Read the repo-facts block second. Note every stated ask for a pull request: each template section, how the issue must be referenced, every artifact the repository requires inside the change (a changelog or release-notes entry, tests, translations), the repository's own check commands, and whether the contribution policy requires disclosure of artificial intelligence use. Read it before the description, so you know what to look for when you get there.
3. Read the plan context third (the accepted plan and the repro evidence it built on; live, plan.md including any Deviations section). Note: the files the plan names, its in-scope and not-in-scope lines, every deliverable it promises (code, tests, docs, configuration), anything it defers on purpose, and each step of its test plan with the output it expects. Read the plan before the diff and the description, so their framing cannot bend your reading of it.
4. Read the candidate description fourth. Note every factual claim it makes about what the change does, contains, or leaves out, every template section present, how it references the issue, any deferral it states, and any sentence about artificial intelligence use.
5. Read the commit list fifth. Note the titles, especially any that say wip, fix, fmt, or misc cleanups.
6. Read the diff sixth, file by file, only now that the plan and the description are in hand. The diff is the thing being graded, so read every hunk, not only the first.
7. Read the test evidence last, so you read it against the plan's test plan and against what the diff actually changed.

## Evidence gathering

1. For diff-matches-plan: list every file in the diff from its `---` and `+++` headers. Mark each file as named by the plan or not named. For each hunk inside a named file, mark it as serving the plan or extra (a new flag or option, a renaming pass, reworded unrelated text, a rewrite of code the plan does not touch). List every deliverable the plan promises and mark each as present in the diff or absent. For every extra or absent item, record whether a deviation note or the description discloses it plainly (it names what was added, left out, or deferred).
2. For description-matches-diff: copy each factual claim from the description. Find the hunk or commit that backs it and mark the claim found, contradicted, or not present. Give extra attention to absolute claims such as "exactly as planned", "no changes outside X", "no functional changes", "the docs now document Y", and "adds a test".
3. For repro-before-after: copy each step of the plan's test plan with its expected result. In the test evidence, find the command, the state of the code it ran on (before the change or after), and the output it printed. Compare the input the evidence used with the input the plan's repro uses (the same file count, the same request type, the same arguments), because a run on a control or unchanged path proves nothing. Mark each step as shown before, shown after, and whether it exercised the path the diff changes. Note whether the test the diff adds checks the fixed behavior itself.
4. For repo-checks-run: list the repository's check commands from the repo-facts block, the plan's test plan, and the template's testing checklist. For each, find it in the test evidence and record the command and its result. Note any bare claim such as "tests pass" that names no command.
5. For diff-clean: scan every added and removed line. Record, with the file, each of: a debug print or log line, commented-out code, an unused or dead function or import, a TODO or work-in-progress marker, a whitespace-only or formatting-only hunk, an import reordering, and lines that are re-printed unchanged. Read the commit titles as supporting evidence only.
6. For template-asks-met: take each ask from step 2 of the read order and mark it met or skipped. Check that each template section has real content and is not blank or placeholder text, that the issue is referenced the way the template asks, and that every artifact the repository requires inside the change appears as a file in the diff headers.
7. For ai-disclosure: find the contribution policy line in the repo-facts block. If it requires disclosure, find the sentence in the description where the author says in their own words how artificial intelligence was used. Record the sentence, or record that it is absent. If the block states no such policy, record that.
8. If the part of the package a check needs is not there, write "absent" for that check's evidence. Do not invent it from other parts, and do not read it from outside the package.

## Check execution

1. Run the checks in this order: diff-matches-plan, description-matches-diff, repro-before-after, repo-checks-run, diff-clean, template-asks-met, ai-disclosure.
2. Grade each check only on its own gathered evidence and its own pass condition. One check's result never changes another's. A good fix does not excuse debris, drift, or a missing ask, and a clean diff does not excuse thin evidence. You may grade from your gathered notes without re-reading the whole package, but go back to the source text when the notes are not enough to quote.
3. Grade pass if the pass condition is clearly met. Grade fail if the fail condition is clearly met.
4. A shortfall the plan, its deviation notes, or the description plainly discloses (it names what was added, left out, or deferred, and why) does not fail diff-matches-plan, description-matches-diff, or repro-before-after by itself. A shortfall that is only visible in the diff, or a description that claims more than the diff holds, fails.
5. Grade unclear only if the evidence is present but you cannot tell which way it goes after re-reading the relevant part once. Never grade unclear because the package is long or hard to read.
6. If the evidence for a required check is absent, grade it fail and say what was missing.
7. Judge outcomes, not shape. Do not fail a package for its length, its headings, or its tone. Do not pass one for its confidence, its checklist, or its polish. Grade what the pass condition names.
8. For every grade, write one line that quotes the diff, the description, the plan, or the evidence that decided it.

## Verdict assembly

1. List the seven grades.
2. Apply the verdict rule: accept only if all seven required checks are pass. An unclear on a required check counts as fail.
3. If the verdict is accept, say so, and quote the plan's test step and the matching line of test evidence as the reason.
4. If the verdict is reject, name the first failed required check in the order of the rubric table and quote the exact text of the diff, description, plan, or evidence that made it fail. List any other failed checks by name only.
5. Output the verdict (accept or reject) in the fenced data block, with the grade of every check, and nothing after it.
