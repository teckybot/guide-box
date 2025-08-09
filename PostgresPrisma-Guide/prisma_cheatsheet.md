
## Prisma Workflow Cheat Sheet 

**Purpose:** A compact, practical reference you can pin to your monitor. Covers commands, recommended workflows, deployment steps, checklists, and troubleshooting so you never mix up `db push`, `migrate dev`, and `migrate deploy` again.

---

### Quick Command Reference

- `npx prisma init` — initialize Prisma in the project (creates `prisma/schema.prisma` and a `.env` entry).
- `npx prisma migrate dev --name <name>` — create a migration file from `schema.prisma`, apply it to the local DB, and regenerate Prisma Client.
- `npx prisma db push` — push the current `schema.prisma` to the database **without** creating migration files (fast local sync).
- `npx prisma migrate deploy` — run committed migrations on the target environment (production/staging).
- `npx prisma migrate reset` — drop the DB, reapply migrations, and rerun seed (destructive — local/dev use only).
- `npx prisma generate` — regenerate the Prisma Client (use when schema changed and you only want the client updated).
- `npx prisma studio` — open web UI to inspect and edit DB data (dev tool).

---

### Recommended Workflows (Simple Rules)

- **Prototype / iterate fast:** use `npx prisma db push` while experimenting locally. Quick and convenient.
- **Keep history for production:** when a change is final (release-ready), run `npx prisma migrate dev --name <descriptive_name>` locally and commit the resulting `prisma/migrations` folder.
- **Deploy to production (or staging):** after pulling the code that contains committed migrations, run `npx prisma migrate deploy` on the target environment.

**Golden Rule:** Never run `db push` in production. Migrations are the source of truth for production schema changes.

---

### Typical Day-to-Day: Example Sequences

**A — Quick prototype session**

1. Edit `prisma/schema.prisma`.
2. Run:
   ```bash
   npx prisma db push
   ```
3. Test locally with your app.

When you decide to release this work, convert the state to migrations (see "Converting prototype to migrations").

**B — Development with migration history (recommended)**

1. Edit `prisma/schema.prisma`.
2. Run:
   ```bash
   npx prisma migrate dev --name add_users_table
   ```
3. Run tests, use `npx prisma studio` to inspect data, and commit `prisma/migrations/` and `prisma/schema.prisma`.
4. Push to GitHub.

**C — Deploy to Production**
If you want to setup for Production on VPS Follow this Guide: [click here](https://github.com/teckybot/guide-box/blob/main/PostgresPrisma-Guide/deploy_to_production.md)
---

## Before deployment
#### Converting prototype (db push-only) to migrations (before deployment)

If you iterated with `db push` and never ran `migrate dev`, do this **before** deploying:

1. Ensure your local `.env` points to the dev DB that matches your current schema.
2. Run:
   ```bash
   npx prisma migrate dev --name init
   ```
   This creates a baseline migration that represents your current schema and regenerates the client.
3. Verify `prisma/migrations/` contains the new migration.
4. Commit `prisma/migrations/`, `prisma/schema.prisma`, and push to Git.
5. On production run `npx prisma migrate deploy`.

> Note: Running `migrate dev` after `db push` is the safe way to produce migration files matching your existing dev DB schema.

---

### CI / CD Recommendations

- **In pipeline (build or deploy step):**

  1. Install dependencies: `npm ci` or `npm install`
  2. Run `npx prisma generate` (so the generated client exists for build/tests)
  3. On deployment runner (before app start): run `npx prisma migrate deploy` to apply migrations.
  4. Start the app.

- **Environment variables:** Ensure `DATABASE_URL` is set in CI/CD secrets and points to the correct environment (staging/production).

- **Staging first:** Always apply migrations in a staging environment that mirrors production, run smoke tests, then deploy to production.

---

### Common Pitfalls & Troubleshooting

- **I committed no migrations but deployed:** `migrate deploy` will have nothing to run. Fix: locally run `npx prisma migrate dev --name init` to generate baseline migrations, commit, and then deploy.
- **Conflicting migrations across branches:** two branches modifying schema can create divergent migration files. Best practice:
  - Prefer small, frequent merges.
  - Rebase feature branches and generate migrations from the latest main branch, or
  - If conflict occurs, create a consolidated migration on a single branch and reapply on top of main.
- **Schema drift (DB != migrations):** investigate with backups, avoid `db push` in prod, consider `prisma migrate resolve` only as a last resort and after backing up the DB.
- **Accidental destructive changes:** migrations can drop columns/tables. Always review generated SQL in `prisma/migrations/<timestamp>_name/steps.sql` before applying to prod.

---

### Safety & Best Practices

- **Back up** production DB before applying migrations.
- **Run migrations on staging first**, then promote to production.
- **Review autogenerated SQL** in migration folders for dangerous operations.
- **Use descriptive migration names**: `add_user_age`, `rename_order_total_to_amount`.
- **Avoid **``** in production** — it has no history and is difficult to reproduce.

---

### Quick Recovery Commands (use carefully)

- `npx prisma migrate reset` — rebuild your local DB from migrations (destructive: drops data).
- `npx prisma generate` — regenerate Prisma Client after schema changes.
- `npx prisma studio` — inspect DB in a browser.

