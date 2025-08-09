# Express + Prisma + Postgres Backend Setup (Local & Production)

This guide walks you through setting up an Express backend with Prisma and PostgreSQL that works seamlessly in both local development and production environments (e.g., Render or VPS).

---

## 1. Project Setup
```bash
mkdir my-backend
cd my-backend
npm init -y
npm install express cors dotenv
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

---

## 2. PostgreSQL Database Configuration

### A. Local Postgres Setup Recap
When you installed PostgreSQL on Windows, you got:
- **psql** → CLI to talk to Postgres
- **pgAdmin** → GUI for Postgres management
- A default `postgres` user and a default `postgres` database

### B. Creating a Development Database for Prisma
Avoid using the default `postgres` database for your app.

**Option 1: Using psql (CLI)**
```bash
psql -U postgres
CREATE DATABASE mydb;
\l   -- lists databases
\q   -- quit
```

**Option 2: Using pgAdmin (GUI)**
- Open pgAdmin
- Right-click **Databases → Create → Database**
- Name it `mydb`
- Owner = `postgres`

---

## 3. Connection String
Once your DB is created, Prisma connects via `.env`:
```env
DATABASE_URL="postgresql://postgres:your_password@localhost:5432/mydb"
```
**Breakdown:**
- `postgresql://` → Protocol
- `postgres` → DB username
- `your_password` → password for that user
- `localhost` → host (local machine)
- `5432` → default Postgres port
- `mydb` → database name

**Production Example:**
```env
DATABASE_URL="postgresql://user:password@host:5432/dbname?sslmode=require"
```

---

## 4. Prisma Schema Setup

### Initial Schema
Open `prisma/schema.prisma`:
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

### Adding Your First Model
Example:
```prisma
model User {
  id    Int     @id @default(autoincrement())
  name  String
  email String  @unique
}
```

### Migrations
When schema changes, run:
```bash
npx prisma migrate dev --name descriptive_name
```
What happens:
Prisma compares current schema to DB.
Generates SQL migration in prisma/migrations/.
Runs SQL in your DB.
Updates Prisma Client so it matches your schema.

If only changing local dev DB without migrations:
```bash
npx prisma db push
```

#### Pro tip:
During development: Use npx prisma db push to quickly sync schema changes to your local database. This is fast and convenient when you're iterating rapidly.
Before pushing to GitHub or deploying: Run npx prisma migrate dev --name meaningful_name to:
Create a migration file in prisma/migrations
Apply it to your local DB
Regenerate the Prisma Client

# Example checklist
✔ Run `migrate dev` for final schema changes
✔ Check that `prisma/migrations` folder is updated
✔ Commit migration files
✔ Push to GitHub


### Running Your First Migration
```bash
npx prisma migrate dev --name init
```
This will:
1. Create the `User` table in your DB
2. Track migration history in `_prisma_migrations`
3. Generate the Prisma Client

---

## 5. Folder Structure Explained
```
prisma/schema.prisma    → Database blueprint (models)
prisma/migrations       → SQL migration scripts
node_modules/@prisma    → Generated Prisma Client code
src/                    → Backend code (server, routes, controllers)
.env                    → Environment variables (DB URL)
```

---

## 6. Updating a Model
Example change:
```prisma
model User {
  id    Int    @id @default(autoincrement())
  name  String
  email String @unique
  age   Int?   // new optional column
}
```
Re-Run:
```bash
npx prisma migrate dev --name add-age-to-user
```
This will:
- Create a new migration file
- Apply it to your DB
- Update the Prisma Client

---

## 7. Common Commands
| Command                                   | When to Use                                                                        | What It Does                                                                                                |
| ----------------------------------------- | ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `npx prisma migrate dev --name some_name` | After changing schema (add/remove columns, models)                                 | Creates migration, applies to DB, regenerates client                                                        |
| `npx prisma generate`                     | Schema changed but no migration needed                                             | Regenerates Prisma Client only                                                                               |
| `npx prisma db push`                      | Force DB to match schema without keeping history (avoid in production)             | Overwrites DB without migrations                                                                             |
| `npx prisma studio`                       | Anytime                                                                            | Opens a browser UI to view/edit DB                                                                           |

---

## 8. Viewing Data in Prisma Studio
```bash
npx prisma studio
```
Opens a browser UI for viewing, adding, and editing data.

---

## 9. How Prisma Works — Big Picture
1. **Schema File (`schema.prisma`)** → Defines your DB blueprint
2. **Migration System (`prisma/migrations`)** → Tracks DB changes as SQL scripts
3. **Generated Client** → Type-safe JS/TS code for querying the DB

**Think of `prisma/migrations` like Git for your database.** It keeps dev and prod in sync.

---

