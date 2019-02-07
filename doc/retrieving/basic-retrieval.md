---
layout: default
title: Basic Retrieval
parent: Retrieving
nav_order: 1
---

# Basic Retrieval

Back in the [DataContext](../datacontext/) section, the
[DataContext.from](../../api-doc/latest/classes/datacontext.html#from) method
was mentioned briefly.  This method is the starting point for retrieving
records, and it takes a single parameter: An
[EntityType](../../api-doc/latest/globals.html#entitytype), which is some
[Table](../../api-doc/latest/globals.html#table)-decorated class.  (See the
[Models](../models) section.)
[DataContext.from](../../api-doc/latest/classes/datacontext.html#from) returns
a [FromAdapter](../../api-doc/latest/classes/fromadapter.html) instance which
has a number of useful operations that we'll step through in the following
sections.

### Retrieve All From a Single Table

Let's say you want to select all
[Person](https://github.com/benbotto/formn-example/blob/1.3.0/src/entity/person.entity.ts)
records from the `people` table that we defined in the [Getting
Started](../getting-started/tutorial-database-setup.html) section.  To do so,
you would call the
[FromAdapter.select](../../api-doc/latest/classes/fromadapter.html#select)
method like so.

```typescript
const query: Select<Person> = dataContext
  .from(Person)
  .select();
```

You get back a [Select](../../api-doc/latest/classes/select.html) instance.  As
was mentioned in the [create](../creating/insert-a-record.html) example, all
Formn queries extend the [Query](../../api-doc/latest/classes/query.html) class
and thus share a common interface.  We can view the SQL associated with the
[Query](../../api-doc/latest/classes/query.html) instance using the
[toString](../../api-doc/latest/classes/query.html#tostring) method.

```typescript
console.log(query.toString());
```

And we can [execute](../../api-doc/latest/classes/query.html#execute) the query
to retrieve the
[Person](https://github.com/benbotto/formn-example/blob/1.3.0/src/entity/person.entity.ts)
records.

```typescript
const people: Person[] = await query
  .execute();
```

### Full Example

A complete example follows.  It's available in the `formn-example` repository under
[src/retrieve/retrieve-all-people.ts](https://github.com/benbotto/formn-example/blob/1.4.1/src/retrieve/retrieve-all-people.ts).

```typescript
import { inspect } from 'util';

import { MySQLDataContext, ConnectionOptions, Select } from 'formn';

import { Person } from '../entity/person.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // Create a Select query, which will pull all People records from the
    // people table.
    const query: Select<Person> = dataContext
      .from(Person)
      .select();

    // Log the generated SQL.
    console.log(query.toString());

    // Execute the query to get all Person records.
    const people: Person[] = await query
      .execute();

    console.log(inspect(people, {depth: null, compact: false}));

    await dataContext.end();
  }
  catch(err) {
    console.error(err);
  }
}

main();
```

Run it using `ts-node`.

```
npx ts-node ./src/retrieve/retrieve-all-people.ts
```

The output will look similar to the following.

```
SELECT  `people`.`createdOn` AS `people.createdOn`,
        `people`.`firstName` AS `people.firstName`,
        `people`.`lastName` AS `people.lastName`,
        `people`.`personID` AS `people.id`
FROM    `people` AS `people`
[
  Person {
    id: 1,
    createdOn: 2019-01-29T02:57:06.000Z,
    firstName: 'Joe',
    lastName: 'Shmo'
  },
  Person {
    id: 2,
    createdOn: 2019-01-29T02:57:06.000Z,
    firstName: 'Rand',
    lastName: 'AlThore'
  },
  Person {
    id: 3,
    createdOn: 2019-01-29T02:57:06.000Z,
    firstName: 'Holly',
    lastName: 'Davis'
  },
  Person {
    id: 4,
    createdOn: 2019-01-29T02:57:06.000Z,
    firstName: 'Jenny',
    lastName: 'Mather'
  },
  Person {
    id: 5,
    createdOn: 2019-01-29T03:05:14.000Z,
    firstName: 'Mickey',
    lastName: 'Mouse'
  },
  Person {
    id: 6,
    createdOn: 2019-01-29T03:06:01.000Z,
    firstName: 'Abe',
    lastName: 'Lincoln'
  }
]
```

