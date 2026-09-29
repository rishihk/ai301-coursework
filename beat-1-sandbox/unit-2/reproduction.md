# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

---

## Your identity upstream

**GitHub username**

rishihk

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51#issuecomment-5880963092

I'm picking this up. Before changing anything, I'll reproduce the gap on a clean clone of my fork: apply migrations `001` and `002` to an empty Postgres from the repo's docker compose, check whether the resulting schema matches the models in `core/models/`, and add a throwaway broken migration to confirm nothing in `.github/workflows/ci.yml` catches it. I'll post my environment, commands, and output as my next comment before proposing any fix.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51#issuecomment-5881199893

Reproduction of the gap described in this issue: no CI job applies migrations to a fresh database or compares the result to the SQLAlchemy models, so both a migration that fails on a clean install and existing migration/model drift pass CI today.

**Environment**
- Fork: rishihk/pathreview-ai301-fa26-s1 at commit `f89c06fc3ff292df2a04a39ac51319d32a76b779` (same as upstream `main`, no local changes to committed files)
- OS: Windows 11 Home 10.0.26200 (x86_64), commands run in Git Bash
- Python 3.12.10 (`pyproject.toml` allows `>=3.11`; note CI uses 3.11), alembic 1.20.0, installed with `pip install -e ".[dev]"` into `.venv`
- PostgreSQL 16.15 (`postgres:16-alpine`) via `docker compose up -d db` on port 5433, Docker 29.8.0
- `DATABASE_URL` from `.env.example` (`postgresql+asyncpg://pathreview:pathreview@localhost:5433/pathreview_dev`)

**1. Current migrations apply to an empty database**

```
$ .venv/Scripts/alembic downgrade base
$ .venv/Scripts/alembic upgrade head
INFO  [alembic.runtime.migration] Running upgrade  -> 001, Initial schema creation for users, profiles, ingested_sources, and reviews.
INFO  [alembic.runtime.migration] Running upgrade 001 -> 002, Add error_message column to reviews table.
$ .venv/Scripts/alembic current
002 (head)
```

**2. The migrated schema does not match the models**

The issue asks for a check that the schema matches the SQLAlchemy models. Running alembic's own comparison against the freshly migrated database:

```
$ .venv/Scripts/alembic check
INFO  [alembic.autogenerate.compare.constraints] Detected removed unique constraint 'uq_users_email' on 'users'
ERROR [alembic.util.messaging] New upgrade operations detected: [('remove_constraint', UniqueConstraint(Column('email', NullType(), table=<users>)))]
FAILED: New upgrade operations detected: [('remove_constraint', UniqueConstraint(Column('email', NullType(), table=<users>)))]
```

The migrated database has a unique constraint `uq_users_email` on `users` that the models do not declare. My hypothesis, from reading the code: `001_initial_schema.py` creates both `UniqueConstraint("email", name="uq_users_email")` and a unique index `ix_users_email`, while `core/models/user.py` declares `email` with `unique=True, index=True`, which SQLAlchemy expresses as the unique index only. This drift exists on `main` today and nothing in CI reports it.

**3. A broken migration fails on a fresh database**

Added a throwaway `alembic/versions/003_broken_demo.py` (`revision = "003_broken_demo"`, `down_revision = "002"`) whose upgrade drops a column that does not exist:

```python
def upgrade():
    op.drop_column("reviews", "nonexistent_column")
```

```
$ .venv/Scripts/alembic upgrade head
INFO  [alembic.runtime.migration] Running upgrade 002 -> 003_broken_demo, ...
...
asyncpg.exceptions.UndefinedColumnError: column "nonexistent_column" of relation "reviews" does not exist
sqlalchemy.exc.ProgrammingError: (sqlalchemy.dialects.postgresql.asyncpg.ProgrammingError) column "nonexistent_column" of relation "reviews" does not exist
[SQL: ALTER TABLE reviews DROP COLUMN nonexistent_column]
```

**4. Nothing in CI runs either check**

```
$ grep -nE "alembic|migration" .github/workflows/ci.yml || echo "no match"
no match
$ ls scripts/validate_migrations.sh
ls: cannot access 'scripts/validate_migrations.sh': No such file or directory
```

The CI jobs are lint, typecheck, test-unit, test-integration, and frontend. `test-integration` starts a Postgres service but only runs `pytest tests/integration`; it never runs `alembic upgrade head` or `alembic check`. The script named in the issue does not exist yet.

**Expected:** a PR whose migrations cannot apply to an empty database, or whose migrated schema does not match the models, fails CI.

**Actual:** no CI job applies migrations or compares them to the models. The broken migration in step 3 and the existing `uq_users_email` drift in step 2 would both pass every current CI job.

**Cleanup:** `.venv/Scripts/alembic downgrade base && rm alembic/versions/003_broken_demo.py`; `git status` then reports a clean working tree.

## Eval iterations

**Run history**

1. `--limit 3` (pkg-01, pkg-02, pkg-03): **1/3**. pkg-01 failed `trigger-faithful` (the report used `http --offline` instead of the issue's `https ... -v`) and pkg-03 failed `outcome-honest` (a one-line control-run note without pasted output). Both were false rejects.
   - Revision: `trigger-faithful` now fails only when the element that triggers the bug is silently changed, and lists harmless mechanics (http vs https, offline/dry-run, verbose flags, install path) that do not count. `outcome-honest` now judges the headline conclusion and certainty words, not brief side notes. Both loosened, so the next run carried canaries.
2. `--only pkg-01,pkg-03,pkg-02,pkg-08,pkg-15,calib-03 --include-calibration`: **5/5 scored** (calib-03 also matched, unscored). The two false rejects flipped to accept; every canary stayed reject.
3. Full run (20 packages): **19/20, PASS**, every category matched. Only miss: pkg-05 (gold accept, graded reject on `steps-rerunnable`). A first attempt at this run crashed before grading anything on a Windows text-encoding error from an emoji in a package; rerunning with `PYTHONUTF8=1` fixed it without touching the harness.
4. Confirming full run with `--save-run eval-run.txt`: **19/20, PASS** (bar 18/20), categories clear-accept 7/8, disclosure 1/1, no-evidence 4/4, unfollowable-comms 3/3, wrong-target 4/4. This is the committed `eval-run.txt`.

**Package analysis**

**pkg-05** (conda/conda#16543). Gold label: **accept**. My rubric's verdict: **reject**, failed on `steps-rerunnable`.

The report shows a real, well-matched artifact: `conda env update --quiet --json -f env.yml 2>/dev/null` prints the `EnvironmentSectionNotValid` warning above the JSON on stdout, and piping it into `python3 -m json.tool` fails with `Expecting value: line 1 column 1 (char 1)`. Behavior, environment, and honesty all passed. But the one input the trigger depends on, `env.yml`, is only described ("a valid `dependencies:` list plus a `category:` section"), never pasted. My `steps-rerunnable` check says a stranger must be able to re-run "without guessing" with "inputs shown inline or from a public source", so the grader read the missing file as a guess and held it.

The gold label treats that description as minimal enough to rebuild in seconds, which is fair. I kept my check as it is on purpose: the same rule is what rejects pkg-18, whose repro lives in a private monorepo with an unshared config. Loosening "inputs shown inline" to "inputs described" would risk letting that kind of report through, and I'm already above the bar.

**Check rationale**

| repo-conventions | The repo-facts block's contribution policy (including any AI-use policy) read against both the claim comment and the repro report. Treat every package as AI-assisted work. | If the policy requires disclosing AI use in issues/comments, pass only if a comment actually discloses it (tool and extent); silence is a fail. If the policy only requires disclosure in PRs, only asks that comments be in the contributor's own words, or states no AI policy, pass unless the comments themselves visibly break a stated rule. | required |

Why it reads this way: the eval set has exactly one disclosure package (pkg-20, ghostty), whose policy says "All AI usage in any form must be disclosed", and the comments never disclose. A check that only asked "does it follow the repo's conventions?" would pass it, because nothing in the comments looks wrong. So the check makes silence an explicit fail when disclosure is required, and tells the grader to treat every package as AI-assisted so it cannot excuse silence.

The second half is there to stop the opposite mistake. pkg-03 (ripgrep) only says comments "must be written by humans in their own words", pkg-09 (fd) asks for disclosure in pull requests but "states no disclosure ask for issue comments", and pkg-05 (conda) is permissive. All three are gold accepts. Writing out those cases keeps the grader from turning any AI-related sentence into a disclosure requirement. In both full runs pkg-20 was rejected, pkg-03 and pkg-09 were accepted, and pkg-05's only failed check was `steps-rerunnable`, not this one.

**Trade-offs**

Loosening `trigger-faithful` and `outcome-honest` after run 1 risked letting wrong-target and no-evidence packages through, so I re-ran canaries with `--only` before spending a full run: pkg-02 (wrong range syntax) and pkg-08 (modified expression) for `trigger-faithful`, pkg-15 (unbacked root-cause claim) for `outcome-honest`, and calib-03 (the colon-for-equals HCL trap) for both. All four stayed reject. I did not canary pkg-20 because neither change touched `repo-conventions`; both full runs confirmed disclosure stayed 1/1.

The case I accept `steps-rerunnable` will miss is pkg-05: a report that describes a tiny input file instead of pasting it gets held. That is the price of the same check reliably rejecting pkg-18's private-repo reproduction.
