---
layout: default
title: CLI Overview
parent: Migrations
nav_order: 1
---

# CLI Overview

The CLI command for migrations is `migrate`, aliased `m`.  You
can see all the commands and options using the `--help` switch.

```
formn --help
formn migrate --help
```

The `migrate` command has the following options:

* `--connections-file`, aliased as `-c`.  The path to a `connections.json`
  file, which defaults to `./connections.json`.
* `--flavor`, aliased `-f`.  The database type, which defaults to `mysql`.
* `--migrations-dir`, aliased `-m`.  A directory that holds migration scripts.
  It defaults to `migrations`.

`migrate` has four sub-commands:

* `create`: Create a new template migration script.
* `up`: Run all new migrations against the connection(s) defined in
  `connections.json`.
* `down`: Undo the most recent migration.
* `run`: Run a custom migration script against the database.

We'll look at each of these sub-commands in the next sections.

