---
layout: default
title: Ordering
parent: Retrieving
nav_order: 3
---

# Ordering

The [Select](../../api-doc/latest/classes/select.html) class has an
[orderBy](../../api-doc/latest/classes/select.html#orderby) method that's used
to sort the query.  The method takes one or more objects, each with

1. a `property` string in the form `<table-alias>.<property-name>`, and
2. a `dir` (direction) that is one of `ASC` or `DESC`.

For example, to sort
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
records by `firstName` then `lastName`:

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .select('p.id', 'p.firstName', 'p.lastName')
  .orderBy(
    {property: 'p.firstName', dir: 'DESC'},
    {property: 'p.lastName',  dir: 'ASC'});
```

The [orderBy](../../api-doc/latest/classes/select.html#orderby)
method can also be passed one or more property strings, in which case
the direction is `ASC`.

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .select('p.id', 'p.firstName', 'p.lastName')
  .orderBy('p.firstName', 'p.lastName');
```

### Full Example

You can find a full example in the formn-example repository.  It's under
[src/retrieve/order.ts](https://github.com/benbotto/formn-example/blob/master/src/retrieve/order.ts).

```typescript
import { inspect } from 'util';

import { MySQLDataContext, ConnectionOptions, Select } from 'formn';

import { Person } from '../entity/person.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // Order by first name then last name.
    const query: Select<Person> = dataContext
      .from(Person, 'p')
      .select('p.id', 'p.firstName', 'p.lastName')
      .orderBy(
        {property: 'p.firstName', dir: 'DESC'},
        {property: 'p.lastName',  dir: 'ASC'});

    // Alternatively orderBy can take strings, in which case
    // the direction is ASC.  E.g.:
    // .orderBy('p.firstName', 'p.lastName');

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
npx ts-node ./src/retrieve/order.ts
```

Output will looks something like this.

```
SELECT  `p`.`personID` AS `p.id`,
        `p`.`firstName` AS `p.firstName`,
        `p`.`lastName` AS `p.lastName`
FROM    `people` AS `p`
ORDER BY `p`.`firstName` ASC, `p`.`lastName` ASC
[
  Person {
    id: 3,
    firstName: 'Holly',
    lastName: 'Davis'
  },
  Person {
    id: 4,
    firstName: 'Jenny',
    lastName: 'Mather'
  },
  Person {
    id: 1,
    firstName: 'Joe',
    lastName: 'Shmo'
  },
  Person {
    id: 2,
    firstName: 'Rand',
    lastName: 'AlThore'
  }
]
```

