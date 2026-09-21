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

With PostgreSQL or MySQL, my application talks to a separate database server.

That means another piece of infrastructure to deploy, configure, secure, monitor, back up, and eventually scale.

SQLite takes a different approach.

The database is just a file that my application can work with directly.

I still get SQL, transactions, indexes, foreign keys, joins, constraints, and all the other things I expect from a relational database.

The biggest difference is that I don't need to run a separate database server just to use them.

---

## "But You Can't Use SQLite in Production, Right?"

And people would argue:

"But you can't use SQLite in production!"

Why not?

SQLite is a production-ready relational database. The question isn't whether it can run in production. It's how you want to run and scale it.

I can develop locally with SQLite and use tools like "sqlc" to generate my Go database code.

When I deploy the application, I can use Turso and its libSQL ecosystem.

I don't have to treat SQLite as something I use temporarily before moving to a "real" database.

SQLite is the real database.

---

## Introducing Turso

Turso is built on libSQL, an open-source fork of SQLite. It gives me a way to use the SQLite ecosystem in production without having to manage the database infrastructure myself.

I can start with a local SQLite database and use Turso when I deploy my application.

What I also like about Turso is its approach to scaling.

With a traditional SQL server, scaling a database can eventually involve things like read replicas, replication, failover, connection pooling, backups, and monitoring.

PostgreSQL and MySQL can absolutely handle large workloads. But once you're running them yourself, you're also responsible for managing the database infrastructure.

With Turso, I can use SQLite locally and still use the same SQLite-based approach in production without having to manage the database server myself.

---

## SQLite + Go + sqlc

For Go, I also really like the combination of SQLite and "sqlc".

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
