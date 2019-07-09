---
layout: default
title: Limiting Columns
parent: Retrieving
nav_order: 2
---

# Limiting Columns

In the [last section](./basic-retrieval.html) we pulled all 
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
records from the database, and we included all properties/columns in the query.  We can
limit the selected properties/columns using the
[FromAdapter.select](../../api-doc/latest/classes/fromadapter.html#select)
method, which optionally takes one or more property names.  Here's how.

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .select('p.id', 'p.firstName');
```

A few notes about the above query.

* An _alias_, `p`, has been provided for `Person`.  Like SQL, an alias lets us
  reference `People` using a short-hand notation in other parts of the query,
  like when limiting the selected columns.  It also lets us disambiguate in
  cases where the same table is joined in multiple times.  Aliasing is
  optional.  If no alias is provided then the table will be referenced by
  _table_ name (`people` in this case).
* The primary key column(s) must be selected for every table referenced in the
  query, including any joined-in tables.  Here we've included `p.id` as part of
  the `select`.
* Each selected column takes the form `<table-alias>.<property-name>`.  Recall
  that in our [example database](../migrations/) the primary key of the
  `people` table is `personID`, and in the
  [models](../models/basic-model-decorators.html) section we opted to associate
  that column with the `Person.id` property.  So, in the above query, we select
  `p.id`, not `p.personID`.

### Full Example

Here's a full example, which can be found in the
[formn-example](https://github.com/benbotto/formn-example/) repository under
[src/retrieve/limit-columns.ts](https://github.com/benbotto/formn-example/blob/master/src/retrieve/limit-columns.ts).

```typescript
import { inspect } from 'util';

import { MySQLDataContext, ConnectionOptions, Select } from 'formn';

import { Person } from '../entity/person.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // Only pull Person.id and Person.firstName.
    const query: Select<Person> = dataContext
      .from(Person, 'p')
      .select('p.id', 'p.firstName');

    console.log(query.toString());

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

Run it with `ts-node`.

```
npx ts-node ./src/retrieve/limit-columns.ts
```

The output will looks similar to the following.

```
SELECT  `p`.`personID` AS `p.id`,
        `p`.`firstName` AS `p.firstName`
FROM    `people` AS `p`
[
  Person {
    id: 1,
    firstName: 'Joe'
  },
  Person {
    id: 2,
    firstName: 'Rand'
  },
  Person {
    id: 3,
    firstName: 'Holly'
  },
  Person {
    id: 4,
    firstName: 'Jenny'
  },
  Person {
    id: 5,
    firstName: 'Mickey'
  },
  Person {
    id: 6,
    firstName: 'Abe'
  }
]
```
