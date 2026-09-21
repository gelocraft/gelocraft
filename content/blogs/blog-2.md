+++
date = '2026-09-21T06:13:41Z'
draft = false
title = 'Why I choose SQLite as my default relational database in 2026 — featuring Turso'
+++


---
If you asked me a few years ago what database I'd choose for a new application, I probably would've said PostgreSQL.

In 2026, my answer is different:

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

Turso is a distributed database built on libSQL, an open-source fork of SQLite. The interesting part is that it keeps the SQLite model while allowing the database to run across multiple locations.

Instead of having one SQLite file sitting on the same server as my application, Turso can distribute database replicas to different locations. My application can connect to a nearby replica, which makes SQLite much more practical for applications that need to serve users from different regions.

This is also what makes the idea of using SQLite in production much more realistic. SQLite itself is excellent for many applications, but a single database file on a single machine isn't designed for the kind of distribution you might need as an application grows. Turso extends that model by distributing the database while keeping the SQLite-compatible ecosystem and workflow.

Compare that with running PostgreSQL yourself. As the application grows, scaling a traditional SQL server can become a much bigger infrastructure problem. You may eventually need read replicas, replication, connection pooling, failover, backups, monitoring, and strategies for handling database migrations and high availability.

PostgreSQL and MySQL are extremely capable databases, but operating them at scale involves managing all of that infrastructure.

Turso takes a different approach: keep the simplicity and familiarity of SQLite, but distribute the database so it can be used in production across multiple locations.

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
