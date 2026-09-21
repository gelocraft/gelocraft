+++
date = '2026-09-21T06:13:41Z'
draft = false
title = 'Why I choose SQLite as my default relational database in 2026 — featuring Turso'
+++


---
If you asked me a few years ago what database I'd choose for a new application, I probably would've said PostgreSQL.

In 2026, my default is different:

**SQLite.**

I like SQLite because it gives me what I need from a relational database without making me run another server.

## SQLite doesn't need a server

With PostgreSQL or MySQL, my application talks to a separate database server:

```
App → Network → Database Server
```

That means another piece of infrastructure to deploy, configure, secure, monitor, back up, and eventually scale.

With SQLite:

```
App → SQLite → Database File
```

That's it.

I still get SQL, transactions, indexes, foreign keys, joins, constraints, and all the other things I expect from a relational database.

---

## "But You Can't Use SQLite in Production, Right?"

And people would argue:

"But you can't use SQLite in production!"

Why not?

SQLite is a production-ready relational database. The question isn't whether it can run in production. It's how you want to run and scale it.

I can develop locally with:

```
Go → sqlc → SQLite
```

And use Turso in production:

```
Go → sqlc → Turso/libSQL
```

I don't have to treat SQLite as something I use temporarily before moving to a "real" database.

SQLite is the real database.

---

## Introducing Turso

Turso is built on libSQL, an open-source fork of SQLite. It lets me use SQLite in production without having to manage the database infrastructure myself.

I can start with a local SQLite database and use Turso when I deploy my application.

The other thing I like about Turso is how it approaches scaling.

With a traditional SQL server, scaling can mean dealing with things like:

```
Primary
  ↓
Read Replicas
  ↓
Replication
  ↓
Failover
  ↓
Connection Pooling
  ↓
Backups
  ↓
Monitoring
```

PostgreSQL and MySQL can absolutely handle large workloads, but operating that infrastructure is another problem on top of building the application.

Turso handles that database infrastructure for me while I continue using the SQLite/libSQL ecosystem.

That's the part I like.

I can start with SQLite without having to plan a migration to a completely different database just because my application grows.

---

## SQLite + Go + sqlc

For Go, I also really like the combination of SQLite + sqlc.

I can write SQL directly:

```sql
-- name: GetUser :one
SELECT id, name, email
FROM users
WHERE id = ?;
```

Then "sqlc" generates the Go code from my queries.

No giant ORM. No hiding SQL behind another abstraction.

Just SQL and Go.

Simple, explicit, and boring.

And honestly, boring is good.

{{< nextprev >}}
