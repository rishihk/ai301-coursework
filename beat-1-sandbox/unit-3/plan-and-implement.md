# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

---

## Posted upstream

**GitHub username**

rishihk

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51#issuecomment-6007025125

Plan for this issue, built from my reproduction above (https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51#issuecomment-5881199893).

**What I found:** no CI job runs `alembic upgrade head` or `alembic check` (`grep -nE "alembic|migration" .github/workflows/ci.yml` gives no match), and `main` already has drift: `alembic check` reports `Detected removed unique constraint 'uq_users_email' on 'users'`. So a new check would be red the first time it runs.

**What I plan to change (one PR):**
1. Add `scripts/validate_migrations.sh`: `alembic upgrade head` on an empty database, then `alembic check`. It rewrites `postgresql://` to `postgresql+asyncpg://`, because `alembic/env.py` uses the async engine.
2. Add a `migrations` job to `.github/workflows/ci.yml` with a fresh `postgres:16-alpine` service that runs the script.
3. Add migration `003` that drops the duplicate `uq_users_email` constraint, so the job starts green.

**Not in this PR:** model changes, edits to `001`/`002`, the existing five jobs, branch protection settings, and any other drift `alembic check` finds (I'll report that here as separate work).

**How I'll test it:** I'll re-run my reproduction. `alembic check` should go from FAILED to exit 0, and a throwaway broken migration should make the script exit non-zero with the `UndefinedColumnError`. I'll then run the workflow on my fork and check that all six jobs pass.

**What I'm unsure about:** I haven't yet confirmed that `ix_users_email` is unique in the migrated database. If it isn't, dropping the constraint would be wrong and I'll change the plan before building. I also tested on Python 3.12 and CI uses 3.11.

I've included the small drift fix in this pull request so the new job isn't red the first time it runs. If you'd rather review it separately, I'm happy to split it out.

---

## Your branch

**Branch**

feat/51-ci-migration-check

**Evidence**

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51. Base commit f89c06fc3ff292df2a04a39ac51319d32a76b779. These are my unit 2 reproduction steps (posted at https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51#issuecomment-5881199893), run before and after the change.

BEFORE (output I posted in unit 2):

```
$ .venv/Scripts/alembic downgrade base
$ .venv/Scripts/alembic upgrade head
INFO  [alembic.runtime.migration] Running upgrade  -> 001, Initial schema creation for users, profiles, ingested_sources, and reviews.
INFO  [alembic.runtime.migration] Running upgrade 001 -> 002, Add error_message column to reviews table.
$ .venv/Scripts/alembic current
002 (head)

$ .venv/Scripts/alembic check
INFO  [alembic.autogenerate.compare.constraints] Detected removed unique constraint 'uq_users_email' on 'users'
ERROR [alembic.util.messaging] New upgrade operations detected: [('remove_constraint', UniqueConstraint(Column('email', NullType(), table=<users>)))]
FAILED: New upgrade operations detected: [('remove_constraint', UniqueConstraint(Column('email', NullType(), table=<users>)))]

$ .venv/Scripts/alembic upgrade head    (with a throwaway 003_broken_demo.py)
asyncpg.exceptions.UndefinedColumnError: column "nonexistent_column" of relation "reviews" does not exist
sqlalchemy.exc.ProgrammingError: (sqlalchemy.dialects.postgresql.asyncpg.ProgrammingError) column "nonexistent_column" of relation "reviews" does not exist
[SQL: ALTER TABLE reviews DROP COLUMN nonexistent_column]

$ grep -nE "alembic|migration" .github/workflows/ci.yml || echo "no match"
no match
$ ls scripts/validate_migrations.sh
ls: cannot access 'scripts/validate_migrations.sh': No such file or directory
```

AFTER (the same steps against my branch, saved with tee):

```
$ alembic downgrade base && alembic upgrade head && alembic current
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running downgrade 003 -> 002, Drop duplicate unique constraint uq_users_email from users table.
INFO  [alembic.runtime.migration] Running downgrade 002 -> 001, Add error_message column to reviews table.
INFO  [alembic.runtime.migration] Running downgrade 001 -> , Initial schema creation for users, profiles, ingested_sources, and reviews.
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 001, Initial schema creation for users, profiles, ingested_sources, and reviews.
INFO  [alembic.runtime.migration] Running upgrade 001 -> 002, Add error_message column to reviews table.
INFO  [alembic.runtime.migration] Running upgrade 002 -> 003, Drop duplicate unique constraint uq_users_email from users table.
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
003 (head)
$ alembic check
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.schemas
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.tables
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.types
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.constraints
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.defaults
INFO  [alembic.runtime.plugins] setting up autogenerate plugin alembic.autogenerate.comments
No new upgrade operations detected.
exit=0
$ throwaway broken migration + validate_migrations.sh
INFO  [alembic.runtime.migration] Running downgrade 001 -> , Initial schema creation for users, profiles, ingested_sources, and reviews.
    super()._handle_exception(error)
  File "C:\dev\Codepath\pathreview-ai301-fa26-s1\.venv\Lib\site-packages\sqlalchemy\connectors\asyncio.py", line 412, in _handle_exception
    self._handle_exception_no_connection(self.dbapi, error)
  File "C:\dev\Codepath\pathreview-ai301-fa26-s1\.venv\Lib\site-packages\sqlalchemy\dialects\postgresql\asyncpg.py", line 840, in _handle_exception_no_connection
    raise translated_error from error
sqlalchemy.exc.ProgrammingError: (sqlalchemy.dialects.postgresql.asyncpg.ProgrammingError) column "nonexistent_column" of relation "reviews" does not exist
[SQL: ALTER TABLE reviews DROP COLUMN nonexistent_column]
(Background on this error at: https://sqlalche.me/e/21/f405)
exit=1
$ grep -nE "alembic|migration" .github/workflows/ci.yml
100:  migrations:
108:          POSTGRES_DB: pathreview_migrations
122:      - name: Validate migrations
123:        run: bash scripts/validate_migrations.sh
125:          DATABASE_URL: postgresql://pathreview:pathreview@localhost:5432/pathreview_migrations
scripts/validate_migrations.sh*
```

Continuous integration on the branch (run from a pull request inside my own fork, closed without merging): all six jobs passed (lint, typecheck, test-unit, test-integration, frontend, migrations). https://github.com/rishihk/pathreview-ai301-fa26-s1/actions/runs/37398674332

---

## Eval iterations

**Run history**

1. `--only pkg-01,pkg-02,pkg-04,pkg-06,pkg-10` (one package per category, as a cheap test): 5/5.
2. Full run, 20 scored packages, `--save-run eval-run.txt`: 20/20 (bar 18/20: PASS). Categories: clear-accept 7/7, scope-creep 4/4, thread-convention 2/2, unbuildable 3/3, wrong-cause 4/4. This is the committed eval-run.txt.

I did not change any skill file between run 1 and run 2. (After run 2 I re-graded pkg-06 alone, without saving, only to read its per-check grades for the next field. It is not a scoring run, and eval-run.txt was not touched.)

**Package analysis**

pkg-06 (kubernetes/minikube#21408, "containerd: preloaded images fail to save"). Gold label: reject (scope-creep). My rubric's verdict: reject.

The repro is small: saving a preloaded image gives an empty tar and exit 0, while a pulled image saves fine. The plan turns that into five projects: regenerate the preload tarballs, upgrade containerd "for every runtime and driver combination, since we are on an old patch series anyway", add a "unified image-operations abstraction" across docker, containerd, and cri-o, surface export errors, and add a CI matrix. The comment itself says "It is a bigger change than the one-line symptom fix".

My rubric failed it on four checks. scope-bounded failed because of the containerd upgrade, the abstraction and the CI matrix, none of which the issue asked for. executable failed because the files are broad areas ("pkg/minikube/cruntime/ (all three runtime implementations)") with no decided approach. test-is-decisive failed because the CI matrix checks docker and cri-o behavior the repro never showed, and nothing checks the loud-failure change. unknowns-honest (preferred, so it does not change the verdict) failed because the comment says it "traced it to the preload structure mismatch" while the issue only calls that a suspicion. cause-fits-repro and thread-and-conventions passed.

Verdict rule: accept only if all five required checks pass, so any one of the three required failures is enough to reject.

**Check rationale**

The check I am quoting is scope-bounded, exactly as it reads in my uploaded rubric.md:

```
| scope-bounded | The plan's in-scope and not-in-scope statements and its list of files, read against what the issue asks for. | Pass if the plan is one change a reviewer could review as a single pull request, it does what the issue asks, and anything else it spots (cleanups, refactors, extra features) is left out or named as separate work. Fail if it adds work the issue did not ask for, or names no limits at all. | required |
```

It reads this way because scope-creep was the failure family where the write-up looks fine but the work is too big. So the pass condition judges the outcome, whether it is one change a reviewer could review as a single pull request that does what the issue asks. It does not count sections or lines. I added "or named as separate work" so a plan that notices extra problems and sets them aside still passes, and only a plan that does the extra work fails. I rejected a size-based rule (a limit on files or lines), because a correct fix can touch several files and a short plan can still be a rewrite. I did not revise this check after any run, since it agreed with the answer key on all four scope-creep packages the first time.

**Trade-offs**

The check gives up cases where the right change is bigger than what the issue literally says. My own plan is an example. The drift fix (migration 003) is work issue #51 did not literally ask for. When I graded my plan, the skill passed it only as a judgment call, because the plan explained that the new check cannot pass on main without the fix and offered to split it out. The pass condition does not say how to treat a change like that, so a stricter grader could fail it. I accept that cost, because tightening it to "only what the issue literally names" would also reject the sensible plans.

pkg-06 is also not held by this check alone. It also fails executable and test-is-decisive, so loosening scope-bounded later would not flip it. I did not edit any check after either run, so I did not need canary packages. Both runs used the same files (the hashes in the eval-run.txt header show them), and the full run matched the answer key on all 20 packages.

One kind of problem this rubric cannot see: it has no check for the house branch-name rule (`type/issue-number-description`). The plan-check run on my own plan noticed that gap, and my rubric would pass a plan that names the wrong branch.
