---
layout: default
title: Creating Migrations
parent: Migrations
nav_order: 2
---

# Creating Migrations

You can use the Formn CLI to generate a migration template script using the
`create` sub-command.  Just supply a name for the migration script and Formn
will generate a migration template for you.  We'll create and run some
migrations in the next sections, but first let's go over the basics.

### The Anatomy of a Migration Script

When you create a migration script, the script's name will be prefixed with a
timestamp.  The timestamp is informative in that it tells the developer when
each migration was created and in what order.  The naming convention is also
important internally: it's how Formn knows the order in which migrations should
be run.

A migration script is a plain old JavaScript file with two methods exported:
`up` and `down`.  The former brings a database up to date; the latter is the
inverse and reverses a migration.  So if, for example, the `up` method creates
a table, the `down` method should drop the table.

Both methods take a single argument: a connected
[DataContext](../../api-doc/latest/classes/datacontext.html) instance (see the
[DataContext](../datacontext/) section). Both methods should return a
[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
instance.  You also can make the methods
[async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).
(An
[async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
method implicitly returns a
[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).)

Behind the scenes, the Formn CLI creates a `formn_migrations` table.  This
table is used to track the migrations that have been run.  The table is
trivial, and includes the name of each migration and a timestamp that shows
when the migration was run.

### The Migration Template

When you create a migration, the generated template will look as follows.

```javascript
'use strict';

module.exports = {
  /**
   * Run the migration.
   */
  up(dataContext) {
    const sql    = ``;
    const params = {};

    console.log(sql);

    return dataContext
      .getExecuter()
      .query(sql, params); 
  },

  /**
   * Bring down a migration.
   */
  down(dataContext) {
    const sql    = ``;
    const params = {};

    console.log(sql);

    return dataContext
      .getExecuter()
      .query(sql, params); 
  }
};
```

Just add SQL strings and any parameters as needed, and return a
[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

[Next up](./create-a-people-table.html) we'll dive into some examples.
