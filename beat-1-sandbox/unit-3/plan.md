# Plan for issue #51: check migrations in CI

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51
Base: fork `rishihk/pathreview-ai301-fa26-s1` at commit `f89c06fc3ff292df2a04a39ac51319d32a76b779`
My reproduction: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51#issuecomment-5881199893

## Diagnosis

CI never touches the migrations, so a broken migration and a schema that has drifted from the models both pass every job today. Evidence from my reproduction:

```
$ grep -nE "alembic|migration" .github/workflows/ci.yml || echo "no match"
no match
$ ls scripts/validate_migrations.sh
ls: cannot access 'scripts/validate_migrations.sh': No such file or directory
```

`test-integration` starts a Postgres service but only runs `pytest tests/integration`. It never runs `alembic upgrade head` or `alembic check`.

There is also real drift on `main` right now:

```
$ .venv/Scripts/alembic check
INFO  [alembic.autogenerate.compare.constraints] Detected removed unique constraint 'uq_users_email' on 'users'
FAILED: New upgrade operations detected: [('remove_constraint', UniqueConstraint(Column('email', NullType(), table=<users>)))]
```

My reading of the code (to be confirmed, see unknowns): `alembic/versions/001_initial_schema.py` creates both `sa.UniqueConstraint("email", name=op.f("uq_users_email"))` and `op.create_index(op.f("ix_users_email"), "users", ["email"], unique=True)`. The model, `core/models/user.py`, declares `email` with `unique=True, nullable=False, index=True`, which SQLAlchemy turns into the unique index only. So the migration makes one extra constraint that the model does not have.

Because of this drift, a new CI check that runs `alembic check` would fail on its first run on `main`. The drift has to be fixed in the same change, or the new job cannot go green.

And a broken migration does fail on a fresh database, but only when someone runs it by hand:

```
$ .venv/Scripts/alembic upgrade head
asyncpg.exceptions.UndefinedColumnError: column "nonexistent_column" of relation "reviews" does not exist
```

## Scope

In scope (one pull request):

1. Add `scripts/validate_migrations.sh`. It runs `alembic upgrade head` on the database in `DATABASE_URL`, then `alembic check`, and exits non-zero if either fails.
2. Add one new job, `migrations`, to `.github/workflows/ci.yml`. It starts a fresh `postgres:16-alpine` service and runs the script.
3. Add `alembic/versions/003_drop_duplicate_users_email_unique.py`, which drops `uq_users_email` (and puts it back on downgrade), so that `alembic check` passes on `main` and the new job starts green.

Not in scope:

- Changing the models in `core/models/`.
- Editing `001` or `002`, since they may already be applied in other databases.
- Changing the existing five jobs.
- Making the new job a required check in branch protection (that is a maintainer setting).
- Any other drift. If `alembic check` still reports something after item 3, I will note it on the issue as separate work and not fix it here.
- Adding integration tests.

## Files I will touch

- `scripts/validate_migrations.sh` (new)
- `.github/workflows/ci.yml` (add one job, nothing else changes)
- `alembic/versions/003_drop_duplicate_users_email_unique.py` (new; revision `"003"`, down_revision `"002"`)

## Approach (in order)

1. On a clean Postgres, run `alembic upgrade head`, then check in `psql` that `ix_users_email` is a unique index and `uq_users_email` is a separate constraint on the same column.
2. Write migration `003` with `op.drop_constraint("uq_users_email", "users", type_="unique")` in `upgrade()` and `op.create_unique_constraint("uq_users_email", "users", ["email"])` in `downgrade()`.
3. Run `downgrade base`, `upgrade head`, `alembic check` and confirm `alembic check` exits 0.
4. Write `scripts/validate_migrations.sh`. `alembic/env.py` runs the async engine, so a plain `postgresql://` URL will not work; the script rewrites a `postgresql://` prefix to `postgresql+asyncpg://` before running alembic. It defaults to the `.env.example` URL when `DATABASE_URL` is not set.
5. Add the `migrations` job to `ci.yml`, copying the Python 3.11, `pip install -e ".[dev]"`, and Postgres service pattern from `test-integration`, with `DATABASE_URL` pointing at the service.
6. Push the branch to my fork, run the CI workflow on it with `workflow_dispatch`, and confirm all six jobs are green.

## Test plan

This re-runs my unit 2 reproduction steps against the change.

| Repro step | Before (posted in unit 2) | Expected after |
|---|---|---|
| 1. `alembic downgrade base` then `alembic upgrade head` on an empty DB | ends at `002 (head)` | runs 001, 002, 003 and `alembic current` prints `003 (head)` |
| 2. `alembic check` | `FAILED: New upgrade operations detected: ... uq_users_email` | exits 0 with `No new upgrade operations detected.` |
| 3. Add throwaway `003_broken_demo.py` (now with `down_revision = "003"`) and run `bash scripts/validate_migrations.sh` | alembic fails with `UndefinedColumnError`, but nothing in CI runs it | the script exits non-zero and prints the same `UndefinedColumnError`. I delete the file afterward. |
| 4. `grep -nE "alembic|migration" .github/workflows/ci.yml` and `ls scripts/validate_migrations.sh` | `no match` and `No such file or directory` | grep shows the `migrations` job and the script runs the two alembic commands; `ls` finds the file |

Also: on the pushed branch, the `migrations` job shows green in the GitHub Actions tab, together with lint, typecheck, test-unit, test-integration, and frontend.

## Risks and unknowns

- I have not yet confirmed that `ix_users_email` is unique in the migrated database. If it is not, dropping `uq_users_email` would remove the only uniqueness guarantee on emails, and I would change the plan (fix the model instead) before touching anything.
- I tested on Python 3.12 and CI uses 3.11. I expect no difference, but I have not run it on 3.11.
- After `uq_users_email` is dropped, `alembic check` may report more drift that it did not show before. I will only know when I run it.
- A maintainer may prefer the drift fix as its own pull request. I included it here so the new job is not red on day one, and I can split it if asked.
- I have not run the workflow on GitHub Actions yet, so the service health check and the script's file permissions in CI are untested.

## Deviations

The build followed the plan: migration 003, scripts/validate_migrations.sh, and the new `migrations` job in .github/workflows/ci.yml, in that order, as three commits. Three things were different from what I wrote, and none of them changed what I built.

1. How I ran the workflow. I planned to run it on my fork with workflow_dispatch. GitHub had not registered the CI workflow in my fork (the workflow page said "This workflow does not exist"), so there was no Run workflow button. I opened a pull request inside my own fork (rishihk/pathreview-ai301-fa26-s1, pull/1, titled "test run only: do not merge") to trigger it, and closed it without merging. All six jobs passed: lint, typecheck, test-unit, test-integration, frontend, and migrations. Run: https://github.com/rishihk/pathreview-ai301-fa26-s1/actions/runs/37398674332

2. How the broken-migration failure looks. The plan said the script would print UndefinedColumnError. With a throwaway 004_broken_demo.py (down_revision "003"), the script exited 1 and printed the SQLAlchemy ProgrammingError wrapper with the same message: column "nonexistent_column" of relation "reviews" does not exist. I only showed the last 8 lines, which cut off the asyncpg line underneath. It is the same failure as in my unit 2 reproduction.

3. The unknowns I listed are now answered. In psql, ix_users_email is a UNIQUE index on email and uq_users_email is a separate UNIQUE CONSTRAINT on the same column, so dropping the constraint keeps email unique. alembic check exited 0 after migration 003, so there was no further drift. The continuous integration job passed on Python 3.11.

The posted plan is still accurate, so I did not add a comment on the issue.
