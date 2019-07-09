---
layout: default
title: Migrations
nav_order: 5
has_children: true
permalink: /doc/migrations/
---

# Migrations

Formn's CLI provides tools for creating and running database migrations.  A
migration is a simple JavaScript script that's run against one or more
databases and can be used to create tables, prepopulate data, add constraints,
etc.

This section assumes that you have set up a `connections.json` file.  If not,
check out the [Connecting](../connecting/) section.  Also make sure you've
installed the `formn-cli` package globally.

```
npm install -g formn-cli
```

