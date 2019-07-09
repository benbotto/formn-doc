---
layout: default
title: Create a People Table
parent: Migrations
nav_order: 3
---

# Create a People Table

To start, let's generate a new migration script.

```
formn m create create_table_people
```

That will create a new `migrations` directory with a single script in it.  All
we have to do is fill out the SQL in the `up` and `down` methods.

```javascript
'use strict';

module.exports = {
  /**
   * Run the migration.
   */
  up(dataContext) {
    const sql = `
      CREATE TABLE people (
        personID INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
        firstName VARCHAR(255),
        lastName VARCHAR(255),
        createdOn TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP)`;
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
    const sql    = `DROP TABLE people`;
    const params = {};

    console.log(sql);

    return dataContext
      .getExecuter()
      .query(sql, params); 
  }
};
```

`up` creates the table, and `down` drops it.  Simple.

### Populate the People Table

Now let's make a migration to populate the `people` table with some dummy data.

```
formn m create insert_people
```

This migration is less trivial than the last.  We'll loop over an array of
people, inserting one at a time and keeping track of the results.  To keep
things simple, we've made the methods
[async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).

```javascript
'use strict';

const people = [
  {firstName: 'Joe', lastName: 'Shmo'},
  {firstName: 'Rand', lastName: 'al\'Thor'},
  {firstName: 'Holly', lastName: 'Davis'},
  {firstName: 'Jenny', lastName: 'Mather'},
];

module.exports = {
  /**
   * Run the migration.
   */
  async up(dataContext) {
    const results = [];

    for (let i = 0; i < people.length; ++i) {
      const sql    = 'INSERT INTO people (firstName, lastName) VALUES (:firstName, :lastName)';
      const params = people[i];

      console.log(sql);
      console.log(params);

      const result = await dataContext
        .getExecuter()
        .query(sql, params);

      results.push(result);
    }

    return results;
  },

  /**
   * Bring down a migration.
   */
  async down(dataContext) {
    const results = [];

    for (let i = 0; i < people.length; ++i) {
      const sql    = 'DELETE FROM people WHERE firstName = :firstName AND lastName = :lastName';
      const params = people[i];

      console.log(sql);
      console.log(params);

      const result = await dataContext
        .getExecuter()
        .query(sql, params);

      results.push(result);
    }

    return results;
  }
};
```

The `down` method is pretty much the same as the `up`, but it deletes records
rather than inserting.

### Run the Migrations

Run the two migrations by using the `up` sub-command.

```
formn m up
```

Formn will run the two scripts sequentially, one after the other, in order.

To bring down the latest migration, use the `down` sub-command.  This command
is especially useful during development when the database schema is subject to
change.

```
formn m down
```

The migrations presented in this section are both available in the
[formn-example](https://github.com/benbotto/formn-example) repository in the
[migrations](https://github.com/benbotto/formn-example/tree/develop/migrations)
folder.  There are a couple more migrations covered in the [following
section](./create-a-phone-numbers-table.html), and they're a bit more advanced.

