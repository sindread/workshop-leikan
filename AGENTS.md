# AGENTS.md

Domain rules (points, scoreboard): `docs/TRONDER_LEIKAN.md`. Write comments, error messages and test names in Norwegian.

## Running

- Use the Aspire CLI (`aspire start`, `aspire wait`, `aspire describe`, `aspire logs`), never `dotnet run` on the AppHost. The `aspire` skills in `.claude/skills/` cover the workflow.
- Run the AppHost only from the main clone, never from a worktree or with `--isolated`. Zitadel needs its fixed port, and the shared Postgres volume and `zitadel-bootstrap/` live only in the main clone. Run only tests in worktrees. This overrides the Aspire skills, which recommend `aspire start --isolated` in worktrees.
- The `migrator` resource applies EF migrations and seeds demo data; the API does not migrate itself. New migration: `dotnet ef migrations add <Name> --project src/TronderLeikan.Infrastructure`, then restart `migrator`.
- Broken local state (Zitadel/Postgres errors): see README "Feilsøking", usually fixed by `./reset-local.sh`.
- Admin login for browser checks: `zitadel-admin@zitadel.localhost` / `Password1!` at `<frontend>/admin`.

## Tests

- `Api.Tests` and `Infrastructure.Tests` use Testcontainers, so Docker must be running.
- `Application.Tests` uses `TestAppDbContext` (EF InMemory), which copies the EF mapping by hand. Update it when you add DbSets, owned types or backing-field collections.

## Conventions

- Business errors are `Result`/`Error` values, not exceptions. Add errors to `Application/Common/Errors/<Feature>Errors.cs`; controllers do `.Match(Ok, Problem)`.
- Handlers use `IAppDbContext`, never `AppDbContext` (which is internal to Infrastructure).
- The API has no authentication. Only the frontend's `src/proxy.ts` guards `/admin`.
- Domain events go to the outbox on `SaveChangesAsync`, but nothing processes the outbox yet. The event handlers are placeholders.
